---
layout: post
title:  "Deploy a service into GCP Cloud Run and secure with an API Key"
author: Jordi Dalmau Heras
date:   2023-08-05 09:40:12 +0200
categories: operations
---

We will learn how to implement a web service made with Node.js as a Cloud Run service and provide access to the server with an API gateway with an API key.

That's why we will first develop a very simple web service with Node.js, deploy it to Cloud Run, configure a GCP service account to access it, define an API with the metrics we will expose with OpenApi v2 format, deploy an API Gateway that will expose the API definition and we will control access using API Keys.

We will [use an API KEY](https://cloud.google.com/endpoints/docs/openapi/when-why-api-key) instead other metods because we only want to:

- Block anonymous traffic
- Control the number of calls made to our API
- Identify usage patterns in our API's traffic
- Filter logs by API key

![API Gateway architecture overview](images/google-apikey-cloudrun.png)

Image from <https://cloud.google.com/api-gateway/docs/architecture-overview>

## Build the web service to deploy

### Develop the service and the Dockerfile

First of all, we have developed a very simple web service that exposes several APIs using Node.js.

```javascript
const http = require("http");

// A JSON array of products
const products = require("./products.json");

const host = '0.0.0.0';
const port = 8080;

const requestListener = function (req, res) {
  console.log(`Received request to url:`, req.url);

  const { headers, method, url } = req;
  let body = [];
  req.on('error', (err) => {
    console.error(err);

    res.setHeader("Content-Type", "application/json");
    res.writeHead(500);
    res.end(`{"message": "Internal server error ${err.message}"}`);
  }).on('data', (chunk) => {
    body.push(chunk);
  }).on('end', () => {
    body = Buffer.concat(body).toString();
    console.log(`Received request [${method}] ${url}`);
    console.log(`Received request with headers:`, headers);
    console.log(`Received request with body data:`, body);

    res.setHeader("Content-Type", "application/json");

    if (url == '/v1/products') {
      res.writeHead(200);
      return res.end(JSON.stringify(products));
    }

    const urlProductRegex = /^\/v1\/products\/[0-9]+/;
    if (urlProductRegex.test(url) === true) {
      let index = url.lastIndexOf('/');
      let idFound = url.substring(index + 1);
      let objectFound = products.find(e => e.id == idFound);
      if (objectFound) {
        res.writeHead(200);
        return res.end(JSON.stringify(objectFound));
      } else {
        res.writeHead(404);
        return res.end(`{"message": "product with id ${idFound} not found"}`);
      }
    }

    // RESOURCE NOT FOUND
    res.writeHead(404);
    return res.end(`{"message": "${url} not found"}`);
  });
};

const server = http.createServer(requestListener);
server.listen(port, host, () => {
  console.log(`Server is running on http://${host}:${port}`);
});
```

If we want to deploy this service into Cloud Run, we need to choose how to do it. Google Cloud provides us with several [options](https://cloud.google.com/run/docs/deploying):

1. [Deploy container images](https://cloud.google.com/run/docs/deploying): the idea is build a container image, push to a container registry ([Google Artifact Registry](https://cloud.google.com/artifact-registry/docs/overview) or [Docker Hub](https://hub.docker.com/)) and build the Cloud Run Service from the image pushed into the container registry
2. [Continous deployment from git](https://cloud.google.com/run/docs/continuous-deployment-with-cloud-build): the idea is link a git repository to Cloud Run and every time the master branch of the repository is updated automatically it deploys a new Cloud Run Service.
3. [Deploy from source code](https://cloud.google.com/run/docs/deploying-source-code): Cloud Run supports several languages and allows to push a source code manually. This deployment method doesn't require a Dockerfile.

For our purpose, we choose the 2nd method: continuous deployment from GitHub. With this method, we will be able to deploy new versions by making commits to the branch we select in the GitHub repository, which will significantly speed up the deployment process. This approach is very useful not only during the development process but also in production for performing maintenance and fixing bugs.

To achieve this, we need a Dockerfile to create a container image and connect GitHub to Google Cloud.

To build a Dockerfile, we need to define several key points:

1. The base image of the container: select the [images](https://hub.docker.com/search?q=&type=image&image_filter=official) that fit your project and has a small size as possible. We choose [node alpine images](https://hub.docker.com/_/node).
2. The port exposed by the container: Cloud Run exposes the port 8080 by default and we set this port to avoid extra [configuration](https://cloud.google.com/run/docs/configuring/services/containers).

Our Dockerfile is:

```Dockerfile
# pull official base image
FROM node:20-alpine

