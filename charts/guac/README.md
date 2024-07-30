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

- **GUAC Visualizer**: The GUAC Visualizer is an experimental utility that can be used to interact with GUAC services. It acts as a way to visualize the software supply chain graph, as well as a means to explore the supply chain and prototype policies. 

- **NATS**: [NATS](https://nats.io/) is a messaging middleware used for communication between the GUAC components.

- **MinIO**: [MinIO](https://min.io/) is a S3 compatible object store used for holding SBOMs for ingesting into GUAC.

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

## Uninstall

`helm delete [RELEASE_NAME]`

## Parameters

### Global parameters

| Name                       | Description                                    | Value             |
| -------------------------- | ---------------------------------------------- | ----------------- |
| `imagePullSecrets[0].name` | Docker registry secret name for pulling images | `imagepullsecret` |

### Guac

This section contains parameters for configuring the different GUAC components.

| Name                                                           | Description                                                                                  | Value                                          |
| -------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| `guac.guacImage.repository`                                    | Path to the GUAC image                                                                       | `ghcr.io/guacsec/guac`                         |
| `guac.guacImage.tag`                                           | Tag if using an image tag.  Optional                                                         | `undefined`                                    |
| `guac.guacImage.digest`                                        | Sha256 Image Digest.  It is strongly recommended to use this for verification.               | `""`                                           |
| `guac.guacImage.pullPolicy`                                    | ImagePullPolicy for kubernetes                                                               | `IfNotPresent`                                 |
| `guac.guacImage.workingDir`                                    | Working Directory for GUAC                                                                   | `/guac`                                        |
| `guac.common.env`                                              | common environment variables apply to all guac services                                      | `""`                                           |
| `guac.common.tolerations`                                      | common tolerations apply to all guac services                                                | `""`                                           |
| `guac.configMap.enabled`                                       | Whether to create the guac-cm configMap                                                      | `true`                                         |
| `guac.ociCollector.enabled`                                    | String Whether to deploy OCI Collector                                                       | `true`                                         |
| `guac.ociCollector.name`                                       | String Name of the OCI Collector component.                                                  | `oci-collector`                                |
| `guac.ociCollector.annotations.reloader.stakater.com/auto`     | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)             | `""`                                           |
| `guac.ociCollector.replicas`                                   | Number of replicas for oci collector deployment                                              | `1`                                            |
| `guac.ociCollector.image.command`                              | Command for the OCI Collector image.  It is not recommended to override this.                | `["sh","-c","/opt/guac/guaccollect image"]`    |
| `guac.ociCollector.env`                                        | Environment variables for OCI Collector.                                                     | `[]`                                           |
| `guac.ociCollector.nodeSelector`                               | - sets the node selector for where to run the deployment                                     | `{}`                                           |
| `guac.ociCollector.tolerations`                                |                                                                                              | `[]`                                           |
| `guac.ociCollector.serviceAccount.create`                      | - whether to create OCI Collector service account                                            | `true`                                         |
| `guac.ociCollector.serviceAccount.annotations`                 | - OCI Collector service account annotations                                                  | `{}`                                           |
| `guac.ociCollector.resources`                                  | - [map] resource requests or limits of the ociCollector deployment                           | `{}`                                           |
| `guac.depsDevCollector.enabled`                                | String Whether to deploy Deps.Dev Collector                                                  | `true`                                         |
| `guac.depsDevCollector.name`                                   | String Name of the Deps.Dev Collector component.                                             | `depsdev-collector`                            |
| `guac.depsDevCollector.annotations.reloader.stakater.com/auto` | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)             | `""`                                           |
| `guac.depsDevCollector.replicas`                               | Number of replicas for depsdev collector deployment                                          | `1`                                            |
| `guac.depsDevCollector.image.command`                          | Command for the Deps.Dev Collector image.  It is not recommended to override this.           | `["sh","-c","/opt/guac/guaccollect deps_dev"]` |
| `guac.depsDevCollector.env`                                    | Environment variables for Deps.Dev Collector.                                                | `[]`                                           |
| `guac.depsDevCollector.nodeSelector`                           | - sets the node selector for where to run the deployment                                     | `{}`                                           |
| `guac.depsDevCollector.tolerations`                            |                                                                                              | `[]`                                           |
| `guac.depsDevCollector.serviceAccount.create`                  | - whether to create depsDevCollector service account                                         | `true`                                         |
| `guac.depsDevCollector.serviceAccount.annotations`             |                                                                                              | `{}`                                           |
| `guac.depsDevCollector.resources`                              | - [map] resource requests or limits of the depsDevCollector deployment                       | `{}`                                           |
| `guac.osvCertifier.enabled`                                    | String Whether to deploy OSV Certifier                                                       | `true`                                         |
| `guac.osvCertifier.name`                                       | String Name of the OSV Certifier component.                                                  | `osv-certifier`                                |
| `guac.osvCertifier.annotations.reloader.stakater.com/auto`     | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)             | `""`                                           |
| `guac.osvCertifier.replicas`                                   | Number of replicas for OSV Certifier deployment                                              | `1`                                            |
| `guac.osvCertifier.image.command`                              | Command for the OSV Certifier Collector image.  It is not recommended to override this.      | `["sh","-c","/opt/guac/guaccollect osv"]`      |
| `guac.osvCertifier.env`                                        | Environment variables for OSV Certifier.                                                     | `[]`                                           |
| `guac.osvCertifier.nodeSelector`                               | - sets the node selector for where to run the deployment                                     | `{}`                                           |
| `guac.osvCertifier.tolerations`                                |                                                                                              | `[]`                                           |
| `guac.osvCertifier.serviceAccount.create`                      | - whether to create osvCertifier service account                                             | `true`                                         |
| `guac.osvCertifier.serviceAccount.annotations`                 | - OSV Certifier service account annotations                                                  | `{}`                                           |
| `guac.osvCertifier.resources`                                  | - [map] resource requests or limits of the OSV Certifier deployment                          | `{}`                                           |
| `guac.cdCertifier.enabled`                                     | String Whether to deploy CD Certifier                                                        | `true`                                         |
| `guac.cdCertifier.name`                                        | String Name of the CD Certifier component.                                                   | `cd-certifier`                                 |
| `guac.cdCertifier.annotations.reloader.stakater.com/auto`      | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)             | `""`                                           |
| `guac.cdCertifier.replicas`                                    | Number of replicas for CD Certifier deployment                                               | `1`                                            |
| `guac.cdCertifier.image.command`                               | Command for the CD Certifier Collector image.  It is not recommended to override this.       | `["sh","-c","/opt/guac/guaccollect cd"]`       |
| `guac.cdCertifier.env`                                         | Environment variables for CD Certifier.                                                      | `[]`                                           |
| `guac.cdCertifier.nodeSelector`                                | - sets the node selector for where to run the deployment                                     | `{}`                                           |
| `guac.cdCertifier.tolerations`                                 |                                                                                              | `[]`                                           |
| `guac.cdCertifier.serviceAccount.create`                       | - whether to create cdCertifier service account                                              | `true`                                         |
| `guac.cdCertifier.serviceAccount.annotations`                  | - CD Certifier service account annotations                                                   | `{}`                                           |
| `guac.cdCertifier.resources`                                   | - [map] resource requests or limits of the cd Certifier deployment                           | `{}`                                           |
| `guac.ingestor.enabled`                                        | String Whether to deploy Ingestor                                                            | `true`                                         |
| `guac.ingestor.name`                                           | String Name of the ingestor component.                                                       | `ingestor`                                     |
| `guac.ingestor.annotations.reloader.stakater.com/auto`         | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)             | `""`                                           |
| `guac.ingestor.replicas`                                       | Number of replicas for ingestor deployment                                                   | `1`                                            |
| `guac.ingestor.image.command`                                  | Command for the ingestor image.  It is not recommended to override this.                     | `["sh","-c","/opt/guac/guacingest"]`           |
| `guac.ingestor.env`                                            | Environment variables for ingestor.                                                          | `[]`                                           |
| `guac.ingestor.nodeSelector`                                   | - sets the node selector for where to run the deployment                                     | `{}`                                           |
| `guac.ingestor.serviceAccount.create`                          | - whether to create ingestor service account                                                 | `true`                                         |
| `guac.ingestor.serviceAccount.annotations`                     | - Ingestor service account annotations                                                       | `{}`                                           |
| `guac.ingestor.tolerations`                                    |                                                                                              | `[]`                                           |
| `guac.ingestor.resources`                                      | - [map] resource requests or limits of the ingestor deployment                               | `{}`                                           |
| `guac.collectSub.enabled`                                      | String Whether to deploy CollectSub                                                          | `true`                                         |
| `guac.collectSub.name`                                         | String Name of the CollectSub component.                                                     | `collectsub`                                   |
| `guac.collectSub.annotations.reloader.stakater.com/auto`       | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)             | `""`                                           |
| `guac.collectSub.replicas`                                     | Number of replicas for CollectSub deployment                                                 | `1`                                            |
| `guac.collectSub.image.command`                                | Command for the CollectSub image.  It is not recommended to override this.                   | `["sh","-c","/opt/guac/guaccsub"]`             |
| `guac.collectSub.env`                                          | Environment variables for CollectSub.                                                        | `[]`                                           |
| `guac.collectSub.image.ports[0].containerPort`                 | Port the CollectSub container listens on                                                     | `2782`                                         |
| `guac.collectSub.svcPorts[0].protocol`                         | Protocol used at CollectSub                                                                  | `TCP`                                          |
| `guac.collectSub.svcPorts[0].port`                             | Port the CollectSub service listens on                                                       | `2782`                                         |
| `guac.collectSub.svcPorts[0].targetPort`                       | Port the CollectSub container listens on                                                     | `2782`                                         |
| `guac.collectSub.nodeSelector`                                 | - sets the node selector for where to run the deployment                                     | `{}`                                           |
| `guac.collectSub.tolerations`                                  |                                                                                              | `[]`                                           |
| `guac.collectSub.serviceAccount.create`                        | - whether to create collectSub service account                                               | `true`                                         |
| `guac.collectSub.serviceAccount.annotations`                   | - CollectSub service account annotations                                                     | `{}`                                           |
| `guac.collectSub.resources`                                    | - [map] resource requests or limits of the collectSub deployment                             | `{}`                                           |
| `guac.graphqlServer.enabled`                                   | String Whether to deploy GraphQL Server                                                      | `true`                                         |
| `guac.graphqlServer.name`                                      | String Name of the GraphQL Server component.                                                 | `graphql-server`                               |
| `guac.graphqlServer.annotations.reloader.stakater.com/auto`    | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)             | `""`                                           |
| `guac.graphqlServer.replicas`                                  | Number of replicas for GraphQL Server deployment                                             | `1`                                            |
| `guac.graphqlServer.image.command`                             | Command for the GraphQL Server image.  It is not recommended to override this.               | `["sh","-c","/opt/guac/guacgql"]`              |
| `guac.graphqlServer.env`                                       | Environment variables for GraphQL Server.                                                    | `[]`                                           |
| `guac.graphqlServer.image.ports[0].containerPort`              | Port the GraphQL Server container listens on                                                 | `8080`                                         |
| `guac.graphqlServer.svcPorts[0].protocol`                      | Protocol used at the the GraphQL Server                                                      | `TCP`                                          |
| `guac.graphqlServer.svcPorts[0].port`                          | Port the GraphQL Server service listens on                                                   | `8080`                                         |
| `guac.graphqlServer.svcPorts[0].targetPort`                    | Port the GraphQL Server container listens on                                                 | `8080`                                         |
| `guac.graphqlServer.nodePortSvcPorts`                          | NodePort service ports definition                                                            | `{}`                                           |
| `guac.graphqlServer.backend`                                   | which backend to use - keyvalue (default) | arango | ent.                                    | `keyvalue`                                     |
| `guac.graphqlServer.debug`                                     | Enable debug mode for graphql server; also enable the UI                                     | `true`                                         |
| `guac.graphqlServer.nodeSelector`                              | - sets the node selector for where to run the deployment                                     | `{}`                                           |
| `guac.graphqlServer.serviceAccount.create`                     | - whether to create graphqlServer service account                                            | `true`                                         |
| `guac.graphqlServer.serviceAccount.annotations`                | - graphql server service account annotations                                                 | `{}`                                           |
| `guac.graphqlServer.service.createNodePortService`             | - Whether to deploy a NodePort type service                                                  | `false`                                        |
| `guac.graphqlServer.additionalVolumeMounts`                    |                                                                                              | `[]`                                           |
| `guac.graphqlServer.additionalVolumes`                         |                                                                                              | `[]`                                           |
| `guac.graphqlServer.tolerations`                               |                                                                                              | `[]`                                           |
| `guac.graphqlServer.resources`                                 | - [map] resource requests or limits of the graphqlServer deployment                          | `{}`                                           |
| `guac.restApi.enabled`                                         | String Whether to deploy the restApi                                                         | `true`                                         |
| `guac.restApi.name`                                            | String Name of the restApi component.                                                        | `rest-api`                                     |
| `guac.restApi.annotations.reloader.stakater.com/auto`          | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)             | `""`                                           |
| `guac.restApi.replicas`                                        | Number of replicas for restApi deployment                                                    | `1`                                            |
| `guac.restApi.image.command`                                   | Command for the restApi image.  It is not recommended to override this.                      | `["sh","-c","/opt/guac/guacrest"]`             |
| `guac.restApi.env`                                             | Environment variables for restApi.                                                           | `[]`                                           |
| `guac.restApi.image.ports[0].containerPort`                    | Port the restApi container listens on                                                        | `8081`                                         |
| `guac.restApi.svcPorts[0].protocol`                            | Protocol used at the the restApi                                                             | `TCP`                                          |
| `guac.restApi.svcPorts[0].port`                                | Port the restApi service listens on                                                          | `8081`                                         |
| `guac.restApi.svcPorts[0].targetPort`                          | Port the restApi container listens on                                                        | `8081`                                         |
| `guac.restApi.serviceAccount.create`                           | - whether to create restApi service account                                                  | `true`                                         |
| `guac.restApi.serviceAccount.annotations`                      | - graphql server service account annotations                                                 | `{}`                                           |
| `guac.restApi.nodeSelector`                                    | - sets the node selector for where to run the deployment                                     | `{}`                                           |
| `guac.restApi.tolerations`                                     |                                                                                              | `[]`                                           |
| `guac.restApi.resources`                                       | - [map] resource requests or limits of the restApi deployment                                | `{}`                                           |
| `guac.visualizer.enabled`                                      | String Whether to deploy the visualizer.                                                     | `true`                                         |
| `guac.visualizer.name`                                         | String Name of the visualizer.                                                               | `visualizer`                                   |
| `guac.visualizer.annotations.reloader.stakater.com/auto`       | Boolean for deploying [stakater/Reloader] (https://github.com/stakater/Reloader)             | `""`                                           |
| `guac.visualizer.replicas`                                     | Number of replicas for visualizer deployment                                                 | `1`                                            |
| `guac.visualizer.image.repository`                             | Path to the Ingestor image                                                                   | `ghcr.io/guacsec/guac-visualizer`              |
| `guac.visualizer.image.tag`                                    | Tag if using an image tag.  Optional                                                         | `v0.0.3`                                       |
| `guac.visualizer.image.digest`                                 | Sha256 Image Digest.  It is strongly recommended to use this for verification.               | `""`                                           |
| `guac.visualizer.image.pullPolicy`                             | ImagePullPolicy for kubernetes                                                               | `IfNotPresent`                                 |
| `guac.visualizer.image.ports[0].containerPort`                 | Port the visualizer container listens on                                                     | `3000`                                         |
| `guac.visualizer.svcPorts[0].protocol`                         | Protocol used at the visualizer                                                              | `TCP`                                          |
| `guac.visualizer.svcPorts[0].port`                             | Port the visualizer service listens on                                                       | `3000`                                         |
| `guac.visualizer.svcPorts[0].targetPort`                       | Port the visualizer container listens on                                                     | `3000`                                         |
| `guac.visualizer.env`                                          | Environment variables for the visualizer.                                                    | `[]`                                           |
| `guac.visualizer.nodeSelector`                                 | - sets the node selector for where to run the deployment                                     | `{}`                                           |
| `guac.visualizer.tolerations`                                  |                                                                                              | `[]`                                           |
| `guac.observability.deployServiceMonitor`                      | Boolean Deploy the service monitor for observability                                         | `false`                                        |
| `guac.sampleData.ingest`                                       | Boolean Whether to ingest sample data after deployment                                       | `false`                                        |
| `guac.sampleData.jobName`                                      | Name of the sample data ingest job                                                           | `ingest-guac-data`                             |
| `guac.sampleData.env`                                          | Environment variables for the sample data ingest job                                         | `[]`                                           |
| `guac.ingress.enabled`                                         | Whether to deploy an Ingress object                                                          | `false`                                        |
| `guac.ingress.ingressClassName`                                | Ingress class name                                                                           | `undefined`                                    |
| `guac.ingress.webuiHostname`                                   | DNS name for the UI components - e.g. Visualizer, GQL playground                             | `undefined`                                    |
| `guac.ingress.apiHostname`                                     | DNS name for the GQL API. When specified, GQL API won't be served at webuiHostname           | `undefined`                                    |
| `guac.ingress.annotations`                                     | Annotations for the ingress object                                                           | `{}`                                           |
| `guac.apiOnlyIngress.enabled`                                  | Whether to deploy an Ingress object to expose API only                                       | `false`                                        |
| `guac.apiOnlyIngress.ingressClassName`                         | Ingress class name for API only ingress                                                      | `undefined`                                    |
| `guac.apiOnlyIngress.apiHostname`                              | DNS name for the GQL API.                                                                    | `undefined`                                    |
| `guac.apiOnlyIngress.annotations`                              | Annotations for the API only ingress object                                                  | `{}`                                           |
| `guac.traefikIngressRoute.enabled`                             | Whether to deploy Traefik IngressRoute object                                                | `false`                                        |
| `guac.backend.ent.db-driver`                                   | database driver to use, one of [postgres | sqlite3 | mysql] or anything supported by sql.DB  | `postgres`                                     |
| `guac.backend.ent.db-address`                                  | Full URL of database to connect to                                                           | `undefined`                                    |
| `guac.backend.ent.db-migrate`                                  | Wether to automatically run database migrations on start                                     | `true`                                         |
| `guac.backend.ent.db-debug`                                    | Enable debug logging for database queries                                                    | `true`                                         |
| `guac.pubSubAddr`                                              | String gocloud connection string for pubsub configured via https://gocloud.dev/howto/pubsub/ | `undefined`                                    |
| `guac.collectorPublishToQueue`                                 | Whether to publish ingestion message to pubsub queue                                         | `true`                                         |
| `guac.blobAddr`                                                | gocloud connection string for blob store configured via https://gocloud.dev/howto/blob/      | `undefined`                                    |
| `guac.additionalResources`                                     |                                                                                              | `{}`                                           |

### nats

This is the configuration for nats.  This is a subchart.  See full documentation [here](https://docs.nats.io/running-a-nats-service/nats-kubernetes/helm-charts).

| Name                                                       | Description                                                                                       | Value                                                                                                                                                                                                                                                                                                                                                                                                               |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `nats.enabled`                                             | Whether to deploy nats                                                                            | `true`                                                                                                                                                                                                                                                                                                                                                                                                              |
| `nats.nats.jetstream.enabled`                              | Boolean for enabling JetStream.                                                                   | `true`                                                                                                                                                                                                                                                                                                                                                                                                              |
| `nats.nats.limits.maxPayload`                              | Max Payload size for nats                                                                         | `64MB`                                                                                                                                                                                                                                                                                                                                                                                                              |
| `nats.nats.statefulSetPodLabels.app.kubernetes.io/part-of` | Label to associate nats with GUAC for monitoring purposes                                         | `{"enabled":true,"nats":{"jetstream":{"enabled":true},"limits":{"maxPayload":"64MB"},"statefulSetPodLabels":{"app.kubernetes.io/part-of":"guac"}},"natsbox":{"enabled":false,"additionalLabels":{"app.kubernetes.io/part-of":"guac"},"podLabels":{"app.kubernetes.io/part-of":"guac"}},"exporter":{"enabled":false,"serviceMonitor":{"enabled":false,"namespace":"monitoring","labels":{"release":"monitoring"}}}}` |
| `nats.natsbox.enabled`                                     | Whehter to run natsbox                                                                            | `false`                                                                                                                                                                                                                                                                                                                                                                                                             |
| `nats.natsbox.additionalLabels.app.kubernetes.io/part-of`  | Label to associate natsbox with GUAC for monitoring purposes                                      | `guac`                                                                                                                                                                                                                                                                                                                                                                                                              |
| `nats.natsbox.podLabels.app.kubernetes.io/part-of`         | Label to associate natsbox with GUAC for monitoring purposes                                      | `guac`                                                                                                                                                                                                                                                                                                                                                                                                              |
| `nats.exporter.enabled`                                    | Boolean to enable data collection                                                                 | `false`                                                                                                                                                                                                                                                                                                                                                                                                             |
| `nats.exporter.serviceMonitor.enabled`                     | Boolean to enable nats service monitor                                                            | `false`                                                                                                                                                                                                                                                                                                                                                                                                             |
| `nats.exporter.serviceMonitor.namespace`                   | String nats service monitor namespace - this is for monitoring purposes and is used by Prometheus | `monitoring`                                                                                                                                                                                                                                                                                                                                                                                                        |
| `nats.exporter.serviceMonitor.labels.release`              | Label to associate nats service monitor with GUAC for monitoring purposes                         | `monitoring`                                                                                                                                                                                                                                                                                                                                                                                                        |

### minio

This is the configuration for minio.  This is a subchart.  See full documentation [here](https://github.com/minio/minio/tree/master/helm/minio).

| Name                 | Description                                                                    | Value          |
| -------------------- | ------------------------------------------------------------------------------ | -------------- |
| `minio.enabled`      | Whehter to deploy minio as part of the Helm deployment                         | `true`         |
| `minio.replicas`     | Number of replicas.                                                            | `1`            |
| `minio.persistence`  | Persistence volume configuration.                                              | `{}`           |
| `minio.mode`         | minio mode, i.e. standalone or distributed                                     | `standalone`   |
| `minio.resources`    | resource requests and limits                                                   | `{}`           |
| `minio.rootUser`     | root user name.                                                                | `rootUser`     |
| `minio.rootPassword` | root user password.                                                            | `rootPassword` |
| `minio.buckets`      | List of buckets to create after deployment.                                    | `{}`           |
| `minio.users`        | List of users, in terms of creds and permissions, to create after deployment.? | `{}`           |

## Developing
For running the unit tests, install the unittest plugin.

`helm plugin install https://github.com/quintush/helm-unittest`

To run unit tests

`helm unittest charts/guac -3`

To run Helm chart-testing (ct) lint and install tests

`ct install --all --helm-extra-args --timeout=600s`