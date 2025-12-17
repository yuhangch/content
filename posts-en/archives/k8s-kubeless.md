---
id: bRk
title: Kubernetes Kubeless 部署函数（CentOS 8）
pubDate: 2020-02-17T08:01:59.000Z
isDraft: true
tags:
  - kubernetes
  - serverless
categories:
  - 笔记
---

Yesterday I saw the open‑source object storage project `MINio`. My graduation project has already solved the problem of imagery storage, and I’m very interested in the ecosystem around Amazon S3 and Lambda, so I decided to set up an open‑source service: a small service that automatically retrieves image header files after remote sensing imagery is uploaded.

Since the performance of Alibaba Cloud is average, I chose to install with `kubeadmin`.

## 1. Turn off SWAP

```bash
$ vim /etc/fstab   # Comment out the line with the UUID

#
# /etc/fstab
# Created by anaconda on Wed Dec 25 03:29:46 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
#UUID=e32cfa7a-df48-4031-8fdf-5eec92ee3039 /              xfs     defaults        0 0
```

## 2. Install Kubeadm, kubelet, kubectl

-   `kubeadm`: the command to bootstrap the cluster.
-   `kubelet`: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
-   `kubectl`: the command line util to talk to your cluster.

```bash
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

$ yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

$ systemctl enable --now kubelet
```

## 3. Initialize

```bash
$ kubeadm init <args>
$ kubead init --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' # Use a domestic mirror
```

## 4. Install a network plugin

```bash
# 1 Used during initialization
$ kubead init --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' --pod-network-cidr=192.168.0.0/16
# 2 Via kubeadm configuration
$ kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
```

## 5. Remove master node isolation

Control plane nodes are isolated by default, but I want to schedule `Pods` on this node, so I first `taint` this node.

```shell
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```

## 6. Add nodes

```shell
$ kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
# This is the command generated after running kubeadm init
```

## 7. Install kubeless

No real pitfalls; just follow the official documentation.

```shell
$ export RELEASE=$(curl -s https://api.github.com/repos/kubeless/kubeless/releases/latest | grep tag_name | cut -d '"' -f 4)
$ kubectl create ns kubeless
$ kubectl create -f https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml

$ kubectl get pods -n kubeless
NAME                                           READY     STATUS    RESTARTS   AGE
kubeless-controller-manager-567dcb6c48-ssx8x   1/1       Running   0          1h

$ kubectl get deployment -n kubeless
NAME                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubeless-controller-manager   1         1         1            1           1h

$ kubectl get customresourcedefinition
NAME                          AGE
cronjobtriggers.kubeless.io   1h
functions.kubeless.io         1h
httptriggers.kubeless.io      1h
```

```shell
$ export OS=$(uname -s| tr '[:upper:]' '[:lower:]')
$ curl -OL https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless_$OS-amd64.zip && \
  unzip kubeless_$OS-amd64.zip && \
  sudo mv bundles/kubeless_$OS-amd64/kubeless /usr/local/bin/
```

Right now GitHub is kind of half‑paralyzed; you can download it manually and then manually move the executable into the /usr/local/bin/ directory.

## 8. Test deploying functions

```shell
$ vi test.py # Write the following content
$ def hello(event, context):
  print event
  return event['data']
```

```shell
$ kubeless function deploy hello --runtime python2.7 \
                                --from-file test.py \
                                --handler test.hello
INFO[0000] Deploying function...
INFO[0000] Function hello submitted for deployment
INFO[0000] Check the deployment status executing 'kubeless function ls hello'
```

Check the status of the `functions`:

```shell
$ kubectl get functions
NAME         AGE
hello        1h

$ kubeless function ls
NAME            NAMESPACE   HANDLER       RUNTIME   DEPENDENCIES    STATUS
hello           default     helloget.foo  python2.7                 1/1 READY
```

If it succeeds:

```shell
$ kubeless function call hello --data 'Hello world!'
hello world！
```

## Issues

### 1. Pod status shows pending

```shell
$ kubectl describe pod POD-UID
$ kubectl describe pod -n kubeless # View detailed pod info, scroll down to the events section and check logs to troubleshoot
```

### 2. cannot get customresourcedefinitions.apiextensions.k8s.io at the cluster scope

This might be because you deployed a `non-rbac` version while `RBAC` is enabled on the `cluster`. You just need to switch back to the default `RBAC` version.

```shell
$ kubectl apply -f https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml
```

##