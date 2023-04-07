# Deployment

This helm chart will deploy [STAC Fast-API](https://github.com/stac-utils/stac-fastapi) onto a Kubernetes cluster.

Once the chart has been deployed using the `helm install` command, you first need to port-forward the `pgstac` service onto the local port `5439`. This can be done in Rancher Desktop via the 'Port-Forwarding' tab or via the `kubectl port-forward` command. You can view the database along with all of its schemas, functions, and tables in PgAdmin (make sure to use the port `5439` in the connection parameters).

Then, port-forward the `stac-fastapi-pgstac` service onto any localhost port, such as `8080`. Navigate to `http://localhost:8080/` in a web browser, where you will then see the STAC Fast-API application API page.
