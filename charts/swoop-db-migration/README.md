# Deployment

This helm chart will apply a migration version to [swoop-db](https://github.com/Element84/swoop-db).

## Adding FilmDrop Helm Chart Repository
To add the FilmDrop Helm Chart repository, do:

`helm repo add e84 https://element84.github.io/filmdrop-k8s-helm-charts`


## Installing and Initializing SWOOP DB
To install the Postgres dependency run:
`helm install postgres e84/postgres`

For waiting for the Postgres pods to be ready and initialize them prior initializing SWOOP DB:
```
kubectl wait --for=condition=ready --timeout=30m pod -l app=postgres
```

To initialize SWOOP DB run:
`helm install swoop-db-init e84/swoop-db-init`

Once the chart has been deployed, you should see the postgres pods and the initialization job completed:

```
$ kubectl get pods

NAME                                               READY   STATUS      RESTARTS   AGE
default       postgres-local-path-provisioner-6f78964c6d-bpdg7   1/1     Running     0          16h
default       postgres-7b6b9ff76-64l9m                           1/1     Running     0          16h
default       db-initialization-8-q4wnp                          0/1     Completed   0          2m16s
default       wait-for-db-initialization-8-zcvfc                 0/1     Completed   0          2m16s
```

Wait for SWOOP DB initialization to complete with:
```
kubectl wait --for=condition=complete --timeout=30m job -l app=swoop-db-init
```

And looking the logs in the db-initialization job, you should see that the swoop roles were created:
```
$ kubectl logs db-initialization-8-q4wnp

Creating owner role 'swoop'...
Creating application role API_ROLE...
Creating application role CABOOSE_ROLE...
Creating application role CONDUCTOR_ROLE...
Creating database 'swoop'...
```

## Apply migration on SWOOP DB
To apply migration on SWOOP DB run:
`helm install swoop-db-migration e84/swoop-db-migration`

Wait for SWOOP DB migration to complete with:
```
kubectl wait --for=condition=complete --timeout=30m job -l app=swoop-db-migration
```

Once the chart has been deployed, you should see the postgres pods and the migration job completed:
```
$ kubectl get pods

NAME                                               READY   STATUS      RESTARTS   AGE
default       postgres-local-path-provisioner-6f78964c6d-bpdg7   1/1     Running     0          16h
default       postgres-7b6b9ff76-64l9m                           1/1     Running     0          16h
default       db-initialization-8-q4wnp                          0/1     Completed   0          4m33s
default       wait-for-db-initialization-8-zcvfc                 0/1     Completed   0          4m33s
default       migration-job-8-mdpx6                              0/1     Completed   0          23s
```

And looking the logs in the db-initialization job, you should see that the swoop roles were created:
```
$ kubectl logs migration-job-8-mdpx6 

Migrating database to version 8
```

<br></br>

## Uninstall swoop-db-migration

To uninstall the release, do `helm uninstall swoop-db-migration`.
