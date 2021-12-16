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
          BASE_URL: 'api.knawat.io'
          DNS_ZONE_NAME: 'knawat-io'
          docker-registry: 'gcr.io'
          IMAGE_REPO_NAME: 'api'
          GCP_PROJECT: 'knawat-app'
          GCP_JSON_KEY: ${{ secrets.GCP_JSON_KEY }}
          SERVICES: "{api,products}"
          IMAGE_TAGS: "$GITHUB_SHA" 
          cluster_name: "cluster-dev"
          cluster_location: "us-central1-c"
          CONTAINER_NAME: "app"
          MOLECULER_APM_ENABLE: "0"
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Usage

If the SERVICES conatins API service, it will create:
1- External IP: "RepoName"-"PR_Number"-pr
    so, for example: I'm working on a repo name: "knawat/app" and I created a PR (263), the IP will create with the following name: app-263-pr
2- DNS: pr-"PR_Number"."BASE_URL"
    so, if I set the BASE_URL=api.knawat.io in the env, the DNS will create with the following name: pr-263.api.knawat.io and it's point to the previous IP address.
3- For moleculer helm package will create the ingress.