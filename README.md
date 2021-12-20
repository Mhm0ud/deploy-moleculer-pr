# GitHub Action for deploy Pull Requests in Google Kubernetes Service for Moleculer App
      
## Description
A Github Action to deploy previews of Pull Requests to GKE via Moleculer Helm Package 🚀

Deploy previews
 - build docker image + tag with meta preview tags
 - publish docker image to container registry
 - create an external IP address
 - create a DNS.
 - deploy chart / preview in Kubernetes
 - add a preview comment to pull request with link to preview

## Prerequisite
This action is (currently) tightly coupled to the following set of tools. 
- Kubernetes cluster (GKE)
- GCP token for authentication
- google registery as container registry (could work with others, but not tested)
- Domain zone hosted on Google cloud
- Moleculer app
- Dockerfile for the application

## Example deployment

```yaml
name: "deploy-pr"
on:
  pull_request:
    types: [opened, edited, reopened, synchronize]
  
jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: deploy-pr
    steps:
      - uses: knawat/github-actions-deploy-moleculer@main
        with: 
          #APPLICATION VARIABLE
          SERVICES: {api,products}
          SERVICEDIR: build/services
          MOLECULER_APM_ENABLE: 1
          AGENT_TOKEN: ${{ secrets.AGENT_TOKEN }}
          AGENT_APIKEY: ${{ secrets.AGENT_APIKEY }}
          TRANSPORTER: tcp
          MONGO_URI: '${{ secrets.MONGO_URI }}' //should include it between ''
          BASE_URL: api.example.com //The gateway URL of your app, the deployment PR URL will be 'pr-[PR_NUMBER].[BASE_URL]'
          HELM_SET: ${{ secrets.HELM_SET }} //To include any other application env var (environment.env.VAR1=VAL1)
          # GCP Configuration 
          DNS_ZONE_NAME: 'EXAMPLE-COM' //It's the cloud DNS Zone name not the DNS name (P.S. DNS name is example.com)
          GCP_PROJECT: 'example-project'
          GCP_JSON_KEY: ${{ secrets.GCP_JSON_KEY }}
          #DOCKER CONFIGURATION
          DOCKER_REGISTRY: 'gcr.io'
          IMAGE_REPO_NAME: 'api'
          IMAGE_TAGS: "$GITHUB_SHA" 
          #K8S Configuration
          CLUSTER_NAME: "cluster-dev"
          CLUSTER_LOCATION: "us-central1-c"
          CONTAINER_NAME: "app"
          #GitHub Configuration
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Usage

If the SERVICES conatins API service, it will create:
1. External IP: "RepoName"-"PR_Number"-pr
    so, for example: I'm working on a repo name: "knawat/app" and I created a PR (263), the IP will create with the following name: app-263-pr
2. DNS: pr-"PR_Number"."BASE_URL"
    so, if I set the BASE_URL=api.knawat.io in the env, the DNS will create with the following name: pr-263.api.knawat.io and it's point to the previous IP address.
3. For moleculer helm package will create the ingress.

If you need to add environment variables to your application you can use HELM_SET variable:

```yaml
 with: 
     HELM_SET: environment.env.VAR1=VAL1,environment.env.VAR2=VAL2
```

If you need to enable molecler Lab:

```yaml
 with: 
     MOLECULER_APM_ENABLE: 1
     AGENT_TOKEN: YourApiToken
     AGENT_APIKEY: YourApiKey
```

If you need to change the transporter use by the application:

```yaml
 with: 
     TRANSPORTER: nats://nats:4222
```

## Configuration

| Variable 	| Description 	| Default value 	| Required 	|
|---	|---	|---	|---	|
| DOCKER_FILE 	| The location of your docker file to build the image, usually it's in the root directory of your application. 	| Dockerfile 	| NO 	|
| DOCKER_REGISTRY 	| Docker image registry. where you want to save your build image. Defaults to GCP container registry 	| gcr.io 	| YES 	|
| IMAGE_REPO_NAME 	| Image repo name in the image registery 	| myImageRepo 	| YES 	|
| IMAGE_TAGS 	| Your build image tag. it's good to use $GITHUB_SHA 	| latest 	| NO 	|
| CONTAINER_NAME 	| The container name that should deploy to the k8s cluster 	| myApp 	| NO 	|
| TRANSPORTER 	| he transporter for the moleculer application 	| tcp 	| NO 	|
| SERVICES 	| The name of the moleculer services you want to deploy, it  should between {} 	| '' 	| NO 	|
| SERVICEDIR 	| The name of the moleculer services directory 	| build/services 	| NO 	|
| MONGO_URI 	| The DB connection string 	| '' 	| YES 	|
| BASE_URL 	| Your app base URL 	| api.example.com,Let's assume that your app gateway URL is 'api.example.com', and you want to create a PR deployment using this action, so the deployment PR URL will be 'pr-[PR_NUMBER].[BASE_URL]'  (pr-1.api.example.com)  	| YES 	|
| CRON_ENABLED 	| If you have a cron in your application and you want to add it to moleculer helm package, you can enable it here and add it's configuration in HELM_SET. 	| false 	| NO 	|
| MOLECULER_APM_ENABLE 	| The moleculer laboratory for your application. 	| 0 	| NO 	|
| AGENT_TOKEN 	| The moleculer lab agent token for the moleculer laboratory. 	| someSecret 	| NO 	|
| AGENT_APIKEY 	| The moleculer lab API key for the moleculer laboratory 	| someSecret 	| NO 	|
| HELM_SET 	| Additional helm values to set environment variables (corresponds to `helm upgrade --set`). Should have format environment.env.VAR1=VAL1,environment.env.VAR2=VAL2. 	|  	| NO 	|
| GCP_PROJECT 	| The ID of your project on Google Cloud. 	| example-project 	| YES 	|
| PORTGCP_JSON_KEY 	| The auth token to authenticate with your GCloud Project. 	|  	| YES 	|
| CLUSTER_NAME 	| The K8S cluster name. 	| example-cluster 	| YES 	|
| CLUSTER_LOCATION 	| The K8S cluster location on GCP. 	| us-central-01 	| YES 	|
| DNS_ZONE_NAME 	| Your DNS Zone Name in GCP Cloud DNS,It's the cloud DNS Zone name not the DNS name (P.S. DNS name is example.com)" 	| xample-com 	| YES 	|
| GITHUB_TOKEN 	| Your GitHub token, you can get it from this GitHub environment variable ${{ secrets.GITHUB_TOKEN }} by default. 	|  	| YES 	|

## Default environment variables of moleculer helm package

| Variable 	| Default Value 	|
|---	|---	|
| NODE_ENV 	| development 	|
| PORT 	| 3000 	|
| LOGGER 	| Laboratory 	|
| LOGLEVEL 	| info 	|
| CACHER 	| memory 	|

For reference you can visit the helm package repo from [here](https://github.com/Knawat/helm-charts).