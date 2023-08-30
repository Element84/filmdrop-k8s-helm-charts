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

For waiting for the MinIO pods to be ready:
```
kubectl wait --for=condition=ready --timeout=30m pod -l app=minio
```

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

Modify the `values.yaml` file for the `workflow-config` helm chart:

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

You should see in the response a status of "accepted"
```
$ python3 swoop_api_payload_test.py

Enter SWOOP API Host (localhost:8000): localhost:8000
Enter STAC ASSETS BUCKET NAME: REDACTED_BUCKET_NAME
***** INPUTS SUMMARY *****
SWOOP API Host: localhost:8000
STAC ASSETS BUCKET NAME: REDACTED_BUCKET_NAME
POST URL: http://localhost:8000/processes/mirror/execution
POST Input payload:
{"inputs": {"payload": {"type": "FeatureCollection", "id": "tx_m_2609719_se_14_060_20201217", "features": [{"id": "tx_m_2609719_se_14_060_20201217", "bbox": [-97.690252, 26.622563, -97.622203, 26.689923], "type": "Feature", "links": [{"rel": "collection", "type": "application/json", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/collections/naip"}, {"rel": "parent", "type": "application/json", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/collections/naip"}, {"rel": "root", "type": "application/json", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/"}, {"rel": "self", "type": "application/geo+json", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/collections/naip/items/tx_m_2609719_se_14_060_20201217"}, {"rel": "preview", "href": "https://planetarycomputer.microsoft.com/api/data/v1/item/map?collection=naip&item=tx_m_2609719_se_14_060_20201217", "title": "Map of item", "type": "text/html"}], "assets": {"image": {"href": "https://naipeuwest.blob.core.windows.net/naip/v002/tx/2020/tx_060cm_2020/26097/m_2609719_se_14_060_20201217.tif", "type": "image/tiff; application=geotiff; profile=cloud-optimized", "roles": ["data"], "title": "RGBIR COG tile", "eo:bands": [{"name": "Red", "common_name": "red"}, {"name": "Green", "common_name": "green"}, {"name": "Blue", "common_name": "blue"}, {"name": "NIR", "common_name": "nir", "description": "near-infrared"}]}, "thumbnail": {"href": "https://naipeuwest.blob.core.windows.net/naip/v002/tx/2020/tx_060cm_2020/26097/m_2609719_se_14_060_20201217.200.jpg", "type": "image/jpeg", "roles": ["thumbnail"], "title": "Thumbnail"}}, "geometry": {"type": "Polygon", "coordinates": [[[-97.623004, 26.622563], [-97.622203, 26.689286], [-97.68949, 26.689923], [-97.690252, 26.623198], [-97.623004, 26.622563]]]}, "collection": "naip", "properties": {"version": 2, "gsd": 0.6, "datetime": "2020-12-17T00:00:00Z", "naip:year": "2020", "proj:bbox": [630384, 2945370, 637080, 2952762], "proj:epsg": 26914, "naip:state": "tx", "proj:shape": [12320, 11160], "proj:transform": [0.6, 0, 630384, 0, -0.6, 2952762, 0, 0, 1]}, "stac_extensions": ["https://stac-extensions.github.io/eo/v1.0.0/schema.json", "https://stac-extensions.github.io/projection/v1.0.0/schema.json"], "stac_version": "1.0.0"}], "process": [{"description": "string", "workflow": "mirror", "upload_options": {"path_template": "s3://REDACTED_BUCKET_NAME/data/${collection}/${id}/", "collections": {"naip": "*"}, "public_assets": [], "headers": {}, "s3_urls": false}, "tasks": {"copy-assets": {"assets": ["thumbnail"], "drop_assets": ["image"]}}}]}}, "response": "document"}
**************************
**** RESPONSE SUMMARY ****
Response from POST to http://localhost:8000/processes/mirror/execution:
 {"processID":"mirror","type":"process","jobID":"018a424a-9201-75c7-9455-e7326c776904","status":"accepted","message":null,"created":"2023-08-29T17:14:57.921969+00:00","started":null,"finished":null,"updated":"2023-08-29T17:14:57.921969+00:00","links":[{"href":"http://localhost:8000/","rel":"root","type":"application/json"},{"href":"http://localhost:8000/jobs/018a424a-9201-75c7-9455-e7326c776904","rel":"self","type":"application/json"},{"href":"http://localhost:8000/jobs/018a424a-9201-75c7-9455-e7326c776904/results","rel":"results","type":"application/json"},{"href":"http://localhost:8000/jobs/018a424a-9201-75c7-9455-e7326c776904/inputs","rel":"inputs","type":"application/json"},{"href":"http://localhost:8000/processes/mirror","rel":"process","type":"application/json"},{"href":"http://localhost:8000/payloadCacheEntries/f7ceceb1-5871-54c9-837d-d24380cb252b","rel":"cache","type":"application/json"}]}
**************************
```

