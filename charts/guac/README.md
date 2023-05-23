# GUAC Helm Chart

This is a [Helm Chart](https://helm.sh/docs/topics/charts/) for deploying [GUAC](https://github.com/guacsec/guac) to [Kubernetes](https://kubernetes.io/).

For a complete demo on how to utilize [GUAC](https://github.com/guacsec/guac) using the services deployed by this chart, see the [guac use cases](https://docs.guac.sh/guac-use-cases/).

Please visit the web site at https://guac.sh for latest news and documentation of [GUAC](https://github.com/guacsec/guac).

:exclamation: **Not Production Ready** - [GUAC](https://github.com/guacsec/guac) is still under active development. This chart is provided for rapidly deploying [GUAC](https://github.com/guacsec/guac) for experiment purposes. 
Production support will be provided when [GUAC](https://github.com/guacsec/guac) is production ready.


## GUAC Components

The full GUAC component deployment is a set of asynchronous services that combine to form a robust and scaleable pipeline. This is represented by the area in green in the diagrom below and is also the scope of this chart.

![Guac Diagram](../../docs/images/GUAC-diagram.svg)

### What is being deployed?

- **GraphQL Server**: Serving GUAC GraphQL queries and storing the data. As the
  in-memory backend is used, no separate backend is needed behind the server.

- **Collector-Subscriber**: This component helps communicate to the collectors
  when additional information is needed.

- **Ingestor**: The ingestor listens for things to ingest through Nats, then
  pushes to the GraphQL Server. The ingestor also runs the assembler and parser
  internally.

- **Image Collector**: This collector can pull OCI image metadata (SBOMs and
  attestations) from registries for further inspection.

- **Deps.dev Collector**: This collector gathers further information from
  [Deps.dev](https://deps.dev/) for supported packages.

- **OSV Certifier**: This certifier gathers OSV vulnerability information from
  [osv.dev](https://osv.dev/) about packages.

- **NATS**: [NATS](https://nats.io/) is a messaging middleware used for communication between the GUAC components.


## Prerequisites

* Kubernetes 1.19+
  * Deploy one of these for local testing if you don't have a k8s cluster ready:
    * [kind](https://kind.sigs.k8s.io/), [minikube](https://minikube.sigs.k8s.io/docs/start/), [colima](https://github.com/abiosoft/colima)
* [Helm](https://helm.sh/docs/intro/install/) v3.9.4+
* [kubectl](https://kubernetes.io/docs/tasks/tools/) v1.22+

## Get Repository/Chart Info

```console
helm repo add kusaridev https://kusaridev.github.io/helm-charts
helm repo update

helm search repo kusaridev/guac
```

_See [`helm repo`](https://helm.sh/docs/helm/helm_repo/) for command documentation._

## Install Chart

```console
helm install [RELEASE_NAME] kusaridev/guac
```

See [Customizing the Chart Before Installing](https://helm.sh/docs/intro/using_helm/#customizing-the-chart-before-installing). To see all configurable options with detailed comments, visit the chart's [values.yaml](./values.yaml).  Below you will find the configuration details, descriptions, and defaults.


### Accessing GUAC 
GUAC exposes a few interfaces for user interaction - including the GraphQL Server/Playground, NATS, and CollectSub service. You can access them via ```kubectl port-forward```
```
kubectl port-forward svc/graphql-server 8080:8080
kubectl port-forward svc/guac-nats 4222:4222
kubectl port-forward svc/collectsub 2782:2782
```

### GUAC Visualizer
The GUAC visualizer can be set up following the documentation at https://docs.guac.sh/guac-visualizer/. 
Note that you will need to port-forward to the graphql service for the visualizer to work.

We will support deploying the Visualizer at a later release.

## Uninstall

`helm delete [RELEASE_NAME]`

## Parameters

### Global parameters

| Name                       | Description                                    | Value             |
| -------------------------- | ---------------------------------------------- | ----------------- |
| `imagePullSecrets[0].name` | Docker registry secret name for pulling images | `imagepullsecret` |

### Guac

This section contains parameters for configuring the different GUAC components.

| Name                                                           | Description                                                                             | Value                                                     |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| `guac.workingDir`                                              | Working Directory for GUAC                                                              | `/workspace`                                              |
| `guac.ociCollector.name`                                       | String Name of the OCI Collector component.                                             | `oci-collector`                                           |
| `guac.ociCollector.annotations.reloader.stakater.com/auto`     | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)        | `""`                                                      |
| `guac.ociCollector.replicas`                                   | Number of replicas for oci collector deployment                                         | `1`                                                       |
| `guac.ociCollector.image.repository`                           | Path to the OCI Collector image                                                         | `ghcr.io/guacsec/guac`                                    |
| `guac.ociCollector.image.tag`                                  | Tag if using an image tag.  Optional                                                    | `""`                                                      |
| `guac.ociCollector.image.digest`                               | Sha256 Image Digest.  It is strongly recommended to use this for verification.          | `""`                                                      |
| `guac.ociCollector.image.pullPolicy`                           | ImagePullPolicy for kubernetes                                                          | `IfNotPresent`                                            |
| `guac.ociCollector.image.command`                              | Command for the OCI Collector image.  It is not recommended to override this.           | `["sh","-c","/cnb/process/guaccollect image"]`            |
| `guac.depsDevCollector.name`                                   | String Name of the Deps.Dev Collector component.                                        | `depsdev-collector`                                       |
| `guac.depsDevCollector.annotations.reloader.stakater.com/auto` | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)        | `""`                                                      |
| `guac.depsDevCollector.replicas`                               | Number of replicas for depsdev collector deployment                                     | `1`                                                       |
| `guac.depsDevCollector.image.repository`                       | Path to the Deps.Dev Collector image                                                    | `ghcr.io/guacsec/guac`                                    |
| `guac.depsDevCollector.image.tag`                              | Tag if using an image tag.  Optional                                                    | `""`                                                      |
| `guac.depsDevCollector.image.digest`                           | Sha256 Image Digest.  It is strongly recommended to use this for verification.          | `""`                                                      |
| `guac.depsDevCollector.image.pullPolicy`                       | ImagePullPolicy for kubernetes                                                          | `IfNotPresent`                                            |
| `guac.depsDevCollector.image.command`                          | Command for the Deps.Dev Collector image.  It is not recommended to override this.      | `["sh","-c","/cnb/process/guaccollect deps_dev"]`         |
| `guac.osvCertifier.name`                                       | String Name of the OSV Certifier component.                                             | `osv-certifier`                                           |
| `guac.osvCertifier.annotations.reloader.stakater.com/auto`     | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)        | `""`                                                      |
| `guac.osvCertifier.replicas`                                   | Number of replicas for OSV Certifier deployment                                         | `1`                                                       |
| `guac.osvCertifier.image.repository`                           | Path to the OSV Certifier Collector image                                               | `ghcr.io/guacsec/guac`                                    |
| `guac.osvCertifier.image.tag`                                  | Tag if using an image tag.  Optional                                                    | `""`                                                      |
| `guac.osvCertifier.image.digest`                               | Sha256 Image Digest.  It is strongly recommended to use this for verification.          | `""`                                                      |
| `guac.osvCertifier.image.pullPolicy`                           | ImagePullPolicy for kubernetes                                                          | `IfNotPresent`                                            |
| `guac.osvCertifier.image.command`                              | Command for the OSV Certifier Collector image.  It is not recommended to override this. | `["sh","-c","/cnb/process/guacone certifier osv --poll"]` |
| `guac.ingestor.name`                                           | String Name of the ingestor component.                                                  | `ingestor`                                                |
| `guac.ingestor.annotations.reloader.stakater.com/auto`         | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)        | `""`                                                      |
| `guac.ingestor.replicas`                                       | Number of replicas for ingestor deployment                                              | `1`                                                       |
| `guac.ingestor.image.repository`                               | Path to the Ingestor image                                                              | `ghcr.io/guacsec/guac`                                    |
| `guac.ingestor.image.tag`                                      | Tag if using an image tag.  Optional                                                    | `""`                                                      |
| `guac.ingestor.image.digest`                                   | Sha256 Image Digest.  It is strongly recommended to use this for verification.          | `""`                                                      |
| `guac.ingestor.image.pullPolicy`                               | ImagePullPolicy for kubernetes                                                          | `IfNotPresent`                                            |
| `guac.ingestor.image.command`                                  | Command for the ingestor image.  It is not recommended to override this.                | `["sh","-c","/cnb/process/guacingest"]`                   |
| `guac.collectSub.name`                                         | String Name of the Collector Sub component.                                             | `collectsub`                                              |
| `guac.collectSub.annotations.reloader.stakater.com/auto`       | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)        | `""`                                                      |
| `guac.collectSub.replicas`                                     | Number of replicas for Collector Sub deployment                                         | `1`                                                       |
| `guac.collectSub.image.repository`                             | Path to the Collector Sub image                                                         | `ghcr.io/guacsec/guac`                                    |
| `guac.collectSub.image.tag`                                    | Tag if using an image tag.  Optional                                                    | `""`                                                      |
| `guac.collectSub.image.digest`                                 | Sha256 Image Digest.  It is strongly recommended to use this for verification.          | `""`                                                      |
| `guac.collectSub.image.pullPolicy`                             | ImagePullPolicy for kubernetes                                                          | `IfNotPresent`                                            |
| `guac.collectSub.image.command`                                | Command for the Collector Sub image.  It is not recommended to override this.           | `["sh","-c","/cnb/process/guaccsub"]`                     |
| `guac.collectSub.ports[0].protocol`                            | Protocol used at Collector Sub                                                          | `TCP`                                                     |
| `guac.collectSub.ports[0].port`                                | Port the Collector Sub service listens on                                               | `2782`                                                    |
| `guac.collectSub.ports[0].targetPort`                          | Port the Collector Sub container listens on                                             | `2782`                                                    |
| `guac.graphqlServer.name`                                      | String Name of the GraphQL Server component.                                            | `graphql-server`                                          |
| `guac.graphqlServer.annotations.reloader.stakater.com/auto`    | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)        | `""`                                                      |
| `guac.graphqlServer.replicas`                                  | Number of replicas for GraphQL Server deployment                                        | `1`                                                       |
| `guac.graphqlServer.image.repository`                          | Path to the GraphQL Server image                                                        | `ghcr.io/guacsec/guac`                                    |
| `guac.graphqlServer.image.tag`                                 | Tag if using an image tag.  Optional                                                    | `""`                                                      |
| `guac.graphqlServer.image.digest`                              | Sha256 Image Digest.  It is strongly recommended to use this for verification.          | `""`                                                      |
| `guac.graphqlServer.image.pullPolicy`                          | ImagePullPolicy for kubernetes                                                          | `IfNotPresent`                                            |
| `guac.graphqlServer.image.command`                             | Command for the GraphQL Server image.  It is not recommended to override this.          | `["sh","-c","/cnb/process/guacgql"]`                      |
| `guac.graphqlServer.ports[0].protocol`                         | Protocol used at the the GraphQL Server                                                 | `TCP`                                                     |
| `guac.graphqlServer.ports[0].port`                             | Port the GraphQL Server service listens on                                              | `8080`                                                    |
| `guac.graphqlServer.ports[0].targetPort`                       | Port the GraphQL Server container listens on                                            | `8080`                                                    |
| `guac.graphqlServer.backend`                                   | which backend to use - only support inmem at the moment.                                | `inmem`                                                   |
| `guac.graphqlServer.debug`                                     | Enable debug mode for graphql server; also enable the UI                                | `true`                                                    |
| `guac.visualizer.enabled`                                      | String Whether to deploy the visualizer.                                                | `false`                                                   |
| `guac.visualizer.name`                                         | String Name of the visualizer.                                                          | `visualizer`                                              |
| `guac.visualizer.annotations.reloader.stakater.com/auto`       | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)        | `""`                                                      |
| `guac.visualizer.replicas`                                     | Number of replicas for visualizer deployment                                            | `1`                                                       |
| `guac.visualizer.image.repository`                             | Path to the Ingestor image                                                              | `ghcr.io/kusaridev/guac-visualizer`                       |
| `guac.visualizer.image.tag`                                    | Tag if using an image tag.  Optional                                                    | `""`                                                      |
| `guac.visualizer.image.digest`                                 | Sha256 Image Digest.  It is strongly recommended to use this for verification.          | `""`                                                      |
| `guac.visualizer.image.pullPolicy`                             | ImagePullPolicy for kubernetes                                                          | `IfNotPresent`                                            |
| `guac.visualizer.ports[0].protocol`                            | Protocol used at the visualizer                                                         | `TCP`                                                     |
| `guac.visualizer.ports[0].port`                                | Port the visualizer service listens on                                                  | `3000`                                                    |
| `guac.visualizer.ports[0].targetPort`                          | Port the visualizer container listens on                                                | `3000`                                                    |
| `guac.observability.deployServiceMonitor`                      | Boolean Deploy the service monitor for observability                                    | `false`                                                   |
| `guac.sampleData.ingest`                                       | Boolean - set to true to ingest sample data after deployment                            | `false`                                                   |
| `guac.sampleData.jobName`                                      | Name of the sample data ingest job                                                      | `ingest-guac-data`                                        |

### nats

This is the configuration for nats.  This is a subchart.  See full documentation [here](https://docs.nats.io/running-a-nats-service/nats-kubernetes/helm-charts).

| Name                                                       | Description                                                                                | Value                                                                                                                                                                                                                                                                                                                                                                               |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `nats.nats.jetstream.enabled`                              | Boolean for enabling JetStream.                                                            | `true`                                                                                                                                                                                                                                                                                                                                                                              |
| `nats.nats.limits.maxPayload`                              | Max Payload size for nats                                                                  | `64MB`                                                                                                                                                                                                                                                                                                                                                                              |
| `nats.nats.statefulSetPodLabels.app.kubernetes.io/part-of` | Label to associate nats with GUAC for monitoring purposes                                  | `{"nats":{"jetstream":{"enabled":true},"limits":{"maxPayload":"64MB"},"statefulSetPodLabels":{"app.kubernetes.io/part-of":"guac"}},"natsbox":{"additionalLabels":{"app.kubernetes.io/part-of":"guac"},"podLabels":{"app.kubernetes.io/part-of":"guac"}},"exporter":{"enabled":true,"serviceMonitor":{"enabled":false,"namespace":"monitoring","labels":{"release":"monitoring"}}}}` |
| `nats.natsbox.additionalLabels.app.kubernetes.io/part-of`  | Label to associate natsbox with GUAC for monitoring purposes                               | `guac`                                                                                                                                                                                                                                                                                                                                                                              |
| `nats.natsbox.podLabels.app.kubernetes.io/part-of`         | Label to associate natsbox with GUAC for monitoring purposes                               | `guac`                                                                                                                                                                                                                                                                                                                                                                              |
| `nats.exporter.enabled`                                    | Boolean to enable data collection                                                          | `true`                                                                                                                                                                                                                                                                                                                                                                              |
| `nats.exporter.serviceMonitor.enabled`                     | Boolean to enable nats service monitor                                                     | `false`                                                                                                                                                                                                                                                                                                                                                                             |
| `nats.exporter.serviceMonitor.namespace`                   | nats service monitor namespace - this is for monitoring purposes and is used by Prometheus | `monitoring`                                                                                                                                                                                                                                                                                                                                                                        |
| `nats.exporter.serviceMonitor.labels.release`              | Label to associate nats service monitor with GUAC for monitoring purposes                  | `monitoring`                                                                                                                                                                                                                                                                                                                                                                        |

## Developing
For running the unit tests, install the unittest plugin.

`helm plugin install https://github.com/quintush/helm-unittest`

To Run unit tests

`helm unittest charts/guac -3`