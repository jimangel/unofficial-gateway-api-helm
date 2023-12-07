# unofficial-gateway-api-helm
A helm chart for managing Gateway API (https://gateway-api.sigs.k8s.io/implementations/) on any provider.

It does two things:
- generates a Gateway object (for baremetal and GKE)
- dynamically generates HTTPRoutes (while waiting on OSS chart adoption of Gateway API)

By default it deploys a gateway (disable: `--set gateway.enabled=false`) and does NOT deploy any HTTPRoute(s).

To deploy HTTP routes update and pass a `values.yaml` file.

## TODO:

- more about "why" (CI automation, BYOgateway API)
- MVP (dynamic gateway for local deployments, dynamic http routes for other, update `cd`)
- add in HPA (from templates)
- add in example for full OSS deployment
- clean up
- update api versions in routes to use helm value vs. range value...

## setup

Assumes you have installed Gateway API CRDs (https://gateway-api.sigs.k8s.io/guides/#installing-gateway-api) and/or a corresponding implementation (https://gateway-api.sigs.k8s.io/implementations/)



## use

```
export GAR_NAME=helm-charts
export PROJECT=out-of-pocket-cloudlab
helm template myrelease oci://us-central1-docker.pkg.dev/$PROJECT/$GAR_NAME/unofficial-gateway-api --version 0.1.0

or

helm template <FAKE-NAME> ./charts

helm pull oci://us-central1-docker.pkg.dev/$PROJECT/$GAR_NAME/unofficial-gateway-api --version 0.1.0

helm template myrelease oci://us-central1-docker.pkg.dev/$PROJECT/$GAR_NAME/unofficial-gateway-api --version 0.1.0


```

## artifactory/helm repo creation

Building the charts:

```
helm create charts
```

Creating charts with help:

```
# find config
kubectl get crd | grep gateway
kubectl describe crd gateways.gateway.networking.k8s.io
# use LLM to generate starting template / values and iterate.
# kubectl describe crd httproutes.gateway.networking.k8s.io
```

## setup artifactory

See what's already out there...

```
gcloud artifacts repositories list
```



Enable API:

```
gcloud services enable artifactregistry.googleapis.com
```

Create repo:

```
export GAR_NAME=helm-charts
export LOCATION=us-central1
export PROJECT=out-of-pocket-cloudlab

gcloud artifacts repositories create $GAR_NAME --repository-format=docker \
--location=us-central1 --description="Creating an OCI helm chart repo"
```

Make public:

```
gcloud artifacts repositories add-iam-policy-binding $GAR_NAME \
--location=$LOCATION --member=allUsers --role=ROLE

gcloud artifacts repositories add-iam-policy-binding $GAR_NAME --location=$LOCATION --member=allUsers --role=roles/artifactregistry.reader
```

## local debug

```
helm template ./charts

```

## deploy/update chart

Configure container runtime

```
# login 
gcloud auth configure-docker us-central1-docker.pkg.dev
```

Package chart for publishing (mainly from: https://cloud.google.com/artifact-registry/docs/helm/store-helm-charts)

```
helm package charts/
```

Output similar to:

```
Successfully packaged chart and saved it to: /Users/jimangel/unofficial-gateway-api-helm/unofficial-gateway-api-0.1.0.tgz
```

```
helm push unofficial-gateway-api-0.1.1.tgz oci://us-central1-docker.pkg.dev/$PROJECT/$GAR_NAME
```

Test:

```
helm pull oci://us-central1-docker.pkg.dev/$PROJECT/$GAR_NAME/unofficial-gateway-api --version 0.1.1

helm template myrelease oci://us-central1-docker.pkg.dev/$PROJECT/$GAR_NAME/unofficial-gateway-api --version 0.1.1 --set gateway.enabled=true | kubectl apply --dry-run=client -f -
```

Clean up:

```
rm -rf unofficial-gateway-api-0.1.0.tgz
```