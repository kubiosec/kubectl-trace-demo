# A quick braindump on how I got started w/ kubectl-trace (bpftrace)
For all information head out to https://github.com/iovisor/bpftrace and https://github.com/iovisor/kubectl-trace

## Get your hands on a K8S cluster
I'm using a self-managed kubeadm cluster using Ubuntu and the latest version of K8S.

## Verify cluster access
Always good practice to verify you have access to the correct cluster and your nodes are in a `ready` state
```
export KUBECONFIG=$PWD/kubeconfig.yaml 
kubectl get nodes
```

## Installing kubectl-trace
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
### Installing kubectl-trace
```
kubectl krew install trace
```
### Verify things are working
Select a K8S node you want to analyse
```
$ kubectl get no
NAME                           STATUS   ROLES    AGE   VERSION
demo-poolebpftracedemo-8ryo2   Ready    <none>   23m   v1.20.2
demo-poolebpftracedemo-8ryol   Ready    <none>   23m   v1.20.2
demo-poolebpftracedemo-8ryop   Ready    <none>   23m   v1.20.2
```

#### Setup a demo app
```
git clone https://github.com/xxradar/app_routable_demo.git
cd ./app_routable_demo
./setup.sh
watch kubectl get po -n app-routable-demo
```

### Set a helper variable for easy cli commands
Make sure you select a worker node to monitor the sample app
```
export NODE=demo-poolebpftracedemo-8ryo2
```

### Clone the bpftrace repo
To get some bpfprobe examples, clone this repo
```
git clone https://github.com/iovisor/bpftrace.git
```

### Capturing pods connections
#### Setup a bpfprobe (ex. monitoring TCP connect events)
```
$ kubectl trace run $NODE -f ./bpftrace/tools/tcpconnect.bt
trace 2b25bf44-ae69-11eb-89a6-061a9c98df32 created
```
```
$ kubectl get po
NAME                                                       READY   STATUS      RESTARTS   AGE
kubectl-trace-2b25bf44-ae69-11eb-89a6-061a9c98df32-pczj7   1/1     Running   0          41s
```
```
$ kubectl attach -it kubectl-trace-2b25bf44-ae69-11eb-89a6-061a9c98df32-pczj7
If you don't see a command prompt, try pressing enter.
13:11:15 7962     kubelet          10.11.2.126                             56608  10.11.2.126                             6443
13:11:16 7962     kubelet          10.11.2.126                             56610  10.11.2.126                             6443
13:11:17 7962     kubelet          10.11.2.126                             56612  10.11.2.126                             6443
13:11:18 7962     kubelet          127.0.0.1                               51342  127.0.0.1                               10259
13:11:18 7962     kubelet          10.11.2.126                             56616  10.11.2.126                             6443
13:11:19 7962     kubelet          10.11.2.126                             56618  10.11.2.126                             6443
13:11:20 7962     kubelet          10.11.2.126                             56620  10.11.2.126                             6443
13:11:21 7962     kubelet          127.0.0.1                               35742  127.0.0.1                               10257
13:11:21 7962     kubelet          10.11.2.126                             56624  10.11.2.126                             6443
13:11:22 7962     kubelet          127.0.0.1                               41082  127.0.0.1                               2381
13:11:22 7962     kubelet          10.11.2.126                             56628  10.11.2.126                             6443
13:11:23 7962     kubelet          10.11.2.126                             56630  10.11.2.126                             6443
...
```
You should see a list of connections being initiated by the pods on the K8S node
