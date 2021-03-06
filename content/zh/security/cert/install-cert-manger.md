---
title: "安装 cert-manager"
weight: 10
---

参考官方文档: [https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html](https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html)

介绍几种安装方式，不管是用哪种我们都先规划一下使用哪个命名空间，推荐使用 `cert-manger` 命名空间，如果使用其它的命名空间需要做些更改，会稍微有点麻烦，先创建好命名空间:

```bash
kubectl create namespace cert-manager
```

cert-manager 部署时会生成 [ValidatingWebhookConfiguration](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) 来注册 \[ValidatingAdmissionWebhook\]\)\(ValidatingAdmissionWebhook\) 来实现 CRD 校验，而`ValidatingWebhookConfiguration` 里需要写入 `cert-manager` 自身校验服务端的证书信息，就需要在自己命名空间创建 `ClusterIssuer` 和 `Certificate` 来自动创建证书，创建这些 CRD 资源又会被校验服务端校验，但校验服务端证书还没有创建所以校验请求无法发送到校验服务端，这就是一个鸡生蛋还是蛋生鸡的问题了，所以我们需要关闭 `cert-manager` 所在命名空间的 CRD 校验，通过打 label 来实现:

```bash
kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true
```

## 使用原生 yaml 资源安装

直接执行 `kubectl apply` 来安装:

```bash
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.11.0/cert-manager.yaml
```

> 使用 `kubectl v1.15` 及其以下的版本需要加上 `--validate=false`，否则会报错。

## 使用 Helm 安装

安装 CRD:

```bash
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml
```

添加 repo:

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

执行安装:

```bash
helm install \
  --name cert-manager \
  --namespace cert-manager \
  --version v0.11.0 \
  jetstack/cert-manager
```

## 校验是否安装成功

检查 cert-manager 相关的 pod 是否启动成功:

``` bash
$ kubectl get pods --namespace cert-manager

NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-cainjector-74bb68d67c-4vplr   1/1     Running   0          101s
cert-manager-f7f8bf74d-j4hcz               1/1     Running   0          101s
cert-manager-webhook-665df569b5-5p6x8      1/1     Running   0          101s
```