You should see now a `018a424a-9201-75c7-9455-e7326c776904-copy-task-someid` pod matching the jobID `018a424a-9201-75c7-9455-e7326c776904` running via:
```
$ kubectl get pods

NAME                                                              READY   STATUS      RESTARTS   AGE
minio-78fc4699df-k9jsp                                            1/1     Running     0          7m33s
postgres-7b6b9ff76-hj4qs                                          1/1     Running     0          6m55s
db-initialization-8-cdqz9                                         0/1     Completed   0          6m22s
wait-for-db-initialization-8-2mdhf                                0/1     Completed   0          6m22s
migration-job-8-q442b                                             0/1     Completed   0          5m42s
swoop-caboose-argo-workflows-workflow-controller-86d7778c8fgfcl   1/1     Running     0          5m24s
swoop-caboose-78cdd68b7b-8dr7n                                    1/1     Running     0          5m24s
swoop-api-cdf497fc8-wmjgw                                         1/1     Running     0          4m24s
swoop-conductor-55fcb7d956-l6q22                                  1/1     Running     0          3m12s
018a424a-9201-75c7-9455-e7326c776904-copy-task-199703505          2/2     Running     0          51s
```

In a couple of minutes, the job will complete, and you will not see the pod running anymore with `kubectl get pods`.

Now if you run the same swoop-api test, with the same parameters, you will see the results from the cache with a status of `successful`:
```
$ python3 swoop_api_payload_test.py

Enter SWOOP API Host (localhost:8000): localhost:8000
Enter STAC ASSETS BUCKET NAME: REDACTED_BUCKET_NAME
***** INPUTS SUMMARY *****
SWOOP API Host: localhost:8000
STAC ASSETS BUCKET NAME: REDACTED_BUCKET_NAME
POST URL: http://localhost:8000/processes/mirror/execution
POST Input payload:
{"inputs": {"payload": {"type": "FeatureCollection", "id": "tx_m_2609719_se_14_060_20201217", "features": [{"id": "tx_m_2609719_se_14_060_20201217", "bbox": [-97.690252, 26.622563, -97.622203, 26.689923], "type": "Feature", "links": [{"rel": "collection", "type": "application/json", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/collections/naip"}, {"rel": "parent", "type": "application/json", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/collections/naip"}, {"rel": "root", "type": "application/json", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/"}, {"rel": "self", "type": "application/geo+json", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/collections/naip/items/tx_m_2609719_se_14_060_20201217"}, {"rel": "preview", "href": "https://planetarycomputer.microsoft.com/api/data/v1/item/map?collection=naip&item=tx_m_2609719_se_14_060_20201217", "title": "Map of item", "type": "text/html"}], "assets": {"image": {"href": "https://naipeuwest.blob.core.windows.net/naip/v002/tx/2020/tx_060cm_2020/26097/m_2609719_se_14_060_20201217.tif", "type": "image/tiff; application=geotiff; profile=cloud-optimized", "roles": ["data"], "title": "RGBIR COG tile", "eo:bands": [{"name": "Red", "common_name": "red"}, {"name": "Green", "common_name": "green"}, {"name": "Blue", "common_name": "blue"}, {"name": "NIR", "common_name": "nir", "description": "near-infrared"}]}, "thumbnail": {"href": "https://naipeuwest.blob.core.windows.net/naip/v002/tx/2020/tx_060cm_2020/26097/m_2609719_se_14_060_20201217.200.jpg", "type": "image/jpeg", "roles": ["thumbnail"], "title": "Thumbnail"}}, "geometry": {"type": "Polygon", "coordinates": [[[-97.623004, 26.622563], [-97.622203, 26.689286], [-97.68949, 26.689923], [-97.690252, 26.623198], [-97.623004, 26.622563]]]}, "collection": "naip", "properties": {"version": 2, "gsd": 0.6, "datetime": "2020-12-17T00:00:00Z", "naip:year": "2020", "proj:bbox": [630384, 2945370, 637080, 2952762], "proj:epsg": 26914, "naip:state": "tx", "proj:shape": [12320, 11160], "proj:transform": [0.6, 0, 630384, 0, -0.6, 2952762, 0, 0, 1]}, "stac_extensions": ["https://stac-extensions.github.io/eo/v1.0.0/schema.json", "https://stac-extensions.github.io/projection/v1.0.0/schema.json"], "stac_version": "1.0.0"}], "process": [{"description": "string", "workflow": "mirror", "upload_options": {"path_template": "s3://REDACTED_BUCKET_NAME/data/${collection}/${id}/", "collections": {"naip": "*"}, "public_assets": [], "headers": {}, "s3_urls": false}, "tasks": {"copy-assets": {"assets": ["thumbnail"], "drop_assets": ["image"]}}}]}}, "response": "document"}
**************************
**** RESPONSE SUMMARY ****
Response from POST to http://localhost:8000/processes/mirror/execution:
 {"processID":"mirror","type":"process","jobID":"018a424a-9201-75c7-9455-e7326c776904","status":"successful","created":"2023-08-29T17:14:57.921969+00:00","started":"2023-08-29T17:14:58+00:00","finished":"2023-08-29T17:16:00+00:00","updated":"2023-08-29T17:16:00+00:00","links":[{"href":"http://localhost:8000/","rel":"root","type":"application/json"},{"href":"http://localhost:8000/jobs/018a424a-9201-75c7-9455-e7326c776904","rel":"self","type":"application/json"},{"href":"http://localhost:8000/jobs/018a424a-9201-75c7-9455-e7326c776904/results","rel":"results","type":"application/json"},{"href":"http://localhost:8000/jobs/018a424a-9201-75c7-9455-e7326c776904/inputs","rel":"inputs","type":"application/json"},{"href":"http://localhost:8000/processes/mirror","rel":"process","type":"application/json"},{"href":"http://localhost:8000/payloadCacheEntries/f7ceceb1-5871-54c9-837d-d24380cb252b","rel":"cache","type":"application/json"}]}
**************************
```

