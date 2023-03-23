### Overview

This project provides a standard installation files for both Calico and ArgoCD for the various clusters.

Files are checked in so we have consistent installations across all four clusters. I'm also not a fan of pulling and installing directly from the internet as the time between working on the clusters may introduce a version change. Pull the files and replace these with new files

Because we want to load images from the local repositor, you'll need to review the yaml files and download them on the docker server, then push it to the 
local repos. Also for security reasons, change the Image Download option to Always.

For Calico, the two yaml files are installed on every cluster after kubeadm has been done and the cluster is fully joined.

First pull the images locally.

    docker pull quay.io/tigera/operator:v1.29.0

    docker tag quay.io/tigera/operator:v1.29.0 bldr0cuomrepo1.dev.internal.pri:5000/operator:v1.29.0

    docker push bldr0cuomrepo1.dev.internal.pri:5000/operator:v1.29.0

Then update the tigera-operator.yaml file:

    cd calico
    kubectl create -f tigera-operator.yaml
    kubectl create -f custom-resources.yaml


For Argocd, run the argocd.yaml file to create the namespace, then apply the install.yaml file. Make sure you add the new namespace when running the install.yaml file. This is only installed on the development cluster as it will be pushing updates to all four clusters as things change.

First pull the images locally. I'm using docker directory but you can do it with Artifactory as well.

    docker pull ghcr.io/dexidp/dex:v2.35.3
    docker pull quay.io/argoproj/argocd:v2.6.6
    docker pull redis:7.0.8-alpine

    docker tag ghcr.io/dexidp/dex:v2.35.3      bldr0cuomrepo1.dev.internal.pri:5000/dex:v2.35.3
    docker tag quay.io/argoproj/argocd:v2.6.6  bldr0cuomrepo1.dev.internal.pri:5000/argocd:v2.6.6
    docker tag redis:7.0.8-alpine              bldr0cuomrepo1.dev.internal.pri:5000/redis:v7.0.8-alpine

    docker push bldr0cuomrepo1.dev.internal.pri:5000/dex:v2.35.3
    docker push bldr0cuomrepo1.dev.internal.pri:5000/argocd:v2.6.6
    docker push bldr0cuomrepo1.dev.internal.pri:5000/redis:v7.0.8-alpine

Then update the install.yaml file to replace the above images with the local image name and apply the changes.

    cd argocd
    kubectl create -f argocd.yaml
    kubectl create -f install.yaml -n argocd


For the Metrics Server, retrieve the components.yaml file from the matrics-server github account and apply it to all clusters.

Pull the images locally and push them to the local repo server.

    docker pull registry.k8s.io/metrics-server/metrics-server:v0.6.3

    docker tag registry.k8s.io/metrics-server/metrics-server:v0.6.3 bldr0cuomrepo1.dev.internal.pri:5000/metrics-server:v0.6.3

    docker push bldr0cuomrepo1.dev.internal.pri:5000/metrics-server:v0.6.3

Then update the components.yaml file 

    cd [environment]/metrics-server
    kubectl create -f components.yaml


### Environments

Currently there are no files however I've created four environments; dev, qa, stage, prod, and in each directory are configuration directories.

    dev
      configmaps
      deployments
      metrics-server
      namespaces
      policies
      secrets
      services
    qa
    stage
    prod

More may be added as time advances.


### Projects

I only have one project right now that I'll be using this for, the llamas website. It's a simple single page so should be a good starting point.

