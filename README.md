# Docker Registry Mirror Helm Chart

This directory contains a Kubernetes chart to deploy a private Docker Registry Mirror that will run the registry as a "pull through cache" and cache the requests to Docker hub. Please note, you cannot push to the docker registry when it works under "pull through cache" mode.

Since November 20, 2020, [Anonymous and Free Docker Hub users are limited to 100 and 200 container image pull requests per six hours](https://www.docker.com/increase-rate-limits). To mitigate the impact of this limit, this Helm chart allows you to deploy a Docker Registry as registry mirror that can be used to cache pull request to docker hub.

Forked from https://github.com/twuni/docker-registry.helm

This Helm chart uses official Docker Registry image: https://hub.docker.com/_/registry/

## Prerequisites Details

* PV support on underlying infrastructure (if persistence is required)

## Chart Details

This chart will do the following:

* Implement a Docker registry deployment

## Installing the Chart

First, add the repo:

```console
$ helm repo add docker-registry-mirror https://t83714.github.io/docker-registry-mirror
```

To install the chart, use the following:

```console
$ helm upgrade --install docker-registry-mirror docker-registry-mirror/docker-registry-mirror
```

Set username & password to remote registry (e.g. docker hub):

```console
$ helm upgrade --install --set proxy.username=xxxx,proxy.password=xxx docker-registry-mirror docker-registry-mirror/docker-registry-mirror 
```

## Configure Minikube to use registry-mirror as Pull cache mirror

1. Find nodePort allocated to the registry mirror:

```console
kubectl get svc --all-namespaces --selector=app=docker-registry-mirror -oyaml | grep nodePort
```

This command will list the `nodePort` assign to your registry mirror service.

To verify the nodePort & registry mirror installation:
```console
# Log into Minikube VM via SSH
minikube ssh
curl http://localhost:xxxxx/v2/_catalog  
```

Here, `xxxxx` is the nodePort number we just find out via "kubectl" command.

We should see:

```console
{"repositories":[]}
```

2. Edit minikube configuration to use registry mirror

Open minikube configuration file: `~/.minikube/machines/minikube/config.json`.

Under key `HostOptions.EngineOptions`, add or replace (if exists) key `RegistryMirror` as:
```js
"RegistryMirror": [
    "http://localhost:xxxxx"
]
```

Here, `xxxxx` is the nodePort number we find out via "kubectl" command in step 1.

3. Restart minikube

```console
# Apply the config and restart minikube
minikube stop
minikube start
```

After restart, you should see docker pull requests appearing in docker registry mirror pod logs.

## Configuration

The following table lists the configurable parameters of the docker-registry chart and
their default values.

| Parameter                   | Description                                                                                | Default         |
|:----------------------------|:-------------------------------------------------------------------------------------------|:----------------|
| `proxy.remoteurl`           | The url of the remote docker registry to be cached                                         | `https://registry-1.docker.io`  |
| `proxy.username`            | Remote docker registry username (optional)                                                 | `nil`  |
| `proxy.password`            | Remote docker registry password (optional)                                                 | `nil`  |
| `image.pullPolicy`          | Container pull policy                                                                      | `IfNotPresent`  |
| `image.repository`          | Container image to use                                                                     | `registry`      |
| `image.tag`                 | Container image tag to deploy                                                              | `2.7.1`         |
| `imagePullSecrets`          | Specify image pull secrets                                                                 | `nil` (does not add image pull secrets to deployed pods) |
| `persistence.accessMode`    | Access mode to use for PVC                                                                 | `ReadWriteOnce` |
| `persistence.enabled`       | Whether to use a PVC for the Docker storage                                                | `false`         |
| `persistence.deleteEnabled` | Enable the deletion of image blobs and manifests by digest                                 | `nil`           |
| `persistence.size`          | Amount of space to claim for PVC                                                           | `10Gi`          |
| `persistence.storageClass`  | Storage Class to use for PVC                                                               | `-`             |
| `persistence.existingClaim` | Name of an existing PVC to use for config                                                  | `nil`           |
| `service.port`              | TCP port on which the service is exposed                                                   | `5000`          |
| `service.type`              | service type                                                                               | `ClusterIP`     |
| `service.clusterIP`         | if `service.type` is `ClusterIP` and this is non-empty, sets the cluster IP of the service | `nil`           |
| `service.nodePort`          | if `service.type` is `NodePort` and this is non-empty, sets the node port of the service   | `nil`           |
| `service.loadBalancerIP`     | if `service.type` is `LoadBalancer` and this is non-empty, sets the loadBalancerIP of the service | `nil`          |
| `service.loadBalancerSourceRanges`| if `service.type` is `LoadBalancer` and this is non-empty, sets the loadBalancerSourceRanges of the service | `nil`           |
| `service.sessionAffinity`       | service session affinity                                                               | `nil`           |
| `service.sessionAffinityConfig` | service session affinity config                                                        | `nil`           |
| `replicaCount`              | k8s replicas                                                                               | `1`             |
| `updateStrategy`            | update strategy for deployment                                                             | `{}`            |
| `podAnnotations`            | Annotations for pod                                                                        | `{}`            |
| `podLabels`                 | Labels for pod                                                                             | `{}`            |
| `podDisruptionBudget`       | Pod disruption budget                                                                      | `{}`            |
| `resources.limits.cpu`      | Container requested CPU                                                                    | `nil`           |
| `resources.limits.memory`   | Container requested memory                                                                 | `nil`           |
| `priorityClassName      `   | priorityClassName                                                                          | `""`            |
| `storage`                   | Storage system to use                                                                      | `filesystem`    |
| `tlsSecretName`             | Name of secret for TLS certs                                                               | `nil`           |
| `secrets.htpasswd`          | Htpasswd authentication                                                                    | `nil`           |
| `secrets.s3.accessKey`      | Access Key for S3 configuration                                                            | `nil`           |
| `secrets.s3.secretKey`      | Secret Key for S3 configuration                                                            | `nil`           |
| `secrets.swift.username`    | Username for Swift configuration                                                           | `nil`           |
| `secrets.swift.password`    | Password for Swift configuration                                                           | `nil`           |
| `haSharedSecret`            | Shared secret for Registry                                                                 | `nil`           |
| `configData`                | Configuration hash for docker                                                              | `nil`           |
| `s3.region`                 | S3 region                                                                                  | `nil`           |
| `s3.regionEndpoint`         | S3 region endpoint                                                                         | `nil`           |
| `s3.bucket`                 | S3 bucket name                                                                             | `nil`           |
| `s3.encrypt`                | Store images in encrypted format                                                           | `nil`           |
| `s3.secure`                 | Use HTTPS                                                                                  | `nil`           |
| `swift.authurl`             | Swift authurl                                                                              | `nil`           |
| `swift.container`           | Swift container                                                                            | `nil`           |
| `nodeSelector`              | node labels for pod assignment                                                             | `{}`            |
| `affinity`                  | affinity settings                                                                          | `{}`            |
| `tolerations`               | pod tolerations                                                                            | `[]`            |
| `ingress.enabled`           | If true, Ingress will be created                                                           | `false`         |
| `ingress.annotations`       | Ingress annotations                                                                        | `{}`            |
| `ingress.labels`            | Ingress labels                                                                             | `{}`            |
| `ingress.path`              | Ingress service path                                                                       | `/`             |
| `ingress.hosts`             | Ingress hostnames                                                                          | `[]`            |
| `ingress.tls`               | Ingress TLS configuration (YAML)                                                           | `[]`            |
| `extraVolumeMounts`         | Additional volumeMounts to the registry container                                          | `[]`            |
| `extraVolumes`              | Additional volumes to the pod                                                              | `[]`            |
| `extraVars`                 | Pass extra environment variables to the Docker Registry container                          | `nil`           |

Specify each parameter using the `--set key=value[,key=value]` argument to
`helm install`.

To generate htpasswd file, run this docker command:
`docker run --entrypoint htpasswd registry:2 -Bbn user password > ./htpasswd`.
