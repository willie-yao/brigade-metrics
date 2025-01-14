# Brigade Metrics: Monitoring for Brigade 2

Brigade Metrics adds monitoring capabilities to a Brigade 2 installation. It
utilizes Brigade APIs to export time series metrics to Prometheus and makes
visualizations of those metrics available through a Grafana dashboard.

Brigade 2 itself is currently in an _beta_ state and remains under active
development, as such, the same is true for this add-on component.

## Getting Started

Follow these steps to get started.

### Prerequisites

Since Brigade Metrics aggregates and exposes metrics for a running Brigade 2
installation, an operational Brigade 2 installation is a prerequisite.

Note that Brigade Metrics is only compatible with the _beta_ series of Brigade 2
releases.

If necessary, please refer to
[Brigade 2's own getting started documentation](https://github.com/brigadecore/brigade/tree/v2)
for guidance in fulfilling this dependency.

Once Brigade 2 is operational, create a service account for use by Brigade
Metrics:

```console
$ brig service-account create \
  --id brigade-metrics \
  --description "Used by Brigade Metrics"
```

This command will display a token that Brigade Metrics can use for
authenticating to the Brigade 2 API server. Take note of this value. It will
be required in subsequent steps and cannot be retrieved later through any other
means.

Now grant the service account global read permissions:

```console
$ brig role grant READER --service-account brigade-metrics
```

### Installing Brigade Metrics

For now, we're using the [GitHub Container Registry](https://ghcr.io) (which is
an [OCI registry](https://helm.sh/docs/topics/registries/)) to host our Helm
chart. Helm 3 has _experimental_ support for OCI registries. In the event that
the Helm 3 dependency proves troublesome for users, or in the event that this
experimental feature goes away, or isn't working like we'd hope, we will revisit
this choice before going GA.

To install Brigade Metrics, begin by pulling the chart from GCR and exporting
it to some location on your local system. Here, we export it to `~/charts`:

```console
$ export HELM_EXPERIMENTAL_OCI=1
$ helm chart pull ghcr.io/brigadecore/brigade-metrics:v0.1.0
$ helm chart export ghcr.io/brigadecore/brigade-metrics:v0.1.0 -d ~/charts
```

Use the following command to extract the full set of configuration options from
the chart. Here we're storing a copy at `~/brigade-metrics-values.yaml`:

```console
$ helm inspect values ~/charts/brigade-metrics > ~/brigade-metrics-values.yaml
```

Edit the configuration (`~/brigade-metrics-values.yaml` in this example). At
minimum, you will need to make the following changes:

* Set the value of `exporter.brigade.apiAddress` to the address of your Brigade 2
  API server. This should utilize the _internal_ DNS hostname by which that API
  server is reachable _within_ your Kubernetes cluster. This value is defaulted
  to `https://brigade-apiserver.brigade.svc.cluster.local`, but may need to be
  updated if you installed Brigade 2 in a different namespace.

* Set the value of `exporter.brigade.apiToken` to the service account token that
  was generated earlier.

* Specify a username and password for the metrics dashboard (Grafana) by setting
  values for `grafana.auth.username` and `grafana.auth.password`.

Install Brigade Metrics, referencing your edited configuration:

```console
$ helm install brigade-metrics ~/charts/brigade-metrics \
  --create-namespace \
  --namespace brigade-metrics \
  --values ~/brigade-metrics-values.yaml
```

### Accessing the Dashboard

Use the following command to determine when the dashboard (Grafana) is ready:

```console
$ kubectl get deployment brigade-metrics-grafana --namespace brigade-metrics 
```

If you deployed Brigade Metrics on a public cloud _and_ kept the default service
type of `LoadBalancer` for the dashboard, then use the following command to
determine when your dashboard has been assigned a public IP:

```console
$ kubectl get service brigade-metrics-grafana --namespace brigade-metrics
```

The dashboard should be accessible at the public IP using HTTPS. If you used
the default, auto-generated certificate, expect to receive a cert warning.

If you deployed Brigade Metrics on a local cluster or changed the service type
for the dashboard to something like `ClusterIP`, then use port forwarding to
access the dashboard:

```console
$ kubectl port-forward \
  service/brigade-metrics-grafana \
  --namespace brigade-metrics \
  8443:443
```

The dashboard should be accessible at `https://localhost:8443`. Expect to
receive a cert warning.

Log in using the username and password you selected in the previous section.

## Contributing

The Brigade project accepts contributions via GitHub pull requests. The
[Contributing](CONTRIBUTING.md) document outlines the process to help get your
contribution accepted.

## Support & Feedback

We have a slack channel!
[Kubernetes/#brigade](https://kubernetes.slack.com/messages/C87MF1RFD) Feel free
to join for any support questions or feedback, we are happy to help. To report
an issue or to request a feature open an issue
[here](https://github.com/brigadecore/brigade-metrics/issues)
