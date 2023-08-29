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

## Installing and configuring `workflow-config` helm chart

Create a bucket in S3 that will be used to hold your output assets from the workflow. Create an IAM user that has read/write permissions to this bucket and generate a set of AWS credentials (Access Key ID and Secret Access Key) for this user. These AWS credentials should have permissions to write to an S3 bucket where the output assets from the workflow will be uploaded.

Create an `input.json` file for the input payload.

Modify the `process/upload_options/path_template`property of the input payload to contain the bucket name of the S3 bucket created in the previous step. For example, if the bucket is named `mirrorworkflowoutput`, the `path_template` should be:

`"path_template": "s3://mirrorworkflowoutput/data/${collection}/${id}/"`

This is a sample UUID used for this example. Enter this in the command line.
`UUID='a5db5e5e-4d5d-45c5-b9db-fb6f4aa93b0a'`

Upload the `input.json` file to a MinIo bucket. Replace the <PATH_TO_INPUT.JSON> with the path to your `input.json` file.

```
export MINIO_ACCESS_KEY=`helm get values minio -a -o json | jq -r .service.accessKeyId | base64 --decode`
export MINIO_SECRET_KEY=`helm get values minio -a -o json | jq -r .service.secretAccessKey | base64 --decode`
mc alias set swoopminio http://127.0.0.1:9000 $MINIO_ACCESS_KEY $MINIO_SECRET_KEY
mc cp  <PATH_TO_INPUT.JSON> swoopminio/swoop/executions/${UUID}/input.json
```

Then, modify the `values.yaml` file for the `workflow-config` helm chart:

- Update the `artifactBucketName` value with the name of the MinIO bucket that contains the `input.json` file and will also eventually contain the `output.json` file once the workflow has completed.
- Update the `AWS Access Key ID`, `AWS Secret Access Key`, and `AWS Region` and/or `AWS Session Token` under the keys `s3.accessKeyId`, `s3.secretAccessKey`, `s3.region`, `s3.sessionToken`, respectively. These values contain dummy data and should be modified to include your AWS credentials. These AWS credentials should have permissions to write to an S3 bucket where the output assets from the workflow will be uploaded. Make sure you are using base-64 encoding for these values.
- Update the `minio.service.endpoint` value with the hostname and port name of the MinIO service. If you deployed MinIO from the helm chart, this should be `minio.default:9000`, respectively.
- Update the `serviceAccountName` with the name of the K8s service account that you want to use to run the workflow. Make sure that this account has the necessary permissions it needs.

Then, do:

`helm install workflow-config e84/workflow-config`

For more information about installing and configuring the `workflow-config` helm chart, please see the [readme](../workflow-config/README.md).

<br></br>

## Testing `swoop-conductor`
First port-forward swoop-api to your local port 8000:
```
kubectl port-forward -n default svc/swoop-api 8000:8000 &
```

Then clone the [copy-assets-stac-task](https://github.com/Element84/copy-assets-stac-task) repository locally, and then run the following from the top level of the your local copy-assets-stac-task clone:
```
python3 swoop_api_payload_test.py
```

The script will ask you for 2 inputs:
```
Enter SWOOP API Host (localhost:8000): localhost:8000
Enter STAC ASSETS BUCKET NAME: <REPLACE_WITH_ASSETS_S3_BUCKET_NAME>
```

Replace the <REPLACE_WITH_ASSETS_S3_BUCKET_NAME>` with an S3 Bucket name. The credentials that you configured in the previous "Installing and configuring workflow-config helm chart" section should have read and write access to this bucket.

<br></br>
## Uninstall swoop-conductor

To uninstall the release, do `helm uninstall swoop-conductor`.
