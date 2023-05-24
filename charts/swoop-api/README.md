# Deployment

This helm chart will deploy [SWOOP API](https://github.com/Element84/swoop) onto a Kubernetes cluster.

To install the chart, do:

`helm repo add e84 https://element84.github.io/filmdrop-k8s-helm-charts`

and

`helm install swoop-api e84/swoop-api`


To uninstall the release, do `helm delete swoop-api`.