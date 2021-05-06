# A quick braindump on how I got started w/ bpftrace
For all information head out to https://github.com/iovisor/bpftrace

## Get your hands on a K8S cluster
I'm using a self-managed kubeadm cluster using Ubuntu and the latest version of K8S.

## Verify cluster access
Always good practice to verify you have access to the correct cluster and your nodes are in a `ready` state
```
export KUBECONFIG=$PWD/kubeconfig.yaml 
kubectl get nodes
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
Update following command with the node name
```
kubectl trace run demo-poolebpftracedemo-8ryo2  -e "tracepoint:syscalls:sys_enter_* { @[probe] = count(); }"
```
You should see someting like
```
$ kubectl trace run demo-poolebpftracedemo-8ryo2  -e "tracepoint:syscalls:sys_enter_* { @[probe] = count(); }"
trace 4f4ea9e4-ae68-11eb-a952-061a9c98df32 created
```

### Clone the bpftrace repo
To get some examples, clone this repo
```
https://github.com/iovisor/bpftrace.git
```
### Capturing pods 
```
$ kubectl trace run demo-poolebpftracedemo-8ryo2 -f ./bpftrace/tools/tcpconnect.bt
trace 2b25bf44-ae69-11eb-89a6-061a9c98df32 created
 
```
$ kubectl get po
NAME                                                       READY   STATUS      RESTARTS   AGE
kubectl-trace-2b25bf44-ae69-11eb-89a6-061a9c98df32-pczj7   0/1     Completed   0          41s
kubectl-trace-4f4ea9e4-ae68-11eb-a952-061a9c98df32-28wjk   0/1     Completed   0          6m50s
```
```
$ kubectl attach -it kubectl-trace-2b25bf44-ae69-11eb-89a6-061a9c98df32-pczj7
...
```

