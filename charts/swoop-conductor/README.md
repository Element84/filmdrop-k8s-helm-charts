# Deployment

This helm chart will deploy [SWOOP Conductor](https://github.com/Element84/swoop-go) onto a Kubernetes cluster.

## Adding FilmDrop Helm Chart Repository
To add the FilmDrop Helm Chart repository, do:

`helm repo add e84 https://element84.github.io/filmdrop-k8s-helm-charts`


## Installing SWOOP Conductor and its dependencies
The [SWOOP Conductor](https://github.com/Element84/swoop-go) will need an object storage for workflow artifacts and a postgres state database present, as well as swoop-caboose and swoop-api for testing purposes. You will need to install the following helm charts in order:
* minio
* postgres
* swoop-db-init
* swoop-db-migration
* swoop-caboose
* swoop-api
* swoop-conductor
* workflow-config

You can either choose to install the MinIO and Postgres Helm Chart available on the FilmDrop Helm Chart Repository or you will need to have an existing MinIO/S3 backend with a Postgres installed and reachable to your SWOOP Caboose.

To install the MinIO dependency run:
`helm install minio e84/minio`

To install the Postgres dependency:
`helm install postgres e84/postgres`

For waiting for the Postgres pods to be ready and initialize them prior initializing SWOOP DB:
```
kubectl wait --for=condition=ready --timeout=30m pod -l app=postgres
```

To initialize SWOOP DB run:
`helm install swoop-db-init e84/swoop-db-init`

For waiting for the SWOOP DB initialization to complete run:
```
kubectl wait --for=condition=complete --timeout=30m job -l app=swoop-db-init
```

To apply migration on SWOOP DB run:
`helm install swoop-db-migration e84/swoop-db-migration`

For waiting for the SWOOP DB migration to complete run:
```
kubectl wait --for=condition=complete --timeout=30m job -l app=swoop-db-migration
```

To install SWOOP Caboose run:
`helm install swoop-caboose e84/swoop-caboose`

For waiting for the SWOOP Caboose pods to be ready and initialize them prior installing SWOOP Conductor:
```
kubectl wait --for=condition=ready --timeout=30m pod -l app=swoop-caboose
```

To install SWOOP API run:
`helm install swoop-api e84/swoop-api`

For waiting for the SWOOP API pods to be ready and initialize them prior installing SWOOP Conductor:
```
kubectl wait --for=condition=ready --timeout=30m pod -l app=swoop-api
```

To install SWOOP API run:
`helm install swoop-conductor e84/swoop-conductor`

For waiting for the SWOOP Conductor pods to be ready:
```
kubectl wait --for=condition=ready --timeout=30m pod -l app=swoop-conductor
```


<br></br>
## Uninstall swoop-conductor

To uninstall the release, do `helm uninstall swoop-conductor`.
