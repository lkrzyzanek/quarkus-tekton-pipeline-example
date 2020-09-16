# quarkus-tekton-pipeline-example
Example of Tekton pipeline for Quarkus simple project

The goal is to use purely official tekton tasks and just configure the pipeline.

## Pipeline design

The pipeline consists of these steps:

1. Clone git repo (git-clone)
2. Compile project (maven task)
3. Run tests (maven)
4. Build and push image to repository (maven task + quarkus container plugin)

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

### Tekton official tasks

```
# git-clone task
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/master/task/git-clone/0.2/git-clone.yaml
# maven task
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/master/task/maven/0.2/maven.yaml
```

## Pipeline Deploy
Apply all pipeline yamls.
1. persistent-volume-claims.yaml - contains volumes for pipeline
2. pipeline.yaml - build pipeline

```
kubectl apply -f pipeline
```
## Pipeline Run

via Tekton CLI:
```
tkn pipeline start build-pipeline \
 --param gitUrl='https://github.com/lkrzyzanek/quarkus-tekton-pipeline-example.git' \
 --param gitRevision='master' \
 --param contextDir='.' \
 --param imageGroup='test' \
 --param imageName='getting-started' \
 --workspace name=shared-workspace,claimName=shared-workspace \
 --workspace name=maven-settings,emptyDir="" \
 --workspace name=local-maven-repo,emptyDir="" \
 --showlog
```

and wait for the output like
```
PipelineRun started: build-pipeline-run-h7p2n
Waiting for logs to be available...
```
It ends classic maven output and these two lines specify where the image was pushed 
```
[build-push-image : mvn-goals] [INFO] [io.quarkus.container.image.jib.deployment.JibProcessor] Container entrypoint set to [java, -Djava.util.logging.manager=org.jboss.logmanager.LogManager, -cp, /app/resources:/app/classes:/app/libs/*, io.quarkus.runner.GeneratedMain]
[build-push-image : mvn-goals] [INFO] [io.quarkus.container.image.jib.deployment.JibProcessor] Pushed container image example.com/test/getting-started:1.0-SNAPSHOT (sha256:03b9eef5d8c123e8a58942533f7612b661660fb7692d7751e802afbc29827946)
```

You can also check details via:
```
tkn pipelinerun describe build-pipeline-run-h7p2n
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