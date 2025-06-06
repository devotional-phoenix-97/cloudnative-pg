# Operator configuration

<!-- SPDX-License-Identifier: CC-BY-4.0 -->

The operator for CloudNativePG is installed from a standard
deployment manifest and follows the convention over configuration paradigm.
While this is fine in most cases, there are some scenarios where you want
to change the default behavior, such as:

- defining annotations and labels to be inherited by all resources created
  by the operator and that are set in the cluster resource
- defining a different default image for PostgreSQL or an additional pull secret

By default, the operator is installed in the `cnpg-system`
namespace as a Kubernetes `Deployment` called `cnpg-controller-manager`.

!!! Note
    In the examples below we assume the default name and namespace for the operator deployment.

The behavior of the operator can be customized through a `ConfigMap`/`Secret` that
is located in the same namespace of the operator deployment and with
`cnpg-controller-manager-config` as the name.

!!! Important
    Any change to the config's `ConfigMap`/`Secret` will not be automatically
    detected by the operator, - and as such, it needs to be reloaded (see below).
    Moreover, changes only apply to the resources created after the configuration
    is reloaded.

!!! Important
    The operator first processes the ConfigMap values and then the Secret’s, in this order.
    As a result, if a parameter is defined in both places, the one in the Secret will be used.

## Available options

The operator looks for the following environment variables to be defined in the `ConfigMap`/`Secret`:

