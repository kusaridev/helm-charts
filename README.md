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

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `neo4j-password` | Initial password for neo4j | `s3cr3ts3cr3t`|

# Developing

In order to ensure proper development, run the following

`helm plugin install https://github.com/quintush/helm-unittest`

To Run unit tests

`helm unittest . -3`