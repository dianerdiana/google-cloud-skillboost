# Getting Started with API Gateway

## Set variables

```
export REGION="us-east1"
export FUNCTION_NAME="gcfunction"
export PROJECT_ID=""
export REDEPLOY_FUNCTION_NAME="helloHttp"
```

## Task 1. Create a Cloud Run function

- Create folder & file

```
mkdir ~/$FUNCTION_NAME && cd $_

touch index.js && touch package.json

cat > index.js <<EOF
const functions = require('@google-cloud/functions-framework');
functions.cloudEvent('$FUNCTION_NAME', () => {
  console.log('Hello World!');
});
EOF

cat > package.json <<EOF
{
  "name": "nodejs-functions-gen2-codelab",
  "version": "0.0.1",
  "main": "index.js",
  "dependencies": {
    "@google-cloud/functions-framework": "^3.0.0",
    "@google-cloud/pubsub": "^3.4.1"
  }
}
EOF
```

- Create Cloud Run Function with allowing unauthenticated invocations

```
gcloud functions deploy $FUNCTION_NAME \
  --gen2 \
  --runtime nodejs22 \
  --entry-point $FUNCTION_NAME \
  --source . \
  --region $REGION \
  --trigger-http \
  --timeout 600s \
  --allow-unauthenticated
```

## Task 2. Create an API Gateway

- Create file openapispec.yaml

```yaml
swagger: "2.0"
info:
  title: gcfunction API
  description: Sample API on API Gateway with a Google Cloud Run functions backend
  version: 1.0.0
schemes:
  - https
produces:
  - application/json
x-google-backend:
  address: https://gcfunction-PROJECT_NUMBER.REGION.run.app
paths:
  /gcfunction:
    get:
      summary: gcfunction
      operationId: gcfunction
      responses:
        "200":
          description: A successful response
          schema:
            type: string
```

## Task 3. Create a Pub/Sub Topic and Publish Messages via API Backend

- Update package.json

```
{
  "dependencies": {
    "@google-cloud/functions-framework": "^3.0.0",
    "@google-cloud/pubsub": "^3.4.1"
  }
}
```

- Update index.js

```
/**
 * Responds to any HTTP request.
 *
 * @param {!express:Request} req HTTP request context.
 * @param {!express:Response} res HTTP response context.
 */
const {PubSub} = require('@google-cloud/pubsub');
const pubsub = new PubSub();
const topic = pubsub.topic('demo-topic');
const functions = require('@google-cloud/functions-framework');

exports.helloHttp = functions.http('helloHttp', (req, res) => {

  // Send a message to the topic
  topic.publishMessage({data: Buffer.from('Hello from Cloud Run functions!')});
  res.status(200).send("Message sent to Topic demo-topic!");
});
```

- Redeploy

```
gcloud functions deploy $REDEPLOY_FUNCTION_NAME \
  --gen2 \
  --runtime nodejs22 \
  --entry-point $REDEPLOY_FUNCTION_NAME \
  --source . \
  --region $REGION \
  --trigger-http \
  --timeout 600s \
  --allow-unauthenticated

```

- Testing API Development afer Create Pub/Sub Topic

```
export GATEWAY_URL=$(gcloud api-gateway gateways describe $REDEPLOY_FUNCTION_NAME --location $REGION --format json | jq -r .defaultHostname) \
echo $GATEWAY_URL
```

- Testing

```
curl -s -w "\n" https://$GATEWAY_URL
```
