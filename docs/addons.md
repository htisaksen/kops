## Installing Kubernetes Addons

With kops you manage addons by using kubectl.

(For a description of the addon-manager, please see [addon_manager.md](addon_manager.md).)

Addons in Kubernetes are traditionally done by copying files to `/etc/kubernetes/addons` on the master.  But this
doesn't really make sense in HA master configurations.  We also have kubectl available, and addons are just a thin
wrapper over calling kubectl.

This document describes how to install some common addons.

### Dashboard

The [dashboard project](https://github.com/kubernetes/dashboard) provides a nice administrative UI:

Install using:
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/kubernetes-dashboard/v1.8.3.yaml
```

And then navigate to `https://api.<clustername>/ui`

(`/ui` is an alias to `https://<clustername>/api/v1/proxy/namespaces/kube-system/services/kubernetes-dashboard`)

The login credentials are:

* Username: `admin`
* Password: get by running `kops get secrets kube --type secret -oplaintext` or `kubectl config view --minify`

#### RBAC

For k8s version > 1.6 and [RBAC](https://kubernetes.io/docs/admin/authorization/rbac/) enabled it's necessary to add your own permission to the dashboard. Please read the [RBAC](https://kubernetes.io/docs/admin/authorization/rbac/) docs before applying permissions. 

Below you see an example giving **full access** to the dashboard.

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
  ```

### Monitoring with Heapster - Standalone

Monitoring supports the horizontal pod autoscaler.

Install using:
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/monitoring-standalone/v1.11.0.yaml
```
Please note that [heapster is deprecated](https://github.com/kubernetes/heapster/blob/master/docs/deprecation.md). Consider using [metrics-server](https://github.com/kubernetes-incubator/metrics-server) and a third party metrics pipeline to gather Prometheus-format metrics instead.

### Monitoring with Prometheus Operator + kube-prometheus

The [Prometheus Operator](https://github.com/coreos/prometheus-operator/) makes the Prometheus configuration Kubernetes native and manages and operates Prometheus and Alertmanager clusters. It is a piece of the puzzle regarding full end-to-end monitoring.

[kube-prometheus](https://github.com/coreos/prometheus-operator/blob/master/contrib/kube-prometheus) combines the Prometheus Operator with a collection of manifests to help getting started with monitoring Kubernetes itself and applications running on top of it.

```console
kubectl apply -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/prometheus-operator/v0.19.0.yaml
```

### Route53 Mapper

Please note that kops installs a Route53 DNS controller automatically (it is required for cluster discovery).
The functionality of the route53-mapper overlaps with the dns-controller, but some users will prefer to
use one or the other.
[README for the included dns-controller](https://github.com/kubernetes/kops/blob/master/dns-controller/README.md)

route53-mapper automates creation and updating of entries on Route53 with `A` records pointing
to ELB-backed `LoadBalancer` services created by Kubernetes. Install using:

The project is created by wearemolecule, and maintained at
[wearemolecule/route53-kubernetes](https://github.com/wearemolecule/route53-kubernetes).
[Usage instructions](https://github.com/kubernetes/kops/blob/master/addons/route53-mapper/README.md)

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/route53-mapper/v1.3.0.yml
```