LABEL version="1.0"

# set working directory
COPY . /app
WORKDIR /app

# Exposem el port de l'aplicacio
EXPOSE 8080
# Definim la comanda per executar l'aplicacio
CMD ["node", "./index.js"]
```

### Test the service in test environment

```bash
# Create and tag the docker image
sudo docker build --tag codebiting/g2l-test-server:v1 .

# Run the docker in test
sudo docker run -p 8080:8080 -d codebiting/g2l-test-server:v1

# Check that the docker is running
sudo docker ps

# Inspect the docker logs
sudo docker logs <container_id>

# Check that from inside the docker can access the app
sudo docker exec -it <container_id> sh -c 'wget http://localhost:8080'

# Get docker info to search "IPAddress" for the docker IP.
sudo docker inspect <container_id>

# Test with a browser the docker
curl http://<IPAddress>:8080
```

## Deploy the service to Cloud Run

See <https://cloud.google.com/run/docs/continuous-deployment-with-cloud-build>

### Create a new Google Cloud project to aisle billing from other projects

In order to isolate the project and its associated costs we will create a new Google Cloud project.
Go to [IAM and admin -> Create a project](https://console.cloud.google.com/projectcreate) and create a new project.

Enable Google Cloud APIs:

- Cloud Source Repositories API
- Cloud Build API

### Setting up continuous deployment from the Cloud Run user interface

1. Go to [Cloud Run Console](https://console.cloud.google.com/run)
2. Create a new service
    1. Select "Continuously deploy new revisions from a source repository"
    2. In the Service settings page, click Set up with Cloud Build
        1. Select the provider and the repositor: GitHub in our case
        2. Select the repository to deploy from GitHub
        3. Select the branch
        4. Select the built type: in our case Dockerfile
        5. Save and Create
3. Set the service name
4. Set the region
5. Select the CPU allocation and pricing, in our case "CPU is only allocated during request processing"
6. Set the auto-scaling
7. Set the ingress control, in aur case "All"
8. Set the authentication, in our case "Require authentication"
9. Create

## Create a service account to allow access to the service

To be able to access the service deployed on Cloud Run, we will create a service account with the minimum permissions required to make calls to the service. API Gateway will then use this service account to access the service.

1. Go to [IAM and admin -> Service accounts](https://cloud.google.com/run/docs/authenticating/service-to-service)
2. Create service account
3. Set a service account name: note that the service account id and the email address are created automatically from the name
4. Set a role: to invoke a cloud run service its needed the role "Cloud Run Invoker"
5. Create

## Create an API Gateway with APY KEY authentication

### Create a OpenApi 2 definition in yaml format for the API specifiying an API-KEY to authenticate the calls

This yaml file must have the following requirements:

1. The title must have an API_ID followed by a description. The value of the title field is used when minting API keys that grant access to this API. See API ID requirements for API naming guidelines.
2. Set the address for each call: this address is made by the Cloud Run Service URL (it can find inside Cloud Run, in service details)
3. Into the securityDefinitions section set [where the API KEY must be provided](https://swagger.io/docs/specification/2-0/authentication/api-keys/), in the URL "query" or into the "header"
4. Verify the [API Key definition limitations](https://cloud.google.com/endpoints/docs/openapi/openapi-limitations#api_key_definition_limitations) to set the api key name.

```yaml
swagger: '2.0'
info:
  title: backend-apigateway-secure-v1 G2L Integration API
  description: G2L Integration API on API Gateway with a Google Cloud Run Backend
  version: 1.0.0
schemes:
  - https
produces:
  - application/json
paths:
  /v1/products:
    get:
      summary: Greet all the products
      operationId: GetProducts
      x-google-backend:
        address: https://g2l-test-server-xxxxxxxxxxx-uc.a.run.app/v1/products
      security:
      - api_key: []
      responses:
        '200':
          description: A successful response
          schema:
            type: string
securityDefinitions:
  # This section configures basic authentication with an API key.
  api_key:
    type: "apiKey"
    name: "x-api-key"
    in: "header"
