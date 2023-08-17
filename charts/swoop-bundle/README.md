# Deployment

This helm chart will the following components into a kubernetes cluster:
* [swoop-api](https://github.com/Element84/swoop)
* [argo-workflows](https://github.com/argoproj/argo-workflows/)
* [swoop-caboose](https://github.com/Element84/swoop-go)
* [swoop-conductor](https://github.com/Element84/swoop-go)

## Adding FilmDrop Helm Chart Repository
To add the FilmDrop Helm Chart repository, do:

`helm repo add e84 https://element84.github.io/filmdrop-k8s-helm-charts`


## Installing SWOOP and its dependencies
The [SWOOP API](https://github.com/Element84/swoop) will need an object storage for workflow artifacts and a postgres state database present.

You can either choose to install the MinIO and Postgres Helm Chart available on the FilmDrop Helm Chart Repository or you will need to have an existing MinIO/S3 backend with a Postgres installed and reachable to your SWOOP API.

To install the MinIO dependency run:
`helm install minio e84/minio`

To install the Postgres dependency run:
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

To install SWOOP components:
`helm install swoop-bundle e84/swoop-bundle`

Once the chart has been deployed, you should see at least 3 deployments: postgres, minio and swoop-api.
<br></br>
<p align="center">
  <img src="../.../../images/swoop-api-deployment-services.png" alt="SWOOP Deployment" width="1776">
</p>
<br></br>

In order to start using the services used by this helm chart, you will need to port-forward `postgres` onto localhost port `5432`, port-forward `minio` onto localhost ports `9000` & `9001` and port-forward `swoop-api` onto localhost port `8000`.
<br></br>
<p align="center">
  <img src="../.../../images/swoop-api-port-forwarding.png" alt="Port forwarding SWOOP" width="1776">
</p>
<br></br>

You will see now, that if you reach the swoop api [http://localhost:8000/](http://localhost:8000/), you should see a sample response:
```
$ curl http://localhost:8000/

{"title":"Example processing server","description":"Example server implementing the OGC API - Processes 1.0 Standard","links":[{"href":"http://localhost:8000/conformance","rel":"http://www.opengis.net/def/rel/ogc/1.0/conformance","type":"application/json","hreflang":null,"title":null}]}%
```
<br></br>

## API tests with Database

To test the API endpoints that make use of data in the postgres database, you will need to load data into the postgres state database or use [swoop-db](https://github.com/Element84/swoop-db) to initialize the schema and load test migrations.

If you want database sample data to test the API, run the following swoop-db command on the postgres pods to load test fixtures:
```
kubectl exec -it --namespace=default svc/postgres  -- /bin/sh -c "swoop-db load-fixture base_01"
```

After loading the database, you should be able to see the jobs in the swoop api jobs endpoint [http://localhost:8000/jobs/](http://localhost:8000/jobs/):
```
$ curl http://localhost:8000/jobs/

{"jobs":[{"processID":"action_2","type":"process","jobID":"81842304-0aa9-4609-89f0-1c86819b0752","status":"accepted","message":null,"created":null,"started":null,"finished":null,"updated":"2023-04-28T15:49:00+00:00","progress":null,"links":null,"parentID":"2595f2da-81a6-423c-84db-935e6791046e"},{"processID":"action_1","type":"process","jobID":"2595f2da-81a6-423c-84db-935e6791046e","status":"successful","message":null,"created":null,"started":null,"finished":null,"updated":"2023-04-28T15:49:03+00:00","progress":null,"links":null,"parentID":"cf8ff7f0-ce5d-4de6-8026-4e551787385f"}],"links":[{"href":"http://www.example.com","rel":null,"type":null,"hreflang":null,"title":null}]}%
```

## API tests with Object Storage
In order to load data into MinIO, follow these steps:

### Install First the MinIO client by running:
```
brew install minio/stable/mc
```

### Then set the MinIO alias, find the ACCESS_KEY and SECRET_KEY by quering the Helm values
```
export MINIO_ACCESS_KEY=`helm get values minio -a -o json | jq -r .service.accessKeyId | base64 --decode`
export MINIO_SECRET_KEY=`helm get values minio -a -o json | jq -r .service.secretAccessKey | base64 --decode`
mc alias set swoopminio http://127.0.0.1:9000 $MINIO_ACCESS_KEY $MINIO_SECRET_KEY
```

### Test MinIO connection by running:
```
$ mc admin info swoopminio

●  127.0.0.1:9000
   Uptime: 23 minutes
   Version: 2023-06-02T23:17:26Z
   Network: 1/1 OK
   Drives: 1/1 OK
   Pool: 1

Pools:
   1st, Erasure sets: 1, Drives per erasure set: 1

0 B Used, 1 Bucket, 0 Objects
1 drive online, 0 drives offline
```

### Load data into MinIO by running:
First clone the [https://github.com/Element84/swoop](https://github.com/Element84/swoop) repository locally, and then run the following from the top level of the your local swoop clone:

```
$ mc cp --recursive tests/fixtures/io/base_01/ swoopminio/swoop/execution/2595f2da-81a6-423c-84db-935e6791046e/

...fixtures/io/base_01/output.json: 181 B / 181 B ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.67 KiB/s 0s
```

### View your data on MinIO by opening your browser on [http://localhost:9001/](http://localhost:9001/) and logging into MinIO
<p align="center">
  <img src="../.../../images/swoop-minio-data.png" alt="SWOOP MinIO" width="1776">
</p>
<br></br>

### Test API with MinIO by running:
```
$ curl http://localhost:8000/jobs/2595f2da-81a6-423c-84db-935e6791046e/payload

{"process_id":"2595f2da-81a6-423c-84db-935e6791046e","payload":"test_input"}%
```

```
$ curl http://localhost:8000/jobs/2595f2da-81a6-423c-84db-935e6791046e/results

{"process_id":"2595f2da-81a6-423c-84db-935e6791046e","payload":"test_output"}%
```
<br></br>

## Verifying SWOOP Caboose installation
Check the logs of the swoop-caboose pod and check your workers have started via:
```
$ kubectl get pods

NAME                                                              READY   STATUS    RESTARTS   AGE
minio-7c94499969-vm5ck                                            1/1     Running   0          6m37s
postgres-local-path-provisioner-6f78964c6d-ljpvm                  1/1     Running   0          6m16s
postgres-5b69c5f5d-dq9bv                                          1/1     Running   0          6m16s
swoop-bundle-argo-workflows-workflow-controller-6b64d7d9b-c4mbh   1/1     Running   0          2m37s
swoop-api-5575bc957d-5bnnk                                        1/1     Running   0          2m37s
swoop-caboose-77fc87775f-8lrl7                                    1/1     Running   0          2m37s
swoop-bundle-argo-workflows-server-56cc4f47f9-bgqtg               1/1     Running   0          2m37s
```

```
$ kubectl logs swoop-caboose-77fc87775f-8lrl7

time="2023-07-31T14:40:28Z" level=info msg="index config" indexWorkflowSemaphoreKeys=true
2023/07/31 14:40:28 starting worker 0
2023/07/31 14:40:28 starting worker 1
2023/07/31 14:40:28 starting worker 3
2023/07/31 14:40:28 starting worker 2
```
<br></br>


## Running an Argo workflow
For full documentation for argo workflows, please visit the [Argo Workflows Official Documentation](https://argoproj.github.io/argo-workflows/).

For a full list of customization values for the argo-workflows helm chart, visit [Argo Workflows ArtifactHub](https://artifacthub.io/packages/helm/argo/argo-workflows).

### Pre-requisites to running Argo Workflows example
You will need an AWS account, with an S3 bucket and an IAM user with read/write access to that bucket.

1. Go to your account in AWS, and create an S3 bucket for your assets, for example:
<br></br>
<p align="center">
  <img src="../../images/argo_assets_bucket.png" alt="Argo Assets Bucket" width="1776">
</p>
<br></br>

2. Create an IAM user with a read/write policy to your bucket.
<br></br>
<p align="center">
  <img src="../../images/argo_iam_user.png" alt="Argo IAM User" width="1776">
</p>
<p align="center">
  <img src="../../images/argo_iam_user_permissions.png" alt="Argo IAM User Permissions" width="1776">
</p>
<br></br>

### Installation of Argo Workflows
Argo Workflows have been included as part of the swoop-bundle, as a dependency of swoop-caboose. To deploy argo workflows deploy the following helm charts in the same order:

To add the FilmDrop Helm Chart repository, do:

`helm repo add e84 https://element84.github.io/filmdrop-k8s-helm-charts`


To install the MinIO dependency run:
`helm install minio e84/minio`

To install the Postgres dependency run:
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

To install SWOOP components:
`helm install swoop-bundle e84/swoop-bundle`

### Running Argo Workflows Copy Assets Stac Task
The [copy-assets-stac-task](https://github.com/Element84/copy-assets-stac-task) example will run an argo workflow which copies specified Assets from Source STAC Item(s), uploads them to S3 and updates Item assets hrefs to point to the new location.

Now, prior to running the argo workflows example, first make sure to port-forward `minio` onto localhost ports `9000` & `9001`.

Via Rancher Desktop:
<br></br>
<p align="center">
  <img src="../../images/minio-port-forwarding-bundle.png" alt="Port forwarding Minio" width="1776">
</p>
<br></br>

or via terminal:

```
kubectl port-forward svc/minio 9000:9000 &
kubectl port-forward svc/minio 9001:9001 &
```

#### Install First the MinIO client by running

```
brew install minio/stable/mc
```

#### Then set the MinIO alias, find the ACCESS_KEY and SECRET_KEY by quering the Helm values

```
export MINIO_ACCESS_KEY=`helm get values minio -a -o json | jq -r .service.accessKeyId | base64 --decode`
export MINIO_SECRET_KEY=`helm get values minio -a -o json | jq -r .service.secretAccessKey | base64 --decode`
mc alias set swoopminio http://127.0.0.1:9000 $MINIO_ACCESS_KEY $MINIO_SECRET_KEY
```

####  Run the [copy-assets-stac-task](https://github.com/Element84/copy-assets-stac-task) argo workflow example

First clone the [copy-assets-stac-task](https://github.com/Element84/copy-assets-stac-task) repository.

After cloning the [copy-assets-stac-task](https://github.com/Element84/copy-assets-stac-task) repository, first proceed to modify the `payload_workflow.json` and replace the `<REPLACE_WITH_ASSETS_S3_BUCKET_NAME>` with the bucket name created on the [Pre-requisites section](#pre-requisites-to-running-argo-workflows-example).

Create a public minio local bucket and copy the `payload_workflow.json` file after replacing the `<REPLACE_WITH_ASSETS_S3_BUCKET_NAME>` name. To do this via the minio client:

```
$ mc mb swoopminio/payloadtest

Bucket created successfully `swoopminio/payloadtest`.
```
```
$ mc anonymous set public swoopminio/payloadtest

Access permission for `swoopminio/payloadtest` is set to `public`
```
```
$ mc cp ./payload_workflow.json swoopminio/payloadtest/

...opy-assets-stac-task/payload_workflow.json: 4.58 KiB / 4.58 KiB ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 127.61 KiB/s 0s
```

Now modify the `secrets.yaml` and replace the `<REPLACE_WITH_BASE64_AWS_ACCESS_KEY>`, `<REPLACE_WITH_BASE64_AWS_SECRET_ACCESS_KEY>` and `<REPLACE_WITH_BASE64_AWS_REGION>` with the base64 encoded version of the AWS_ACCESS_KEY, AWS_SECRET_ACCESS_KEY and AWS_REGION of the iam user and bucket created on the [Pre-requisites section](#pre-requisites-to-running-argo-workflows-example).

Create the kubernetes secret by executing:
```
kubectl apply -n default -f ./secret.yaml
```

Then modify the `<REPLACE_WITH_MINIO_HOST>:<REPLACE_WITH_MINIO_PORT>` variables inside the `workflow_copyassets_no_template.yaml` and the `workflow_copyassets_with_template.yaml`. If you're running this example from a minio created by the terraform stack, you should be able to replace `<REPLACE_WITH_MINIO_HOST>:<REPLACE_WITH_MINIO_PORT>` with `minio.default:9000`.

Finally, run and watch the argo workflow task via:
```
argo submit -n default --watch ./workflow_copyassets_no_template.yaml
```

You should be able to see your workflow pod succeed in the terminal:
<br></br>
<p align="center">
  <img src="../../images/argo_workflow_success.png" alt="Argo Workflow Success" width="776">
</p>
<br></br>

And you should be able to see your asset S3 bucket populated with a tumbnail image:
```
aws s3 ls s3://copy-assets-stac-task-bucket/data/naip/tx_m_2609719_se_14_060_20201217/

2023-08-17 10:00:08       9776 thumbnail.jpg
```
<br></br>


Notes:
* When utilizing the argo workflow installation provided via the swoop-bundle, you should be able to run argo workflows in the following manner:
```
argo submit -n default --watch <FULL PATH TO THE SAMPLE WORKFLOW YAML FILE>
```
* The're is a service account created to support the required argo permissions `serviceAccountName: argo`
* You should not expect the argo server, nor archive logs to be functional with the default argo installation. In order to enable those settings please see:
  * [Argo Workflows Official Documentation](https://argoproj.github.io/argo-workflows/)
  * [Argo Workflows ArtifactHub](https://artifacthub.io/packages/helm/argo/argo-workflows)


<br>


## Uninstall swoop-bundle

To uninstall the release, do `helm uninstall swoop-bundle`.
