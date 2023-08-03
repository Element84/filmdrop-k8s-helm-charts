# Deployment

This helm chart will initialize [swoop-db](https://github.com/Element84/swoop-db).

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
postgres-local-path-provisioner-6f78964c6d-ljgtw   1/1     Running     0          11m
postgres-5b69c5f5d-fj254                           1/1     Running     0          11m
db-initialization-8-dp9rp                          0/1     Completed   0          3m50s
wait-for-db-initialization-8-6zfdt                 0/1     Completed   0          3m50s
```

And looking the logs in the db-initialization job, you should see that the swoop roles were created:
```
$ kubectl logs db-initialization-8-dp9rp

Creating owner role 'user_owner'...
Creating application role API_ROLE...
Creating application role CABOOSE_ROLE...
Creating application role CONDUCTOR_ROLE...
Role already exists, skipping.
Creating database 'swoop'...
```
<br></br>

## Uninstall swoop-db-init

To uninstall the release, do `helm uninstall swoop-db-init`.
