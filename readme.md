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

Note that the metrics server is set to retrieve certificates via InternalIP,ExternalIP,Hostname (kubelet flags). Since the certs are created with the hostnames and not IPs, it means the metrics server won't start. To fix it, you need to do two things:

Basically add the serverTLSBootstrap: true line to the kubelet-config configmap after the 'kind: KubeletConfiguration' line.

    kubectl edit configmap kubelet-config -n kube-system

Then in the same place on all the servers, in the /var/lib/kubelet/config.yaml file. Then restart kubelet.

When done, get a list of certificate signing requests (csrs)

```
$ kubectl get csr
NAME        AGE     SIGNERNAME                                    REQUESTOR                                      REQUESTEDDURATION   CONDITION
csr-76kdv   8m45s   kubernetes.io/kubelet-serving                 system:node:bldr0cuomknode1.dev.internal.pri   <none>              Pending
csr-gblwb   12m     kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:21v04z                        <none>              Approved,Issued
csr-hgz6g   8m6s    kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:21v04z                        <none>              Approved,Issued
csr-mzsqd   7m26s   kubernetes.io/kubelet-serving                 system:node:bldr0cuomknode3.dev.internal.pri   <none>              Pending
csr-nhd9n   13m     kubernetes.io/kubelet-serving                 system:node:bldr0cuomkube1.dev.internal.pri    <none>              Pending
csr-nr67g   8m59s   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:21v04z                        <none>              Approved,Issued
csr-nzkb9   11m     kubernetes.io/kubelet-serving                 system:node:bldr0cuomkube2.dev.internal.pri    <none>              Pending
csr-p4n4g   13m     kubernetes.io/kubelet-serving                 system:node:bldr0cuomkube1.dev.internal.pri    <none>              Pending
csr-rknd8   7m27s   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:21v04z                        <none>              Approved,Issued
csr-sdwwb   7m50s   kubernetes.io/kubelet-serving                 system:node:bldr0cuomknode2.dev.internal.pri   <none>              Pending
csr-sft7n   8m19s   kubernetes.io/kubelet-serving                 system:node:bldr0cuomkube3.dev.internal.pri    <none>              Pending
csr-swjmh   13m     kubernetes.io/kube-apiserver-client-kubelet   system:node:bldr0cuomkube1.dev.internal.pri    <none>              Approved,Issued
csr-t8hvw   8m33s   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:21v04z                        <none>              Approved,Issued
```

Approve them:

```
for i in `kubectl get csr | grep Pending | awk '{print $1}'`
do
  kubectl certificate approve $i
done
```

And verify all are approved.

```
$ kubectl get csr
NAME        AGE     SIGNERNAME                                    REQUESTOR                                      REQUESTEDDURATION   CONDITION
csr-76kdv   9m24s   kubernetes.io/kubelet-serving                 system:node:bldr0cuomknode1.dev.internal.pri   <none>              Approved,Issued
csr-gblwb   12m     kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:21v04z                        <none>              Approved,Issued
csr-hgz6g   8m45s   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:21v04z                        <none>              Approved,Issued
csr-mzsqd   8m5s    kubernetes.io/kubelet-serving                 system:node:bldr0cuomknode3.dev.internal.pri   <none>              Approved,Issued
csr-nhd9n   14m     kubernetes.io/kubelet-serving                 system:node:bldr0cuomkube1.dev.internal.pri    <none>              Approved,Issued
csr-nr67g   9m38s   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:21v04z                        <none>              Approved,Issued
csr-nzkb9   12m     kubernetes.io/kubelet-serving                 system:node:bldr0cuomkube2.dev.internal.pri    <none>              Approved,Issued
csr-p4n4g   14m     kubernetes.io/kubelet-serving                 system:node:bldr0cuomkube1.dev.internal.pri    <none>              Approved,Issued
csr-rknd8   8m6s    kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:21v04z                        <none>              Approved,Issued
csr-sdwwb   8m29s   kubernetes.io/kubelet-serving                 system:node:bldr0cuomknode2.dev.internal.pri   <none>              Approved,Issued
csr-sft7n   8m58s   kubernetes.io/kubelet-serving                 system:node:bldr0cuomkube3.dev.internal.pri    <none>              Approved,Issued
csr-swjmh   14m     kubernetes.io/kube-apiserver-client-kubelet   system:node:bldr0cuomkube1.dev.internal.pri    <none>              Approved,Issued
csr-t8hvw   9m12s   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:21v04z                        <none>              Approved,Issued
```

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

