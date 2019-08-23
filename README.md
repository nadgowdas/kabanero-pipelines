![](https://raw.githubusercontent.com/kabanero-io/kabanero-website/master/src/main/content/img/Kabanero_Logo_Hero.png)

# kabanero-pipelines
The kabanero-pipelines repository contains a collection of tasks and pipelines that are intended to work out of the box with the featured Kabanero collections.

# Manual pipeline execution

Login to your cluster.  For example for OKD,

```
oc login <master node IP>:8443
```

## Clone the kabanero pipelines repo

```
git clone https://github.com/kabanero-io/kabanero-pipelines
cd kabanero-pipelines
```

## Switch to working in the kabanero namespace
```
oc project kabanero
```

## Setup the appropriate permissions to run the pipelines on OKD clusters

```
cd ./pipelines/common
kubectl apply -f appsody-service-account.yaml
kubectl apply -f appsody-cluster-role-binding.yaml
```

## Create a secret with your docker credentials to publish the image to the docker repo and update the appropriate service account

```
apiVersion: v1
kind: Secret
metadata:
  name: my-docker-secret
  annotations:
    tekton.dev/docker-0: https://index.docker.io/v1/ 
type: kubernetes.io/basic-auth
stringData:
  username: <your docker userid>
  password: <your docker password>
```
```
kubectl apply -f my-docker-secret.yaml
kubectl edit serviceaccount kabanero-sa
```
Add the following:
```
secrets:
- name: my-docker-secret
```

## Update the pipeline-resources.yaml with github & docker repo info to create the PipelineResources

Sample pipeline-resources.yaml files are provided for each featured collection under the manual-pipeline-runs dir.  Update the docker-image URL.  You can use the sample github repo provided or update it to point to your github repo.

After updating the file, apply it

```
kubectl apply -f <collection-name>-pipeline-resources.yaml
```

## The featured collections should have activated the tasks and pipelines already.  If you are creating a new task or pipeline, activate them manually

```
kubectl apply -f <task.yaml>
kubectl apply -f <pipeline.yaml>
```

## Run the pipeline

Sample PipelineRun files are provided under ./pipelines/manual-pipeline-runs.  Locate the appropriate pipeline-run file and execute it.
```
kubectl apply -f <collection-name>-pipeline-run.yaml
```

## Check the status of the pipeline run

```
kubectl get pipelineruns
kubectl -n kabanero describe pipelinerun.tekton.dev/<pipeline-run-name> 
```

# Execute pipelines using Tekton Dashboard Webhook Extension

You can also leverage the Tekton Dashboard Webhook Extensions to drive the pipelines automatically by configuring webhooks to github.  Events such as commits in a github repo can be setup to automatically trigger pipeline runs.

Visit https://github.com/tektoncd/experimental/blob/master/webhooks-extension/docs/GettingStarted.md for instructions on configuring a webhook.
