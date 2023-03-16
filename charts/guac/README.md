# GUAC Helm Chart

This is a [Helm Chart](https://helm.sh/docs/topics/charts/) for deploying [GUAC](https://github.com/guacsec/guac) to [Kubernetes](https://kubernetes.io/).  This chart offers the following features
* Neo4j deployment - community or enterprise
  

## Prerequisites

* Kubernetes 1.19+
* Helm v3.9.4+


## Usage

[Helm](https://helm.sh) must be installed to use the charts.  Please refer to
Helm's [documentation](https://helm.sh/docs) to get started.

Once Helm has been set up correctly, add the repo as follows:

  helm repo add kusaridev https://kusaridev.github.io/guac-helm-charts

If you had already added this repo earlier, run `helm repo update` to retrieve
the latest versions of the packages.  You can then run `helm search repo
kusaridev` to see the charts.

To install the guac chart:

    helm install my-guac kusaridev/guac

To uninstall the chart:

    helm delete my-guac


## Configuration
__Content to be generated__


# Developing

In order to ensure proper development, run the following

`helm plugin install https://github.com/quintush/helm-unittest`

To Run unit tests

`helm unittest . -3`
