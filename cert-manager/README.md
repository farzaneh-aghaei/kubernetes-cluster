# Cert-Manager Chart

cert-manager is a Kubernetes addon to automate the management and issuance of TLS certificates from various issuing sources.

[Overview of Cert Manager](https://github.com/cert-manager/cert-manager)

Trademarks: This software listing is packaged by Bitnami. The respective trademarks mentioned in the offering are owned by the respective companies, and use of them does not imply any affiliation or endorsement.

### Introduction
This chart bootstraps a [cert-manager](https://cert-manager.io/) Deployment in a [Kubernetes](https://kubernetes.io/) cluster using the Helm [Helm](https://helm.sh/) package manager.

Be sure never to embed cert-manager as a sub-chart of other Helm charts; cert-manager manages non-namespaced resources in your cluster and care must be taken to ensure that it is installed exactly once.

### Prerequisites

- Kubernetes 1.19+
- Helm v3.2.0+
- SC provides a way for administrators to describe the "classes" of storage

### Installing the Chart
Full installation instructions, including details on how to configure extra functionality in cert-manager can be found in the [installation docs](https://cert-manager.io/docs/installation/supported-releases/).

Before installing the chart, you must first install the cert-manager CustomResourceDefinition resources. This is performed in a separate step to allow you to easily uninstall and reinstall cert-manager without deleting your installed custom resources.
Install CRDs with kubectl with the [releases](https://github.com/cert-manager/cert-manager/releases) :
```
$ kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/{{RELEASE_VERSION}}/cert-manager.crds.yaml
```
To install the chart with the [releases](https://github.com/cert-manager/cert-manager/releases) name my-release:
```
$ helm repo add jetstack https://charts.jetstack.io
$ helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version {{RELEASE_VERSION}} -f ./cert-manager-values.yaml
```
Tip: List all releases using helm list

> **Note:** Complete guide of this helm repo is provided in [this link](https://cert-manager.io/docs/installation/helm).

## Upgrading the Chart

Special considerations may be required when upgrading the Helm chart, and these
are documented in our full [upgrading guide](https://cert-manager.io/docs/installation/upgrading/).

**Please check here before performing upgrades!**

## Uninstalling the Chart

To uninstall/delete the `cert-manager` deployment:

```
$ helm delete cert-manager
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

If you want to completely uninstall cert-manager from your cluster, you will also need to
delete the previously installed CustomResourceDefinition resources:

```
$ kubectl delete -f https://github.com/jetstack/cert-manager/releases/download/{{RELEASE_VERSION}}/cert-manager.crds.yaml
```

## Additional Information

When we wanted to deploy cert-manager for ingress, the problem is that it is going to use our CA to sign the requests but finds out that the CA is malformed , in this case we used the Ingress Default TLS in order to provide the wildcard SSL for the services , more details added on the ingress controller.

After the installation we create a new self-signed clusterissuer in order to use inside the cluster.

In order to add an Issuer you can apply the 'issuer.yaml' file. This will create an issuer in the cert-manager namespace. If you dont have a CA you can apply 'selfsigned-clusterissuer.yaml' that will use a selfsigned cert6ificate.

You can apply the 'cluster-issuer.yaml' to add a samle ClusterIssuer to your cluster which is cross namespace.

> **Note:**  Note that to use it you should apply a secret name that contains a CA or SelfSigned Certificate. An example of this CA and self-signed is provided in the repo. for the slefsigned secret you can apply 'selfsigned-secret.yaml' file.

To test the Issuer you can use a sample certificate requester by applying 'certificate.yaml' file. Choose 'issuerRef.kind: ClusterIssuer' or 'issuer' and also provide the namespace if you are using issuer as it is namespaced.


```
$ kubectl apply -f  selfsigned-clusterissuer.yaml -n cert-manager
clusterissuer.cert-manager.io/selfsigned-clusterissuer created

```
```
$ kubectl get secret  -n cert-manager
NAME                                  TYPE                                  DATA   AGE
cert-manager-cainjector-token-qwv55   kubernetes.io/service-account-token   3      1m56s
cert-manager-token-84ktf              kubernetes.io/service-account-token   3      1m46s
cert-manager-webhook-ca               Opaque                                3      2m22s
cert-manager-webhook-token-bldh6      kubernetes.io/service-account-token   3      4m36s
default-token-t2td9                   kubernetes.io/service-account-token   3      14m
sh.helm.release.v1.cert-manager.v1    helm.sh/release.v1                    1      5m16s
$ kubectl get all  -n cert-manager
NAME                                          READY   STATUS    RESTARTS   AGE
pod/cert-manager-64d679f55b-y5rc4             1/1     Running   0          5m7s
pod/cert-manager-cainjector-5994c55b6-rwvnj   1/1     Running   0          5m7s
pod/cert-manager-webhook-756c768b6c-d65dj     1/1     Running   0          5m7s

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/cert-manager           ClusterIP   1.1.1.102   <none>        9402/TCP   3m9s
service/cert-manager-webhook   ClusterIP   1.1.1.54    <none>        443/TCP    5m5s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cert-manager              1/1     1            1           5m5s
deployment.apps/cert-manager-cabnjector   1/1     1            1           5m5s
deployment.apps/cert-manager-webhook      1/1     1            1           5m5s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/cert-manager-64d679f55b             1         1         1       5m5s
replicaset.apps/cert-manager-cainjector-5494c55b6   1         1         1       5m5s
replicaset.apps/cert-manager-webhook-759c766b6c     1         1         1       5m5s
```
```
$ kubectl get clusteri
clusterinformations.crd.projectcalico.org  clusterissuers.cert-manager.io
```
```
$ kubectl get clusterissuer
NAME                       READY   AGE
selfsigned-clusterissuer   True    2m1s
```