```

### Create an API Key

1. Go to [API and services -> Credentials](https://console.cloud.google.com/apis/credentials) and [create an API Key](https://cloud.google.com/docs/authentication/api-keys#creating_an_api_key), by default the API can call any enabled API and has no restrictions.
2. Restrit to "API Gateway API"
3. Copy the key to use when calling the API

### Create the API Gateway

1. Go to [API gateway](https://console.cloud.google.com/api-gateway/api)
2. Create an API Gateway
   1. API Name: set a descriptive name, ex: `G2L Integration API`
   2. API ID: set the API ID, set the same id as the title of the OpenAPI 2 yaml specification, ex: `backend-apigateway-secure-v1`
3. Setup the API config
   1. Upload the OpenApi 2 spec in yaml format
   2. API config name, ex: `G2L integration config`
   3. Select a service account with permissions to invoke Cloud Run services
4. Setup Gateway details:
   1. Gateway name, ex: `G2L Integration Gateway`
   2. Select the region, ex: `europe-west1`
5. Create the API gateway
6. Wait until the creation is finished, can last several minutes
7. Click in the API Gateway to see the details, in "Gateways" tab, copy the external URL generated by Google, ex: `https://g2l-integration-gateway-xxxxxxxxxxx.ew.gateway.dev`
8. Copy the managed service name so secure the API with a key, ex: `g2l-integration-api-v1-xxxxxxxxxxx.apigateway.g2l-integration.cloud.goog`

Enable the managed service:

1. Go to [APIs and services -> Enabled APIs and services](https://console.cloud.google.com/apis/dashboard), search by the Cloud Run Service managed service name, and enable it. it can be enabled with the command `gcloud services enable g2l-integration-api-v1-xxxxxxxxxxx.apigateway.g2l-integration.cloud.goog`

## Test the service

Try to acces the Cloud Run Service directly, it will fail becaouse we need an authentication:

```bash
curl https://g2l-test-server-xxxxxxxxxxx-uc.a.run.app/v1/products

<html><head>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
<title>403 Forbidden</title>
</head>
<body text=#000000 bgcolor=#ffffff>
<h1>Error: Forbidden</h1>
<h2>Your client does not have permission to get URL <code>/</code> from this server.</h2>
<h2></h2>
</body></html>

```

Try to access the Cloud Run Service directly with an Oauth2 token, it will succeed because we use OAuth authentication:

```bash
# Configure the project and region
gcloud config set project g2l-integration
gcloud config set run/region europe

# Login with a user with privileges to execute the service 
gcloud auth login

# Generate the token
TOKEN=$(gcloud auth print-identity-token)

# Call the service with the token
curl -H "Authorization: Bearer $TOKEN" -H 'Content-Type: text/plain' -d '**Hello Bold Text**' https://onion-g2l-test-server-xxxxxxxxxxx-uc.a.run.app/v1/products
[ ... ]
```

Try to access the service through the API Gateway the API with no authentication (without key in header o query), it will return an error:

```bash
curl https://g2l-integration-gateway-xxxxxxxxxxx.ew.gateway.dev/v1/products
{"code":401,"message":"UNAUTHENTICATED:Method doesn't allow unregistered callers (callers without established identity). Please use API Key or other form of API consumer identity to call this API."}
```

Try to access the API with the API key support disabled, it will return an error:

```bash
curl --location --request GET 'https://g2l-integration-gateway-xxxxxxxxxxx.ew.gateway.dev/v1/products' \
--header 'Content-Type: application/json' \
--header 'x-api-key: KKKKKKKKKKKKKKKKKKKKKKKK'
{"message":"UNAUTHENTICATED:Method doesn't allow unregistered callers (callers without established identity). Please use API Key or other form of API consumer identity to call this API.","code":401}
```

Try to access the API with a wrong key, it will return an error:

```bash
curl --location --request GET 'https://g2l-integration-gateway-xxxxxxxxxxx.ew.gateway.dev/v1/products' --header 'Content-Type: application/json' --header 'x-api-key: EEEEEEEEEEEEEEEEEEEEE'
{"code":400,"message":"INVALID_ARGUMENT:API key not valid. Please pass a valid API key."}
```

Try to access with the correct key:

```bash
curl --location --request GET 'https://g2l-integration-gateway-xxxxxxxxxxx.ew.gateway.dev/v1/products' --header 'Content-Type: application/json' --header 'x-api-key: KKKKKKKKKKKKKKKKKKKKKKKK'
[ ... ]
```

## Next steps

Now that we have a service that can be called through an API Gateway, it's time to enhance security and analyze the logs.

Keep in mind that API KEY authentication is vulnerable to a man-in-the-middle attack. To reduce this vulnerability, use OAuth2 authentication

To grant more security [secure the APY key with restrictions](https://cloud.google.com/docs/authentication/api-keys#creating_an_api_key)
