# Apicize API Samples

This project consists of a couple of APIs that help demonstrate the functionality in [Apicize][https://github.com/apicize/apicize].  They are implemented using AWS Lambda and API Gateway 
and are deployed using the [AWS Serverless Application Model](https://aws.amazon.com/serverless/sam/)

## Functional Overview

### `https://sample-api.apicize.com/image/(right|flip|left)` (POST)

Accepts an image file posted as Base64 and rotates it.  Image size is limited by [Lambda's invocation payload](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html) of 6 MB.  Requires bearer token issued by `/token`.

### `https://sample-api.apicize.com/quote[/:id]` (GET, POST, PUT, DELETE)

CRUD services to store quote information in a [DynamoDb](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html) database.   Requires bearer token issued by `/token`, and entries are tied to the token used to submit them.

### `https://sample-api.apicize.com/token` (POST)

A minimial OAuth2-ish that issues tokens which expire in 10 minutes.  (note: this is *not* a "real" authentication/authorization service)

## Configuration and Deployment

If you are going to deploy this for yourself, refer to the [SAM template](./template.yaml) for parameters to configure.  

To run locally, you can create an `env.local.json` with the following contents

```json
{
    "IssueTokenFunction": {
        "TABLE_NAME_TOKENS": "my-token-table",
        "CIPHER_KEY": "xxxxxxxxxxxx"
    },
    "ImageFunction": {
        "TABLE_NAME_TOKENS": "my-token-table",
        "CIPHER_KEY": "xxxxxxxxxxxx"
    },
    "QuotesFunction": {
        "TABLE_NAME_TOKENS": "my-token-table",
        "TABLE_NAME_QUOTES": "my-quote-table",
        "CIPHER_KEY": "xxxxxxxxxxxx"
    }
}
```

You can generate a cipher key using the following NodeJS code:

```js
    Buffer.from(await crypto.subtle.exportKey('raw', await crypto.subtle.generateKey({name: 'AES-CBC', length: 256},true,['encrypt','decrypt']))).toString('base64')
```

NPM scripts are included to assist with running locally and deploying:

* **build**: builds all Lambda functions using **esbuild**
* **start**: launches services locally using AWS SAM
* **debug**: launches services locally using AWS SAM with debug enabled
* **deploy**: deploys services to Lambda (you will probaby want to run `sam deploy --guided` the firsd time to generate a `samconfig.toml` file)