# Release Test via Jenkinsfile

## Overview
This project test
* Using The Single Jenkinsfile
* Build Multiple Branches
* and Deploy that to Multiple Kubernetes Clusters
* and Cloud handle pipelines via Envitionment Variables

## Environment Variables
```bash
### for Jenkins Agent Pod
#### Kubernetes
* KUBECONFIG_VOLUME=stg-kube-config
* SERVICE_ACCOUNT=default

### for Build
#### Gradle
* GRADLE_USER_HOME=/root/gradle/${JOB_NAME}
#### Docker
* DOCKER_CRED=DOCKER_CRED
* DOCKER_REGISTRY=registry-1.docker.io

### for Deploy
#### kubectl
* K8S_NAMESPACE='cicd'
* KUBECONFIG=/var/secret/kube-config
#### Skaffold
* SKAFFOLD_CMD=build
* SKAFFOLD_OPTS=--dry-run
```
