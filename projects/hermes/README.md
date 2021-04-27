# Solarix | Hermes

**Note: This README is extracted from the original source project that I created. Please see [WARNING & COPYRIGHT NOTICE](../../../README.md#warning--copyright-notice) for more info.**

## Configuration

Hermes is deployed to the `docs.solarix.tools` Amazon S3 bucket in the `/hermes` directory.

### Allowing Restricted Access to Non-Public S3 Bucket

Since Hermes contains sensitive data the S3 bucket is not publicly accessible. Visiting https://s3-us-west-2.amazonaws.com/docs.solarix.tools/hermes/index.html responds with `403` forbidden error:

```xml
<Error>
    <Code>AccessDenied</Code>
    <Message>Access Denied</Message>
    <RequestId>771A75D6609BA987</RequestId>
    <HostId>
        7AIxINfhHatVxquIILa/OcSBP6b/HZFUfBvkEqILHq+auyt6xw/DRXryoDuFwgZO/kI0EFYUOYw=
    </HostId>
</Error>
```

The solution is to force requests to the S3 bucket through a specific CloudFront URL, which we can then restrict to only allow requests to that CloudFront URL from specific IP addresses. The following steps were taken to get everything configured properly.

1. Create a new CloudFront Distribution (url: `https://dfqll17oxg5t.cloudfront.net/`, srn: `srn:cloudfront:solarix:hermes::distribution`) that uses the `docs.solarix.tools` S3 bucket as its origin.
2. Create a CloudFront Origin Access Identity (OAI) (srn: `srn:cloudfront:solarix:hermes::oai`, id: `origin-access-identity/cloudfront/EM9LGTIS77W8Z`).
3. Assign the `srn:cloudfront:solarix:hermes::oai` OAI to the CloudFront Distribution to restrict access to the underlying S3 bucket.
4. Create a new Bucket Policy on the S3 bucket to allow the `s3:GetObject` permission for the OAI:

```json
{
  "Version": "2008-10-17",
  "Id": "PolicyForCloudFrontPrivateContent",
  "Statement": [
    {
      "Sid": "Allow-OAI-Access-To-Bucket",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity EM9LTIS77W8Z"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::docs.solarix.tools/*"
    }
  ]
}
```

We're halfway there. A request to the CloudFront Distribution URL (https://dfqll1n7oxg5t.cloudfront.net/index.html) now returns the `index.html` object from our bucket. However, this is still not secure as anyone with the CloudFront URL still has public access.

To prevent access from requests made directly to the CloudFront Distribution URL we need to add an AWS Web Application Firewall (WAF) ACL and rule.

1. Create a new AWS WAF IP set (`srn-wafv2-solarix-hermes_ip-set-solarix-domains`).
2. Set the IP addresses to `54.213.172.234/32` -- the IP of our internal `srn:ec2:solarix:domains::instance` EW2 instance.
3. Create a new AWS WAF ACL (`srn-wafv2-solarix-hermes_acl-solarix-domains`) and a associated rule set (`srn-wafv2-solarix-hermes_rule-solarix-domains`) that blocks all traffic _except_ from the origin IP of our `srn:ec2:solarix:domains::instance` instance.
4. Finally, associate this WAF ACL with our CloudFront Distribution (`srn:cloudfront:solarix:hermes::distribution`).

Now, a request to both the direct S3 bucket and the CloudFront Distribution URL results in a `403` error, so we've shut down both routes to public access. The next step is to setup our private NGINX server (`srn:ec2:solarix:domains::instance`) to route to the CloudFront Distribution URL.

1. SSH to `srn:ec2:solarix:domains::instance`.
2. Open `/etc/nginx/sites-available/solarix.tools.conf`.
3. Add a new virtual host entry:

```nginx
server {
  include snippets/default.conf;

  server_name docs.solarix.tools;

  auth_basic "Private";
  auth_basic_user_file /etc/nginx/.htpasswd;

  # Proxy to CloudFront
  location / {
    proxy_pass http://dfqll1n7oxg5t.cloudfront.net;
  }
}
```

Now requests coming into `docs.solarix.tools` proxy to our CloudFront Distribution URL. We also perform basic auth with the password specified in `/etc/nginx/.htpasswd`.

4. Save and restart NGINX: `sudo systemctl restart nginx`

Visiting `https://docs.solarix.tools/` now properly displays the Hermes homepage! However, another problem must be addressed: Because we're accessing our S3 bucket via a REST API endpoint and not the website endpoint, we need a way to tell CloudFront how to deal with non-root objects when requesting a subpath.

For example, making a request to `https://docs.solarix.tools/solarix-resource-names/` attempts to download an empty object, since there is nothing at that URL as far as CloudFront + S3 are concerned. We want CloudFront to handle such requests by pointing to the underlying `index.html`. This can be accomplished with a simple `Lambda@Edge` function.

1. Create a new Lambda@Edge function ([redirectCloudFrontOriginToIndexHtml](https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions/redirectCloudFrontOriginToIndexHtml/versions/$LATEST?tab=configuration)). _Note: Lambda@Edge functions can only be created in the `us-east-1` region, but since we're using it with CloudFront it acts as a global anyway._
2. Add the following code, which grabs the request and adds the missing `index.html` string to the URI, which points to the correct object in the backend S3 bucket:

```js
// redirectCloudFrontOriginToIndexHtml
exports.handler = async (event, context, callback) => {
  // Get request
  const request = event.Records[0].cf.request;
  // Replace slash (/) with index.html
  request.uri = request.uri.replace(/\/$/, '/index.html');
  return callback(null, request);
};
```

3. Add a trigger to the Lambda function, select our CloudFront Distribution, and approve deployment.

That's it! Now, a request coming into `https://docs.solarix.tools` is proxied to the CloudFront Distribution URL (`https://dfll1n7oxg5t.cloudfront.net`), which is the only valid way to access our private S3 bucket content. Request URIs automatically have `index.html` appended, and any request that doesn't pass through the `https://docs.solarix.tools` URL is denied with a 403 error.

## Darkmode Theme

Darkmode theme courtesy of [Mister-Hope](https://github.com/Mister-Hope) (see [PR#2301](https://github.com/vuejs/vuepress/pull/2301)).