Lastly, you should be able to see the output on MinIO.

First port-forward MinIO port 9000 via:
```
kubectl port-forward -n default svc/minio 9000:9000 &
```

Then connect to the MinIO client via:
```
export MINIO_ACCESS_KEY=`helm get values minio -a -o json | jq -r .service.accessKeyId | base64 --decode`
export MINIO_SECRET_KEY=`helm get values minio -a -o json | jq -r .service.secretAccessKey | base64 --decode`
mc alias set swoopminio http://127.0.0.1:9000 $MINIO_ACCESS_KEY $MINIO_SECRET_KEY
```

Check that you have an output file:
```
$ mc ls swoopminio/swoop/executions/018a424a-9201-75c7-9455-e7326c776904/

[2023-08-29 13:14:57 EDT] 2.5KiB STANDARD input.json
[2023-08-29 13:15:50 EDT] 2.1KiB STANDARD output.json
[2023-08-29 13:16:00 EDT] 5.0KiB STANDARD workflow.json
```

Copy the output file locally:
```
$ mc cp swoopminio/swoop/executions/018a424a-9201-75c7-9455-e7326c776904/output.json .

...7326c776904/output.json: 2.05 KiB / 2.05 KiB ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 49.29 KiB/s 0s
```

Check the contents of the output.json:
```
$ cat output.json

{"type": "FeatureCollection", "features": [{"type": "Feature", "stac_version": "1.0.0", "id": "tx_m_2609719_se_14_060_20201217", "properties": {"version": 2, "gsd": 0.6, "datetime": "2020-12-17T00:00:00Z", "naip:year": "2020", "proj:bbox": [630384, 2945370, 637080, 2952762], "proj:epsg": 26914, "naip:state": "tx", "proj:shape": [12320, 11160], "proj:transform": [0.6, 0, 630384, 0, -0.6, 2952762, 0, 0, 1], "processing:software": {"copy-assets": "0.1.0"}}, "geometry": {"type": "Polygon", "coordinates": [[[-97.623004, 26.622563], [-97.622203, 26.689286], [-97.68949, 26.689923], [-97.690252, 26.623198], [-97.623004, 26.622563]]]}, "links": [{"rel": "collection", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/collections/naip", "type": "application/json"}, {"rel": "parent", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/collections/naip", "type": "application/json"}, {"rel": "self", "href": "https://planetarycomputer.microsoft.com/api/stac/v1/collections/naip/items/tx_m_2609719_se_14_060_20201217", "type": "application/geo+json"}, {"rel": "preview", "href": "https://planetarycomputer.microsoft.com/api/data/v1/item/map?collection=naip&item=tx_m_2609719_se_14_060_20201217", "type": "text/html", "title": "Map of item"}], "assets": {"thumbnail": {"href": "https://REDACTED_BUCKET_NAME.s3.us-west-2.amazonaws.com/data/naip/tx_m_2609719_se_14_060_20201217/thumbnail.jpg", "type": "image/jpeg", "title": "Thumbnail", "roles": ["thumbnail"]}}, "bbox": [-97.690252, 26.622563, -97.622203, 26.689923], "stac_extensions": ["https://stac-extensions.github.io/processing/v1.1.0/schema.json", "https://stac-extensions.github.io/eo/v1.0.0/schema.json", "https://stac-extensions.github.io/projection/v1.0.0/schema.json"], "collection": "naip"}], "process": {"description": "string", "tasks": {"copy-assets": {"assets": ["thumbnail"], "drop_assets": ["image"]}}, "upload_options": {"path_template": "s3://REDACTED_BUCKET_NAME/data/${collection}/${id}/", "collections": {"naip": "*"}, "public_assets": [], "headers": {}, "s3_urls": false}, "workflow": "mirror"}}%
```

<br></br>
## Uninstall swoop-conductor

To uninstall the release, do `helm uninstall swoop-conductor`.
