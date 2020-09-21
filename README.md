# quarkus-tekton-pipeline-example
Example of Tekton pipeline for Quarkus simple project

The goal is to use purely official tekton tasks and just configure the pipeline.

EDIT: The official maven task doesn't  provide a way for local maven repo


## Pipeline design

The pipeline consists of these steps:

1. Clone git repo (git-clone)
2. Compile project
3. Test projects
2. Build and push image to repository (maven task + quarkus container plugin)

## Prerequisites

### Minikube
Install Minikube: https://kubernetes.io/docs/tasks/tools/install-minikube/

Mac:
```
brew install minikube
```
Start minikube
```
minikube start
```

Add registry helpers
```
minikube addons enable registry && minikube addons enable registry-aliases
```

### Tekton
Install Tekton: https://github.com/tektoncd/pipeline/blob/master/docs/install.md

```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

Optionally install Tekton CLI: https://github.com/tektoncd/cli
or Tekton Dashboard: https://github.com/tektoncd/dashboard/blob/master/docs/install.md#installing-tekton-dashboard-on-kubernetes

### Tekton official tasks

```
# git-clone task
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/master/task/git-clone/0.2/git-clone.yaml
# maven task
# commented - used improved task in pipeline/task-maven-with-cache.yaml
# kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/master/task/maven/0.2/maven.yaml
```

## Pipeline Deploy
Apply all pipeline yamls.
1. persistent-volume-claims.yaml - contains volumes for pipeline
2. pipeline.yaml - build pipeline
3. task-maven-with-cache.yaml - maven tekton task with enabled caching

```
kubectl apply -f pipeline
```
## Pipeline Run

Pipeline with persistent volume claim cache which ensures sharing maven repository storage between pipeline runs 
```
kubectl apply -f pipeline/run/pipelinerun-pvc-cache.yaml
```

## Deploy app

```
kubectl create deployment getting-started --image=example.com/test/getting-started:1.0-SNAPSHOT
kubectl expose deployment getting-started --type=LoadBalancer --port=8080
minikube service getting-started
```
try the rest endpoint:
http://127.0.0.1:53430/hello

## Resources

* https://redhat-developer-demos.github.io/tekton-tutorial-staging/tekton-tutorial/setup.html
* https://quarkus.io/guides/container-image
* https://github.com/tektoncd/catalog/tree/master/task/buildpacks/0.1/