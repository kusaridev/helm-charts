# GUAC Helm Chart

This is a [Helm Chart](https://helm.sh/docs/topics/charts/) for deploying [GUAC](https://github.com/guacsec/guac) to [Kubernetes](https://kubernetes.io/).  This chart offers the following features
* Neo4j deployment - community or enterprise
  

## Prerequisites

* Kubernetes 1.19+
* Helm v3.9.4+

## Install

`helm install --name guac guac-helm-chart`

## Uninstall

`helm delete guac`

## Configuration

The following are parameters that can be configured, as well as their default values

| Component | Parameter | Description | Default |
| --------- | --------- | ----------- | ------- |
| global | `guac.workingDir` | Working directory for images | `/guac` |
| global | `guac.imagePullSecrets` | Docker secret for images where auth is needed | none |
| Collectsub | `guac.collectsub.name` | Name of the collectsub component | `collectsub` |
| Collectsub | `guac.collectsub.annotations` | Annotations for deployment | `reloader.stakater.com/auto: "true"` |
| Collectsub | `guac.collectsub.replicas` | Replica count for deployment | `1` |
| Collectsub | `guac.collectsub.image.repository` | Path to collectsub image | `ghcr.io/kusaridev/local-organic-guac` |
| Collectsub | `guac.collectsub.image.digest` | Image Digest for collectsub| `sha256:4ceb73778530d652755777c6e81de6994f7f94e103ee4a3ff55b797e813ac646` |
| Collectsub | `guac.collectsub.image.tag` | Image tag, used if no digest is present | none |
| Collectsub | `guac.collectsub.image.command` | Command for collectsub | `sh -c /opt/guac/guacone csub-server` |
| Collectsub | `guac.collectsub.image.ports` | Port collectsub listens on | `2782` |
| GraphQL Server | `guac.graphql_server.name` | Name of the GraphQL component | `graphql-server` |
| GraphQL Server | `guac.graphql_server.annotations` | Annotations for deployment | `reloader.stakater.com/auto: "true"` |
| GraphQL Server | `guac.graphql_server.replicas` | Replica count for deployment | `1` |
| GraphQL Server | `guac.graphql_server.image.repository` | Path to GraphQL image | `ghcr.io/kusaridev/local-organic-guac` |
| GraphQL Server | `guac.graphql_server.image.digest` | Image Digest for GraphQL| `sha256:4ceb73778530d652755777c6e81de6994f7f94e103ee4a3ff55b797e813ac646` |
| GraphQL Server | `guac.graphql_server.image.tag` | Image tag, used if no digest is present | none |
| GraphQL Server | `guac.graphql_server.image.command` | Command for GraphQL | `sh -c /opt/guac/guacone gql-server` |
| GraphQL Server | `guac.graphql_server.image.ports` | Port GraphQL listens on | `8080` |

# Developing

In order to ensure proper development, run the following

`helm plugin install https://github.com/quintush/helm-unittest`

To Run unit tests

`helm unittest . -3`