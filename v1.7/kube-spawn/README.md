## To Reproduce:

* Install Fedora 27 from https://cloud.fedoraproject.org/ (GP2 image) on AWS:
  - m4.large
  - Disk: at least 50GiB
* Copy kube-spawn v0.2.0 binaries from https://github.com/kinvolk/kube-spawn/releases/tag/v0.2.0 into the AWS EC2 instance:
  - cni-noop
  - cnispawn
  - kube-spawn-runc
  - kube-spawn
* Start a kube-spawn cluster:
```
sudo setenforce 0
sudo dnf install -y btrfs-progs git go iptables libselinux-utils polkit qemu-img systemd-container
mkdir go
export GOPATH=$HOME/go
go get -u github.com/containernetworking/plugins/plugins/...
export CNI_PATH=$GOPATH/bin
mkdir -p $GOPATH/src/github.com/kinvolk/kube-spawn
cp kube-spawn-runc $GOPATH/src/github.com/kinvolk/kube-spawn/kube-spawn-runc
sudo -E ./kube-spawn create --nodes=3
sudo -E ./kube-spawn start
export KUBECONFIG=/var/lib/kube-spawn/default/kubeconfig
KUBERNETES_VERSION=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
sudo curl -Lo /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/${KUBERNETES_VERSION}/bin/linux/amd64/kubectl
sudo chmod +x /usr/local/bin/kubectl
kubectl get nodes # repeat until the nodes are "Ready"
kubectl version > version.txt
```
* Run Sonobuoy with the command:

```
curl -L https://raw.githubusercontent.com/cncf/k8s-conformance/master/sonobuoy-conformance.yaml | kubectl apply -f -
```

* Wait until the sonobuoy pods are running:
```
kubectl get pods --all-namespaces
```
* Check the filename of the Sonobuoy report with `kubectl logs -f -n sonobuoy sonobuoy`
  - Visible with `Results available at /tmp/sonobuoy/2017....tar.gz`
* Copy it with `kubectl cp`
  - `kubectl cp sonobuoy/sonobuoy:/tmp/sonobuoy/2017....tar.gz 2017....tar.gz`