| Name                                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `VOLUME_MIGRATION`                   | This setting is used to select either manual or auto Volume migration. When not set, the operator requests a value and disables further reconciliation. When set to `manual`, instances need to be recreated manually. When set to `remap` PV's would be preserved, PVC's would be recreated with proper name, and Pod would be recreated with new PVC names in it's Volume block.                                                                                                                                                                                                                    |
| `CERTIFICATE_DURATION`                    | Determines the lifetime of the generated certificates in days. Default is 90.                                                                                                                                                                                                                                                                                                                                                                                                                |
| `CLUSTERS_ROLLOUT_DELAY`                  | The duration (in seconds) to wait between the roll-outs of different clusters during an operator upgrade. This setting controls the timing of upgrades across clusters, spreading them out to reduce system impact. The default value is `0` which means no delay between PostgreSQL cluster upgrades.                                                                                                                                                                                       |
| `CREATE_ANY_SERVICE`                      | When set to `true`, will create `-any` service for the cluster. Default is `false`.                                                                                                                                                                                                                                                                                                                                                                                                          |
| `DATA_VOLUME_SUFFIX`                      | When set, the PersistentVolumeClaim for data volumes will be named according to the pod name suffix'ed with this value.                                                                                                                                                                                                                                                                                                                                                                      |
| `ENABLE_AZURE_PVC_UPDATES`                | Enables to delete Postgres pod if its PVC is stuck in Resizing condition. This feature is mainly for the Azure environment (default `false`).                                                                                                                                                                                                                                                                                                                                                |
| `ENABLE_INSTANCE_MANAGER_INPLACE_UPDATES` | When set to `true`, enables in-place updates of the instance manager after an update of the operator, avoiding rolling updates of the cluster (default `false`).                                                                                                                                                                                                                                                                                                                             |
| `EXPIRING_CHECK_THRESHOLD`                | Determines the threshold, in days, for identifying a certificate as expiring. Default is 7.                                                                                                                                                                                                                                                                                                                                                                                                  |
| `INCLUDE_PLUGINS`                         | A comma-separated list of plugins to be always included in the Cluster's reconciliation.                                                                                                                                                                                                                                                                                                                                                                                                     |
| `INHERITED_ANNOTATIONS`                   | List of annotation names that, when defined in a `Cluster` metadata, will be inherited by all the generated resources, including pods.                                                                                                                                                                                                                                                                                                                                                       |
| `INHERITED_LABELS`                        | List of label names that, when defined in a `Cluster` metadata, will be inherited by all the generated resources, including pods.                                                                                                                                                                                                                                                                                                                                                            |
| `KUBERNETES_CLUSTER_DOMAIN`               | Defines the domain suffix for service FQDNs within the Kubernetes cluster. If left unset, it defaults to "cluster.local".                                                                                                                                                                                                                                                                                                                                                                    |
| `INSTANCES_ROLLOUT_DELAY`                 | The duration (in seconds) to wait between roll-outs of individual PostgreSQL instances within the same cluster during an operator upgrade. The default value is `0`, meaning no delay between upgrades of instances in the same PostgreSQL cluster.                                                                                                                                                                                                                                          |
| `MONITORING_QUERIES_CONFIGMAP`            | The name of a ConfigMap in the operator's namespace with a set of default queries (to be specified under the key `queries`) to be applied to all created Clusters.                                                                                                                                                                                                                                                                                                                           |
| `MONITORING_QUERIES_SECRET`               | The name of a Secret in the operator's namespace with a set of default queries (to be specified under the key `queries`) to be applied to all created Clusters.                                                                                                                                                                                                                                                                                                                              |
| `OPERATOR_IMAGE_NAME`                     | The name of the operator image used to bootstrap Pods. Defaults to the image specified during installation.                                                                                                                                                                                                                                                                                                                                                                                  |
| `POSTGRES_IMAGE_NAME`                     | The name of the PostgreSQL image used by default for new clusters. Defaults to the version specified in the operator.                                                                                                                                                                                                                                                                                                                                                                        |
| `PULL_SECRET_NAME`                        | Name of an additional pull secret to be defined in the operator's namespace and to be used to download images                                                                                                                                                                                                                                                                                                                                                                                |
| `STANDBY_TCP_USER_TIMEOUT`                | Defines the [`TCP_USER_TIMEOUT` socket option](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-TCP-USER-TIMEOUT) for replication connections from standby instances to the primary. Default is 0 (system's default).                                                                                                                                                                                                                                              |
| `DRAIN_TAINTS`                            | Specifies the taint keys that should be interpreted as indicators of node drain. By default, it includes the taints commonly applied by [kubectl](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/), [Cluster Autoscaler](https://github.com/kubernetes/autoscaler), and [Karpenter](https://github.com/aws/karpenter-provider-aws): `node.kubernetes.io/unschedulable`, `ToBeDeletedByClusterAutoscaler`, `karpenter.sh/disrupted`, `karpenter.sh/disruption`. |
| `WAL_VOLUME_SUFFIX`                       | When set, the PersistentVolmeClaim for WAL volumes will be named according to the pod name suffix'ed with this value.                                                                                                                                                                                                                                                                                                                                                                        |

Values in `INHERITED_ANNOTATIONS` and `INHERITED_LABELS` support path-like wildcards. For example, the value `example.com/*` will match
both the value `example.com/one` and `example.com/two`.

When you specify an additional pull secret name using the `PULL_SECRET_NAME` parameter,
the operator will use that secret to create a pull secret for every created PostgreSQL
cluster. That secret will be named `<cluster-name>-pull`.

The namespace where the operator looks for the `PULL_SECRET_NAME` secret is where
you installed the operator. If the operator is not able to find that secret, it
will ignore the configuration parameter.

## Defining an operator config map

The example below customizes the behavior of the operator, by defining
the label/annotation names to be inherited by the resources created by
any `Cluster` object that is deployed at a later time, by enabling
[in-place updates for the instance
manager](installation_upgrade.md#in-place-updates-of-the-instance-manager),
and by spreading upgrades.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cnpg-controller-manager-config
  namespace: cnpg-system
data:
  CLUSTERS_ROLLOUT_DELAY: '60'
  ENABLE_INSTANCE_MANAGER_INPLACE_UPDATES: 'true'
  INHERITED_ANNOTATIONS: categories
  INHERITED_LABELS: environment, workload, app
  INSTANCES_ROLLOUT_DELAY: '10'
```

## Defining an operator secret

The example below customizes the behavior of the operator, by defining
the label/annotation names to be inherited by the resources created by
any `Cluster` object that is deployed at a later time, and by enabling
[in-place updates for the instance
manager](installation_upgrade.md#in-place-updates-of-the-instance-manager),
and by spreading upgrades.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cnpg-controller-manager-config
  namespace: cnpg-system
type: Opaque
stringData:
  CLUSTERS_ROLLOUT_DELAY: '60'
  ENABLE_INSTANCE_MANAGER_INPLACE_UPDATES: 'true'
  INHERITED_ANNOTATIONS: categories
  INHERITED_LABELS: environment, workload, app
  INSTANCES_ROLLOUT_DELAY: '10'
```

## Restarting the operator to reload configs

For the change to be effective, you need to recreate the operator pods to
reload the config map. If you have installed the operator on Kubernetes
using the manifest you can do that by issuing:

```shell
kubectl rollout restart deployment \
    -n cnpg-system \
    cnpg-controller-manager
```

In general, given a specific namespace, you can delete the operator pods with
the following command:

```shell
kubectl delete pods -n [NAMESPACE_NAME_HERE] \
  -l app.kubernetes.io/name=cloudnative-pg
```

!!! Warning
    Customizations will be applied only to `Cluster` resources created
    after the reload of the operator deployment.

Following the above example, if the `Cluster` definition contains a `categories`
annotation and any of the `environment`, `workload`, or `app` labels, these will
be inherited by all the resources generated by the deployment.

## pprof HTTP Server

The operator can expose a PPROF HTTP server with the following endpoints on `localhost:6060`:

- `/debug/pprof/`. Responds to a request for "/debug/pprof/" with an HTML page listing the available profiles
- `/debug/pprof/cmdline`. Responds with the running program's command line, with arguments separated by NULL bytes.
- `/debug/pprof/profile`. Responds with the pprof-formatted cpu profile. Profiling lasts for duration specified in seconds GET parameter, or for 30 seconds if not specified.
- `/debug/pprof/symbol`. Looks up the program counters listed in the request, responding with a table mapping program counters to function names.
- `/debug/pprof/trace`. Responds with the execution trace in binary form. Tracing lasts for duration specified in seconds GET parameter, or for 1 second if not specified.

To enable the operator you need to edit the operator deployment add the flag `--pprof-server=true`.

You can do this by executing these commands:

```shell
kubectl edit deployment -n cnpg-system cnpg-controller-manager
```

Then on the edit page scroll down the container args and add
`--pprof-server=true`, as in this example:

```yaml
      containers:
      - args:
        - controller
        - --enable-leader-election
        - --config-map-name=cnpg-controller-manager-config
        - --secret-name=cnpg-controller-manager-config
        - --log-level=info
        - --pprof-server=true # relevant line
        command:
        - /manager
```

Save the changes; the deployment now will execute a roll-out, and the new pod
will have the PPROF server enabled.

Once the pod is running you can exec inside the container by doing:

```shell
kubectl exec -ti -n cnpg-system <pod name> -- bash
```

Once inside execute:

```shell
curl localhost:6060/debug/pprof/
```
