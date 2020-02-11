# Creating a KF Cluster

### Create a Kubernetes Cluster

Kubeflow works on **Kubernetes <=1.14**. Ensure you have a Kubernetes cluster with at most v1.14.10.

If you are trying this on your local machine, I highly recommend using KinD.<br>
[KinD](https://kind.sigs.k8s.io/) is a super-easy way to create clusters on local machines. You can get a cluster up and running in under **30s**.

Grab the latest (v0.7.0) version from [here](https://github.com/kubernetes-sigs/kind#installation-and-usage). <br>
We can create a Kubernetes v1.14.10 cluster in one single step:
```bash
kind create cluster --name kf-k8s --image kindest/node@sha256:81ae5a3237c779efc4dda43cc81c696f88a194abcc4f8fa34f86cf674aa14977
```

It is recommended to either re-tag this image for your own image or build your own because the image format might change in the near future.
You can get them from the release notes [here](https://github.com/kubernetes-sigs/kind/releases).

### Get kfctl
```bash
wget -O kfctl.tar.gz https://github.com/kubeflow/kfctl/releases/download/v1.0-rc.3/kfctl_v1.0-rc.2-13-g521fcfe_linux.tar.gz
tar -zvf kfctl.tar.gz
rm kfctl.tar.gz
chmod +x ./kfctl
mv ./kfctl /usr/local/bin/
# verify kfctl version
kfctl version
```
You can grab the macOS version of the binary from the kfctl [releases page](https://github.com/kubeflow/kfctl/releases).
### Quick check for pre-requisites
- Storage Class.

We need dynamic volume provisioning for getting PVs for the Kubeflow component PVCs. Check the Storage Class. <br>
```bash
alias k='kubectl'
k get sc
```

### Let's build a KF cluster!
```bash
export KFAPP=kf-k8s
mkdir -p ${KFAPP}
cd ${KFAPP}
wget -O kfctl_k8s.yaml \
https://raw.githubusercontent.com/kubeflow/manifests/v1.0-branch/kfdef/kfctl_k8s_istio.v1.0.0.yaml
kfctl build -V -f kfctl_k8s.yaml
# Delete the spartakus part if you want to opt out of giving anonymized usage statistics
kfctl apply -V -f kfctl_k8s.yaml
```

### Check KF cluster readiness
```bash
# 1. Check if all Istio pods are running
k get po -n istio-system
# 2. Check if all PVCs are Bound
k get pvc -n kubeflow
# 3. Check if all Kubeflow apps are running
k get po -n kubeflow
# 4. Check if the KF Access Management App is up
k get all -n kubeflow --selector=kustomize.component=profiles
```
