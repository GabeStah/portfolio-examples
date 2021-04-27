- [Solarix Resource Names (SRNs)](#solarix-resource-names-srns)
  - [SRN Format](#srn-format)
  - [SRN Tag Hierarchy](#srn-tag-hierarchy)
  - [SRN Examples](#srn-examples)
  - [SRN Limitations](#srn-limitations)
- [Domains](#domains)
- [CI/CD (DTSP)](#cicd-dtsp)
  - [Repository Branches](#repository-branches)
  - [Stage 1: Development > Testing](#stage-1-development--testing)
  - [Stage 2: Testing > Staging](#stage-2-testing--staging)
  - [Stage 3: Staging > Production](#stage-3-staging--production)
- [VPCs](#vpcs)
  - [Solarix - Primary](#solarix---primary)
  - [NGINX](#nginx)
    - [Example: solarix.dev](#example-solarixdev)
- [DNS (Route 53)](#dns-route-53)
  - [Create Resource Records](#create-resource-records)
  - [Get Nameservers](#get-nameservers)
- [Certificates / Acme Certbot](#certificates--acme-certbot)
  - [IAM User](#iam-user)
  - [Creating Certificates](#creating-certificates)
  - [Add Certs to Server](#add-certs-to-server)
    - [Example: NGINX](#example-nginx)
  - [Prompt](#prompt)
- [MongoDB](#mongodb)
- [Nginx](#nginx-1)

## Solarix Resource Names (SRNs)

A `Solarix Resource Name (SRN)` is a string that uniquely identifies a Solarix resource, such as a server, credential set, service, virtual network, etc. Wherever possible, an appropriate SRN should be assigned to a given resource and used to reference it in all technical documentation or discussion. SRNs are **heavily** influenced by [Amazon Resource Names](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) which serve much the same purpose.

### SRN Format

An SRN is a string identifier comprised of specifically-ordered tags, concatenated by colon (`:`) separators. The format below shows the required tag types, along with optional tag types in brackets:

```
srn:service:org[:project][:app][:environment]::resource-type[/resource-id]
```

Here are the basic definitions of each tag type with a few examples:

- `srn` **(Required)** - Static prefix value of `srn`. The presence allows for programmatic SRN verification.
- `service` **(Required)** - The service in which this resource resides (e.g. s3, ec2, rds, stripe, sendgrid, discord, etc).
- `org` **(Required)** - Organizational entity. Typically, this is an internal org name, a client name, or individual user name (e.g. solarix, voyager, wcasg, raritan, gabe, kyle, etc).
- `project` _(Optional)_ - Name of the overall project or internal grouping (e.g. widget, core, etc).
- `app` _(Optional)_ - Specific application or repository (e.g. domains, dashboard, tracker, connector, etc).
- `environment` _(Optional)_ - Environment in which this resource resides (e.g. development, testing, staging, or production).
- `resource-type` **(Required)** - Represents the Type of resource (e.g. instance, security group, vpc, subnet, pem, cert, db, cache, etc).
- `resource-id` _(Optional)_ - Identifier for the specific resource. This identifier **must complete a wholly unique SRN** as compared to all other SRNs with similar contextual tags.

### SRN Tag Hierarchy

The most basic SRN consists of the (4) required tags (`srn`, `service`, `org`, and `resource-type`):

```
srn:service:org::resource-type
```

Notice the explicit **double-colon** separator (`::`) between the two _required_ sections. Specifically, the double-colon must always proceed the `resource-type`, which helps parsers check for optional vs required tags.

An SRN's tag ordering is hierarchical, such that each additional tag adds more specificity. Tags are ordered such that the existence of a tag _implies_ the existence of all prior tags up to that point. To reiterate: **the inclusion of an optional tag _requires_ the existence of ALL previous tags.**

For example, it doesn't make sense for a resource to have an `app` tag if there's no `project` to associate that app with. Similarly, if there's no `app` then the concept of a specific `environment` does not apply. Some examples:

- `srn:service:org::resource-type`: Valid
- `srn:service:org:project::resource-type`: Valid
- `srn:service:org:project:app::resource-type`: Valid
- `srn:service:org:project:app:environment::resource-type`: Valid
- `srn:service:org:project:app:environment::resource-type/resource-id`: Valid - Includes all possible tags.
- `service:org:project:app:environment::resource-type/resource-id`: Invalid - Missing the explicit `srn` prefix.
- `srn:service:org:::resource-type`: Invalid - The tags are correct but there's an extra colon separator.
- `srn:service:org:environment::resource-type`: Invalid - Inclusion of an `environment` without proceeding `project` and `app` tags.
- `srn:service:org:project:environment::resource-type`: Invalid - The `app` tag is missing, which is required to have an `environment` tag.

### SRN Examples

Below are some "real world" examples of how SRNs are formatted based on the resource's contextual information.

| Service    | Org  | Project             | App | Environment | Resource      | SRN                                              |
| ---------- | ---- | ------------------- | --- | ----------- | ------------- | ------------------------------------------------ |
| Amazon EC2 | Acme | Road Runner Tracker | API | Development | Sole instance | `srn:ec2:acme:tracker:api:development::instance` |

Since this is the only EC2 instance applicable to this context (app, environment, etc), we can leave off the additional `resource-id` identifier/qualifier.

| Service    | Org  | Project             | App | Environment | Resource | SRN                                                     |
| ---------- | ---- | ------------------- | --- | ----------- | -------- | ------------------------------------------------------- |
| Amazon EC2 | Acme | Road Runner Tracker | API | Production  | Primary  | `srn:ec2:acme:tracker:api:production::instance/primary` |
| Amazon EC2 | Acme | Road Runner Tracker | API | Production  | Canary   | `srn:ec2:acme:tracker:api:production::instance/canary`  |

In this example the `Production` environment of the same API app uses canary deployment, which means there are two active EC2 instances for this context, so we need to include an identifier.

Always explicitly include the `resource-type`, even if it may seem obvious or is repeated, as in the case of a `vpc` `resource-type` within the Amazon VPC service:

| Service    | Org  | Project             | App | Environment | Resource                                | SRN                                 |
| ---------- | ---- | ------------------- | --- | ----------- | --------------------------------------- | ----------------------------------- |
| Amazon VPC | Acme | Road Runner Tracker |     |             | Security group providing dev SSH access | `srn:vpc:acme:tracker::sg/dev-ssh`  |
| Amazon VPC | Acme | Road Runner Tracker |     |             | Public VPC dedicated to this project    | `srn:vpc:acme:tracker::vpc/public`  |
| Amazon VPC | Acme | Road Runner Tracker |     |             | Private VPC dedicated to this project   | `srn:vpc:acme:tracker::vpc/private` |

Here we see VPC resources that are used across ALL apps on ALL environments within Acme's "Road Runner Tracker" project. However, because of our explicit [SRN tag hierarchy](#srn-tag-hierarchy) we can quickly scan and recognize the (4) required tags plus the (2) optional `project` and `resource-id` tags.

| Service    | Org     | Project | App | Environment | Resource                            | SRN                                 |
| ---------- | ------- | ------- | --- | ----------- | ----------------------------------- | ----------------------------------- |
| Amazon VPC | Solarix |         |     |             | Org-wide VPC for Solarix            | `srn:vpc:solarix::vpc`              |
| Amazon VPC | Solarix |         |     |             | Public Subnet A for all of Solarix  | `srn:vpc:solarix::subnet/public-a`  |
| Amazon VPC | Solarix |         |     |             | Public Subnet B for all of Solarix  | `srn:vpc:solarix::subnet/public-b`  |
| Amazon VPC | Solarix |         |     |             | Private Subnet A for all of Solarix | `srn:vpc:solarix::subnet/private-a` |
| Amazon VPC | Solarix |         |     |             | Private Subnet B for all of Solarix | `srn:vpc:solarix::subnet/private-b` |

Here we see more examples of minimal tag usage while retaining appropriate formatting. Since Solarix has just one VPC, we only need to include the (4) required tags of `srn`, `service`, `org`, and `resource-type`. The subnets don't apply to any particular projects, but since there are multiple subnets we must include a unique `resource-id`.

### SRN Limitations

SRNs are primarily used as AWS tags and internal identifiers when discussing particular resources. However, there are a few known limitations:

- Max Length: SRNs cannot exceed 255 total characters.
- Must Be Lowercase: Consistency demands that all SRNs remain lowercase. While a computer can easily distinguish subtle differences between casing in an SRN string, mixed-casing is too ambiguous for human discussions.
- Character Requirements: SRNs should contain _only_ alphanumeric or the `:-/` special characters.

## Domains

- Naming Scheme: `app.project.client.domain.tld`

| Base Domain   | Deployment Stage | Client Dashboard URL  | Project App / Dashboard URL     | Project App URL                       |
| ------------- | ---------------- | --------------------- | ------------------------------- | ------------------------------------- |
| solarix.tools | internal         | N/A                   | [project].solarix.tools         | [app].[project].solarix.tools         |
| solarix.dev   | testing          | [client].solarix.dev  | [project].[client].solarix.dev  | [app].[project].[client].solarix.dev  |
| solarix.site  | staging          | [client].solarix.site | [project].[client].solarix.site | [app].[project].[client].solarix.site |
| solarix.host  | production       | [client].solarix.host | [project].[client].solarix.host | [app].[project].[client].solarix.host |

## CI/CD (DTSP)

The deployment process should generally occur across four consecutive stages: development, testing, staging, and production.

### Repository Branches

While smaller, simpler projects need not abide by this, larger/long-term projects should adopt a **trifecta** branching scheme. Each branch corresponds with one of the three deployment environments, making the automated deployment of a given commit easy to understand and follow.

- `master` branch is the primary development branch. CI/CD should push commits on this branch to the **testing** environment (`solarix.dev`).
- `staging` branch is the staging branch. CI/CD should push commits on this branch to the **staging** environment (`solarix.site`).
- `production` branch is the production branch. CI/CD should push commits on this branch to the **production** environment (`solarix.host`).

### Stage 1: Development > Testing

- Developer makes changes in local dev environment.
- Developer commits change.
- Developer pushes change to either:
  - New branch and generates pull request.
  - `master` branch.
- If CI/CD:
  - This `master` commit is deployed to **testing** environment (`solarix.dev`).

### Stage 2: Testing > Staging

- If tests are green from **testing**:
  - Developer pushes commit to `staging` branch.
- If CI/CD:
  - This `staging` commit is deployed to **staging** environment (`solarix.site`).

### Stage 3: Staging > Production

- If tests are green from **staging**:
  - Developer pushes commit to `production` branch.
- If CI/CD:
  - This `production` commit is deployed to **production** environment (`solarix.host`).

## VPCs

### Solarix - Primary

Primary VPC that contains a **Public Subnet** for internet-accessible instances and a **Private Subnet** for non-public instances.

- Name: `vpc-solarix`
- Id: `vpc-0508b5f46395c819`
- IPv4 CIDR Block: `20.0.0.0/16`
- Subnets
  - Public
    - Name: `pub-sub-solarix`
    - Id: `subnet-035779f30f345f38`
  - Private
    - Name: `pri-sub-solarix`
    - Id: `subnet-0f4c648df7ea8d57`
- ACLs
  - Name: `acl-solarix`
  - Id: `acl-03e9713612bd713a`

### NGINX

#### Example: solarix.dev

- File: `/etc/nginx/snippets/default.conf`

```conf
listen 80;
listen [::]:80;

listen 443 ssl;
listen [::]:443 ssl;

ssl_certificate     /home/ubuntu/.acme.sh/solarix.dev/solarix.dev.cer;
ssl_certificate_key /home/ubuntu/.acme.sh/solarix.dev/solarix.dev.key;

# Add index.php to the list if you are using PHP
index index.html index.htm index.nginx-debian.html;
```

- File: `/etc/nginx/sites-available/solarix.dev.conf`

```conf
server {
	include snippets/default.conf;

	root /var/www/html/solarix/solarix.dev;
	server_name solarix.dev www.solarix.dev;
}
```

## DNS (Route 53)

Docs: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/DomainNameFormat.html#domain-name-format-asterisk

- Root domains or subdomains should use asterisk (`*`) as a catchall for all _non-specific_ domains. Consider these records:
  - `solarix.dev A 34.223.160.131 300`
  - `*.solarix.dev A 34.223.160.131 300`
  - `domains.solarix.dev A 123.456.789.012 300`
  - A request to `solarix.dev` resolves to the first explicit record of `34.223.160.131`.
  - A request to `random.solarix.dev` resolves to the second wildcard record of `34.223.160.131`.
  - A request to `domains.solarix.dev` finds the explicit third record and resolves to `123.456.789.012`.

### Create Resource Records

Docs: https://docs.aws.amazon.com/cli/latest/reference/route53/change-resource-record-sets.html

1. Create JSON configuration file specifying new resource records, e.g:

```json
{
  "Comment": "solarix.site - Creating default A record sets.",
  "Changes": [
    {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "solarix.site",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [
          {
            "Value": "34.223.160.131"
          }
        ]
      }
    },
    {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "*.solarix.site",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [
          {
            "Value": "34.223.160.131"
          }
        ]
      }
    }
  ]
}
```

2. Execute `route53 change-resource-record-sets` command pointing to config with matching hosted zone id:

```
λ aws route53 change-resource-record-sets --change-batch file://awscli\route53\solarix.site\create-default-A-records.json --hosted-zone-id Z08202973TX0FDDYUODO3
```

3. Confirm result shows pending change:

```json
{
  "ChangeInfo": {
    "Id": "/change/C00393492QWT0PRSJZZIQ",
    "Status": "PENDING",
    "SubmittedAt": "2020-04-11T21:52:57.242000+00:00",
    "Comment": "solarix.site - Creating default A record sets."
  }
}
```

### Get Nameservers

Via CLI:

```
λ aws route53 get-hosted-zone --id "/hostedzone/Z08211673J8S41S7BTNZ"
{
    "HostedZone": {
        "Id": "/hostedzone/Z08211673J8S41S7BTNZ",
        "Name": "solarix.dev.",
        "CallerReference": "F707326D-CE51-F099-8FFC-D571BEC6BA80",
        "Config": {
            "Comment": "",
            "PrivateZone": false
        },
        "ResourceRecordSetCount": 4
    },
    "DelegationSet": {
        "NameServers": [
            "ns-411.awsdns-51.com",
            "ns-1809.awsdns-34.co.uk",
            "ns-777.awsdns-33.net",
            "ns-1174.awsdns-18.org"
        ]
    }
}
```

## Certificates / Acme Certbot

Tool: https://github.com/acmesh-official/acme.sh

### IAM User

- ARN: `arn:aws:iam::696585593512:user/solarix-certbot`
- Groups: `grp-certbot`
  - Policies: `arn:aws:iam::696585593512:policy/route53-hosted-zone-resources`
    - [URL](https://console.aws.amazon.com/iam/home?region=us-west-2#/policies/arn:aws:iam::696585593512:policy/route53-hosted-zone-resourcesjsonEditor)
  -

### Creating Certificates

Docs: https://github.com/acmesh-official/acme.sh/wiki/dnsapi#10-use-amazon-route53-domain-api

1. Export AWS ENV vars with `solarix-certbot` access keys.

```
export  AWS_ACCESS_KEY_ID=XXXXXXXXXX
export  AWS_SECRET_ACCESS_KEY=XXXXXXXXXXXXXXX
```

2. Issue **DNS-based** certificate generation command for domain(s):

```
$ acme.sh --issue --dns dns_aws -d solarix.dev -d www.solarix.dev -d solarix.host -d www.solarix.host -d solarix.site -d www.solarix.site -d solarix.tools -d www.solarix.tools -d app.project.client.solarix.dev
...
[Sun Apr 12 00:05:10 UTC 2020] Your cert is in  /home/ubuntu/.acme.sh/solarix.dev/solarix.dev.cer
[Sun Apr 12 00:05:10 UTC 2020] Your cert key is in  /home/ubuntu/.acme.sh/solarix.dev/solarix.dev.key
[Sun Apr 12 00:05:10 UTC 2020] The intermediate CA cert is in  /home/ubuntu/.acme.sh/solarix.dev/ca.cer
[Sun Apr 12 00:05:10 UTC 2020] And the full chain certs is there:  /home/ubuntu/.acme.sh/solarix.dev/fullchain.cer
```

### Add Certs to Server

#### Example: NGINX

```conf
ssl_certificate     /home/ubuntu/.acme.sh/solarix.dev/solarix.dev.cer;
ssl_certificate_key /home/ubuntu/.acme.sh/solarix.dev/solarix.dev.key;
```

---

1. create vpc
2. add 4 subnet (2 pub 2 pri)
   - pub: west-2a -2b
   - pri: west-2c -2d
3. rds mysql
4. elasticache redis

### Prompt

```
$ nano ~/.bashrc
```

- Adjust `PS1`:

```
PS1='${debian_chroot:+($debian_chroot)}\u@dashboard:widget:wcasg:production:\w\$ '
```

- Extra PHP Extensions

```
sudo apt install php-pear php-dom php-mbstring php-curl php-zip php-gd php-apcu php-dev zip unzip yarn
sudo pecl install mongodb
```

```
# Update Node
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
```

```
# Install Yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update && sudo apt install yarn
```

```
# Update node modules
sudo yarn install
```

## MongoDB

- Allow incoming connections from `0.0.0.0` (any).

```conf
# /etc/mongod.conf

# Listen to all
bind_ip = 0.0.0.0
```

- Control access via AWS security groups.

## Nginx

For non-HTTP proxy to upstream server, if needed.

```conf
# /etc/nginx/nginx.conf
stream {
  server {
    listen 3000;

    proxy_connect_timeout 1s;
    proxy_timeout 3s;
    proxy_pass    localhost:27017;
  }
}
```
