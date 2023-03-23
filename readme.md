### Overview

This project provides a standard installation files for both Calico and ArgoCD for the various clusters.

Files are checked in so we have consistent installations across all four clusters. I'm also not a fan of pulling and installing directly from the internet as the time between working on the clusters may introduce a version change. Pull the files and replace these with new files

For Calico, the two yaml files are installed on every cluster after kubeadm has been done and the cluster is fully joined.

    cd calico
    kubectl create -f tigera-operator.yaml
    kubectl create -f custom-resources.yaml

For Argocd, run the argocd.yaml file to create the namespace, then apply the install.yaml file. This is only installed on the development cluster as 
it will be pushing updates to all four clusters as things change.

    cd argocd
    kubectl create -f argocd.yaml
    kubectl create -f install.yaml


### Environments

Currently there are no files however I've created four environments; dev, qa, stage, prod, and in each directory are configuration directories.

    dev
      configmaps
      deployments
      namespaces
      secrets
      services
    qa
    stage
    prod

More may be added as time advances.


### Projects

I only have one project right now that I'll be using this for, the llamas website. It's a simple single page so should be a good starting point.

