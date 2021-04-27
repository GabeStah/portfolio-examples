**Note: This README is extracted from the original source project that I created. Please see [WARNING & COPYRIGHT NOTICE](../../../README.md#warning--copyright-notice) for more info.**

An AWS Lambda app that provides a secure API proxy for Google Text-to-Speech API requests.

## Development

1. Install [SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html).
1. Run `yarn install` to install node modules.
1. Make local code changes, based off of `index.js` entry point. To prevent pathing issues, try to keep core modules in the root directory.
1. Run `yarn dev` to start a local API at http://127.0.0.1:3000
1. Send a requests to local endpoint (e.g. `POST` to `http://127.0.0.1:3000/standard`).

**NOTE**: Making requests against the local development API's Docker endpoint (`127.0.0.1:3000`) feature significant overhead, which produces a significant response delay. Pay attention to the reported `Billed Duration` period in the console log output to evaluate the _actual_ response time that can be expected from production deployment:

```
...
START RequestId: d0057584-bf7e-1856-ad2e-99c2b41575d5 Version: $LATEST
REPORT RequestId: d0057584-bf7e-1856-ad2e-99c2b41575d5  Init Duration: 270.06 ms        Duration: 138.95 ms     Billed Duration: 200 ms Memory Size: 512 MB     Max Memory Used: 48 MB
2020-05-10 03:15:10 127.0.0.1 - - [10/May/2020 03:15:10] "POST /voices HTTP/1.1" 200 -
...
```

## Testing

Run `yarn test`.

## Building

Run `yarn build` to generate new a local build in `.aws-sam/build` directory.

## Deploying

Run `yarn deploy` to deploy based on the `template.yaml` and `samconfig.toml` configuration settings.

## Staging / Production

Once deployed, send requests to the API Gateway endpoint for the appropriate environment:

- Production: https://9qytmg2n1.execute-api.us-west-2.amazonaws.com/Prod
- Staging: https://9qytmg2n1.execute-api.us-west-2.amazonaws.com/Stage

## Example Requests

### Standard TTS Production Request

```
$ curl --location --request POST 'https://9qytmg2n1.execute-api.us-west-2.amazonaws.com/Prod/standard' \
--header 'Content-Type: application/json' \
--data-raw '{
    "audioConfig": {
        "audioEncoding": "LINEAR16",
        "speakingRate": 1,
        "pitch": 0,
        "sampleRateHertz": 24000,
        "volumeGainDb": 0,
        "effectsProfileId": [
            "handset-class-device"
        ]
    },
    "input": {
        "text": "This is some text to read."
    },
    "voice": {
        "languageCode": "en-US",
        "name": "en-US-Wavenet-D",
        "ssmlGender": "FEMALE"
    }
}'
```

Returns:

```json
{
  "audioContent": "UklGRlgYAQBXQVZFZm10IBAAAAABAAEAwF0AAIC7AAACABAAZGF0[TRUNCATED]"
}
```

### Get Voices Production Request

```
$ curl --location --request POST 'https://9qytg2tn1.execute-api.us-west-2.amazonaws.com/Prod/voices' \
--header 'Content-Type: application/json' \
--data-raw '{
    "languageCode": "en-US"
}'
```

Returns:

```json
{
    "voices": [
        {
            "languageCodes": [
                "en-IN"
            ],
            "name": "en-IN-Wavenet-D",
            "ssmlGender": "FEMALE",
            "naturalSampleRateHertz": 24000
        },
        {
            "languageCodes": [
                "en-AU"
            ],
            "name": "en-AU-Wavenet-A",
            "ssmlGender": "FEMALE",
            "naturalSampleRateHertz": 24000
        },
        ...
    ]
}
```

## Configuration

- SAM App: https://us-west-2.console.aws.amazon.com/lambda/home?region=us-west-2#/applications/lambda-to-google-tts
- Functions:
  - Standard: https://us-west-2.console.aws.amazon.com/lambda/home?region=us-west-2#/functions/lambda-to-google-tts-StandardFunction-NFN9L93VVHN?tab=configuration (`srn:sam:wcasg:widget::app/lambda-to-google-tts`)
  - Voices: https://us-west-2.console.aws.amazon.com/lambda/home?region=us-west-2#/functions/lambda-to-google-tts-VoicesFunction-53Y79VH41WG?tab=configuration (`srn:sam:wcasg:widget::app/lambda-to-google-tts`)
- Deployment S3 Bucket: https://s3.console.aws.amazon.com/s3/buckets/aws-sam-cli-managed-default-samclisourcebucket-dl6z30u7pypu/lambda-to-google-tts/?region=us-west-2&tab=overview

## Order of Operations

- End-user makes WCASG Dashboard `/api/widget/{token}` request
- WCASG Dashboard identifies requesting origin `Site` and generates payload
- Payload contains webpackified JS to load and execute Widget on end-user device
- End-user makes TTS request
- AWS API Gateway forwards request to AWS Lambda `/standard` function
- `/standard` Lambda makes Google TTS request
- `/standard` returns TTS response to end-user device
- Device plays back transformed text audio

### Challenge

AWS Lambda must invoke a secondary process that ultimately generates a Coeus API request with `Site` and TTS payload information.

Need a way to transmit `Site` identifier to `/standard` Lambda request, which can invoke backend queue message/processing.

- Identifier _MUST_ originate from WCASG Dashboard
- Identifier _MUST_ be compact
- Identifier _MUST NOT_ be guessable
- Identifier _MUST NOT_ be reversible

## Troubleshooting

- "Cannot Find Module" error in local API dev: Typically a problem with Docker directory/drive mounting. Try removing and re-adding the appropriate development directory/drive as a Docker mount.
