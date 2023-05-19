### Overview

This is part of the documentation for preparing and installing ArgoCD.

ArgoCD shall be installed only one one cluster and configured to make updates to all other clusters.


### ArgoCD Installation

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

