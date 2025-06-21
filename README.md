# openshift-cluster-gitops

This is the repository for configuring your fresh OpenShift cluster with all the operators needed to run a successful proof of concept. 

First step: Clone this repository. You will need to add additional customized manifests and configuration for your environment. This repository is designed to be a starter for a gitops-centric OpenShift POC. 

After forking, you will need to obtain a personal access token (PAT) to get read access to the repository. https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token

## Layers

This repository is designed as a GitOps App-of-Apps repository using sync waves to coordinate and orchestrate the layers. 

### OpenShift Gitops

The first layer is the actual OpenShift Gitops installation and configuration. We will not be utilizing the default OpenShift GitOps instance of ArgoCD, but rather creating a more customized one for our purposes. This is not abnormal as the default ArgoCD install is usually too restrictive for most enterprise environments. 

### Foundational Configurations

This layer is for the items needed prior to any Operator installs.

### Operators 

All the Operators needed for a solid POC cluster

## Instructions

### Login
   
```shell
oc login --server=https://api.clustername.domain:6443 -u kubeadmin -p <password>
```

### Install OpenShift Gitops 

```shell 
./install-openshift-gitops.sh
```

Don't trust the script. Go look at it. It has some neat tricks on how to programatically solve manifest synchronization. 


### Add Secrets

Add Credentials for the cloned Github respository containing these manifests to the `cluster-gitops` namespace

```shell
oc apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: openshift-cluster-gitops-credentials
  namespace: cluster-gitops
  labels:
    argocd.argoproj.io/secret-type: repository
type: Opaque
stringData:
  url: https://github.com/<org>/openshift-cluster-gitops.git
  username: <username>
  password: <github-pat>
  name: openshift-cluster-gitops
  project: cluster
EOF
```

### SED and Apply the Cluster ApplicationSet

In the `cluster-application-set.yaml` file, update the `repoURL` value to set it to your GitHub repository. After this is complete, you can apply the ApplicationSet.

```shell
oc apply -f openshift-cluster-gitops-application-set.yaml
```

### Storage

You need to add your storage into the folder. 