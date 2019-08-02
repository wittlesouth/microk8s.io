---
layout: docs
title: "User Guide"
---

<h1 id="microk8s-documentation">MicroK8s documentation <img src="https://assets.ubuntu.com/v1/6731169e-certified-kubernetes-color.png?w=60" style="margin-left: 1rem; position: relative; top: -8px;" align="right"></h1>

Kubernetes in a [snap](https://snapcraft.io/microk8s) that you can run locally.

## User guide

Snaps are frequently updated to match each release of Kubernetes. The quickest way to get started is to install directly from the snap store. You can install MicroK8s and let it update to the latest stable upstream Kubernetes release with:

```
snap install microk8s --classic
```

Alternatively, you can select a MicroK8s channel that will follow a specific Kubernetes release series. For example, you install MicroK8s and let it follow the `v1.14` series with:

```
snap install microk8s --classic --channel=1.14/stable
```

You can read more on the MicroK8s release channels in the [Release Channels and Upgrades](/docs/release-channels) doc.

At any point you can check MicroK8s' availability with:

```
microk8s.status
```

During installation you can use the `--wait-ready` flag to wait for the kubernetes services to initialise:

```
microk8s.status --wait-ready
```

> In order to install MicroK8s make sure
> - you go through the [list of ports](/docs/ports) that need to be available

### Accessing Kubernetes

To avoid colliding with a `kubectl` already installed and to avoid overwriting any existing Kubernetes configuration file, MicroK8s adds a `microk8s.kubectl` command, configured to exclusively access the new MicroK8s install. When following instructions online, make sure to prefix `kubectl` with `microk8s.`.

```
microk8s.kubectl get nodes
microk8s.kubectl get services
```

If you do not already have a version of `kubectl` installed you can alias `microk8s.kubectl` to `kubectl` using the
following command:

```
snap alias microk8s.kubectl kubectl
```

This measure can be safely reverted at any time by running:

```
snap unalias kubectl
```
If you already have `kubectl` installed and you want to use it to access the MicroK8s deployment you can export the cluster's config with:

```
microk8s.kubectl config view --raw > $HOME/.kube/config
```

Note: The API server on port 8080 is listening on all network interfaces. In its kubeconfig file MicroK8s is using the loopback interface, as you can see with `microk8s.kubectl config view`. The `microk8s.config` command will output a kubeconfig with the host machine's IP (instead of the 127.0.0.1) as the API server endpoint.

### Kubernetes add-ons

MicroK8s installs a barebones upstream Kubernetes. This means just the `api-server`, `controller-manager`, `scheduler`, `kubelet`, `cni`, `kube-proxy` are installed and run. Additional services like `kube-dns` and `dashboard` can be run using the `microk8s.enable` command.

```
microk8s.enable dns dashboard
```

These add-ons can be disabled at anytime using the `disable` command

```
microk8s.disable dashboard dns
```

With `microk8s.status` you can see the list of available add-ons and which ones are currently enabled. You can find the add-on manifests and/or scripts under `${SNAP}/actions/`, with `${SNAP}` pointing by default to `/snap/microk8s/current`.

#### List of available add-ons
- **dns**: Deploy CoreDNS. This add-on may be required by others thus we recommend you always enable it. In environments where the external dns servers `8.8.8.8` and `8.8.4.4` are blocked you will need to update the upstream dns servers in `microk8s.kubectl -n kube-system edit configmap/coredns` after enabling the add-on.
- **dashboard**: Deploy Kubernetes dashboard as well as Grafana and InfluxDB. To access Grafana point your browser to the url reported by `microk8s.kubectl cluster-info`. Access the dashboard with the default token found with `microk8s.kubectl -n kube-system get secret` and `microk8s.kubectl -n kube-system describe secret default-token-{xxxx}`.
- **storage**: Create a default storage class. This storage class makes use of the hostpath-provisioner pointing to a directory on the host. Persistent volumes are created under `${SNAP_COMMON}/default-storage`. Upon disabling this add-on you will be asked if you want to delete the persistent volumes created.
- **ingress**: Create an ingress controller.
- **gpu**: Expose GPU(s) to MicroK8s by enabling the nvidia runtime and nvidia-device-plugin-daemonset. Requires NVIDIA drivers to be already installed on the host system.
- **istio**: Deploy the core [Istio](https://istio.io/) services. You can use the `microk8s.istioctl` command to manage your deployments.
- **registry**: Deploy a private image registry and expose it on `localhost:32000`. The storage add-on will be enabled as part of this add-on. See [the registry documentation](/docs/working) for more details.
- **metrics-server**: Deploy the [Metrics Server](https://kubernetes.io/docs/tasks/debug-application-cluster/core-metrics-pipeline/#metrics-server).
- **prometheus**: Deploy the [Prometheus Operator](https://github.com/coreos/prometheus-operator) v0.25.
- **fluentd**: Deploy the [Elasticsearch-Kibana-Fluentd](https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/) logging and monitoring solution.
- **jaeger**: Deploy the [Jaeger Operator](https://github.com/jaegertracing/jaeger-operator) v1.8.2 in the "simplest" configuration.
- **linkerd**: Deploy linkerd2 [Linkerd](https://linkerd.io/2/overview/) service mesh.  By default proxy auto inject
   is not enabled. To enable auto proxy injection, simply use `microk8s.enable linkerd:proxy-auto-inject`.
   If you need to pass more arguments, separate them with `;` and enclose the addons plus arguments with double quotes.
   For example:  `microk8s.enable "linkerd:proxy-auto-inject;tls=optional;skip-outbound-ports=1234,3456"`.
   Use `microk8s.linkerd` command to interact with Linkerd.
- **rbac**: Enable RBAC ([Role-Based Access Control](https://kubernetes.io/docs/reference/access-authn-authz/rbac/))
   authorisation mode. Note that other add-ons may not work with RBAC enabled.
- **knative**: Enable [Knative](https://knative.dev/) with `microk8s.enable knative`.

### Stopping and restarting MicroK8s

You may wish to temporarily shutdown MicroK8s when not in use without un-installing it.

MicroK8s can be shutdown with:

```
microk8s.stop
```

MicroK8s can be restarted later with:

```
microk8s.start
```

### Removing MicroK8s

Before removing MicroK8s, use `microk8s.reset` to stop all running pods.

```
microk8s.reset
snap remove microk8s
```

### Configuring MicroK8s services

The following systemd services will be running in your system:
- **snap.microk8s.daemon-apiserver**, is the [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) daemon started using the arguments in `${SNAP_DATA}/args/kube-apiserver`
- **snap.microk8s.daemon-controller-manager**, is the [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) daemon started using the arguments in `${SNAP_DATA}/args/kube-controller-manager`
- **snap.microk8s.daemon-scheduler**, is the [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) daemon started using the arguments in `${SNAP_DATA}/args/kube-scheduler`
- **snap.microk8s.daemon-kubelet**, is the [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) daemon started using the arguments in `${SNAP_DATA}/args/kubelet`
- **snap.microk8s.daemon-proxy**, is the [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) daemon started using the arguments in `${SNAP_DATA}/args/kube-proxy`
- **snap.microk8s.daemon-containerd**, is the [containerd](https://containerd.io/) daemon started using the configuration in `${SNAP_DATA}/args/containerd` and `${SNAP_DATA}/args/containerd-template.toml`.
- **snap.microk8s.daemon-etcd**, is the [etcd](https://coreos.com/etcd/docs/latest/v2/configuration.html) daemon started using the arguments in `${SNAP_DATA}/args/etcd`

Normally, `${SNAP_DATA}` points to `/var/snap/microk8s/current`.

To reconfigure a service you will need to edit the corresponding file and then restart the respective daemon. For example:

```
echo '-l=debug' | sudo tee -a /var/snap/microk8s/current/args/containerd
sudo systemctl restart snap.microk8s.daemon-containerd.service
```

### Deploy behind a proxy

To let MicroK8s use a proxy enter the proxy details in `${SNAP_DATA}/args/containerd-env` and restart the containerd daemon service with:
```
sudo systemctl restart snap.microk8s.daemon-containerd.service
```


## Troubleshooting

To troubleshoot a non-functional MicroK8s deployment, start by running the `microk8s.inspect` command. This command performs a set of tests against MicroK8s and collects traces and logs in a report tarball. In case any of the aforementioned daemons are failing you will be urged to look at the respective logs with `journalctl -u snap.microk8s.<daemon>.service`. `microk8s.inspect` may also make suggestions on potential issues it may find. If you do not manage to resolve the issue you are facing please file a [bug](https://github.com/ubuntu/microk8s/issues), attaching the inspection report tarball.

Some common problems and solutions are listed below.

### My dns and dashboard pods are CrashLooping...

The [Kubenet](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) network plugin used by MicroK8s creates a `cbr0` interface when the first pod is created. If you have `ufw` enabled, you'll need to allow traffic on this interface:

```
sudo ufw allow in on cbr0 && sudo ufw allow out on cbr0
```

### My pods can't reach the internet or each other (but my MicroK8s host machine can)...

Make sure packets to/from the pod network interface can be forwarded
to/from the default interface on the host via the `iptables` tool.
Such changes can be made persistent by installing the `iptables-persistent` package:

```
sudo iptables -P FORWARD ACCEPT
sudo apt-get install iptables-persistent
```

or, if using `ufw`:

```
sudo ufw default allow routed
```

The MicroK8s inspect command can be used to check the firewall configuration:

```
microk8s.inspect
```

A warning will be shown if the firewall is not forwarding traffic.

### My log collector is not collecting any logs...

By default container logs are located in `/var/log/pods/{id}`. You have to mount this location in your log collector for that to work. Following is an example diff for [fluent-bit](https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-ds.yaml):

```diff
@@ -36,6 +36,9 @@
         - name: varlibdockercontainers
           mountPath: /var/lib/docker/containers
           readOnly: true
+        - name: varlibdockercontainers
+          mountPath: /var/snap/microk8s/common/var/lib/containerd/
+          readOnly: true
         - name: fluent-bit-config
           mountPath: /fluent-bit/etc/
       terminationGracePeriodSeconds: 10
@@ -45,7 +48,7 @@
           path: /var/log
       - name: varlibdockercontainers
         hostPath:
-          path: /var/lib/docker/containers
+          mountPath: /var/snap/microk8s/common/var/lib/containerd/
       - name: fluent-bit-config
         configMap:
           name: fluent-bit-config
```

### I've enabled the registry, but am unable to access it (or any other NodePort service) from localhost...

Some Linux distributions (like Ubuntu) have IPv6 enabled by default, and 
microk8s networking does not currently support IPv6. A specific symptom
of this problem is that curl to the registry port (or any other NodePort service)
via localhost will hang. You can edit your /etc/hosts file to ensure that localhost 
does not resolve to IPv6 addresses to resolve this problem. See [#498|https://github.com/ubuntu/microk8s/issues/498]
for more details.
