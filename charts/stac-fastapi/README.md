# Deployment

This helm chart will deploy [STAC Fast-API](https://github.com/stac-utils/stac-fastapi) onto a Kubernetes cluster.

To install the chart, do:

`helm repo add e84 https://element84.github.io/filmdrop-k8s-helm-charts`

and

`helm install stac e84/stac-fastapi`


Once the chart has been deployed, port-forward the `pgstac` service onto the localhost port `5439`. Then, port-forward the `stac-fastapi-pgstac` service onto another localhost port, such as `8080`. This can be done in Rancher Desktop via the 'Port-Forwarding' tab or via the `kubectl port-forward` command (see below).
<br></br>
<p align="center">
  <img src="../../images/port-forward-stacfastapi.png" alt="Port forwarding STAC FastAPI" width="1776">
</p>
<br></br>

`kubectl port-forward service/pgstac 5439:5439`

`kubectl port-forward service/stac-fastapi-pgstac 8080:8080`
<br></br>
Navigate to `http://localhost:8080/` in a web browser, where you will then see the STAC Fast-API landing page.
<br></br>
You can view the database along with all of its schemas, functions, and tables in PgAdmin (make sure to use the port `5439` in the database server connection parameters).

To uninstall the release, do `helm delete stac`.