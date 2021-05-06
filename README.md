# A quick braindump on how I got started w/ bpftrace
For all information head out to https://github.com/iovisor/bpftrace

## Get your hands on a K8S cluster
I'm using a managed K8S cluster on DigitalOcean, but this should work on the majority of clusters that are around.

## Verify cluster access
Always good practice to verify you have access to the correct cluster and your nodes are in a `ready` state
```
export KUBECONFIG=$PWD/kubeconfig.yaml 
kubect get nodes
```

## Installing bpftrace
### Install 'krew'
Check out this page https://krew.sigs.k8s.io/docs/user-guide/setup/install/ to find more information in the 'krew' plugin for 'kubectl'
```
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.tar.gz" &&
  tar zxvf krew.tar.gz &&
  KREW=./krew-"${OS}_${ARCH}" &&
  "$KREW" install krew
)

export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

kubectl krew
```
### Installing bpftrace
```
sudo apt-get install -y bpftrace
```
