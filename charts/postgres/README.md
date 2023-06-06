# Deployment

This helm chart will deploy Postgres as a state database for [SWOOP API](https://github.com/Element84/swoop) onto a Kubernetes cluster.

To install the chart, do:

`helm repo add e84 https://element84.github.io/filmdrop-k8s-helm-charts`

and

`helm install postgres e84/postgres`

Once the chart has been deployed, you should see at least 1 deployment for postgres.
<br></br>
<p align="center">
  <img src="../../images/postgres-deployment-services.png" alt="Postgres Deployment" width="1776">
</p>
<br></br>

In order to start using the services used by this helm chart, you will need to port-forward `postgres` onto localhost port `5432`.
<br></br>
<p align="center">
  <img src="../../images/postgres-port-forwarding.png" alt="Port forwarding Postgres" width="1776">
</p>
<br></br>

## Test connection to postgres database

For a comprehensive set of steps initialize and use the postgres database, please refer to the swoop-db repo [https://github.com/Element84/swoop-db](https://github.com/Element84/swoop-db).

To test the connection to the postgres database, export the following postgres environment variables:
```
export PGHOST="127.0.0.1"
export PGUSER="`helm get values postgres -a -o json | jq -r .postgres.service.dbUser | base64 --decode`"
export PGPASSWORD="`helm get values postgres -a -o json | jq -r .postgres.service.dbPassword | base64 --decode`"
export PGPORT="`helm get values postgres -a -o json | jq -r .postgres.service.port`"
export PGDATABASE="`helm get values postgres -a -o json | jq -r .postgres.service.dbName`"
export PGAUTHMETHOD="trust"

```

Then connect to the postgres database by running a psql command:
```
$ psql -p $PGPORT -U $PGUSER $PGDATABASE

psql (14.7 (Homebrew), server 15.3 (Debian 15.3-1.pgdg110+1))
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
Type "help" for help.

swoop=#
```


## Uninstall postgres

To uninstall the release, do `helm uninstall postgres`.
