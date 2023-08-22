# Deployment

This chart deploys the SWOOP configuration file and workflow templates onto a K8s cluster.

The SWOOP configuration file contains the high-level configuration parameters and values used by SWOOP components such as workflows, callbacks, conductors, and handlers, to run workflows on the cluster.

The workflow templates contain more detailed configurations used by each individual workflow to run, and describe their tasks, input and output artifacts, any additional workflow templates referenced by the main workflow (if any), and secrets being used.

The workflow-config helm chart uses Argo Workflows since the WorkflowTemplate resources are Argo-specific configurations. Therefore, install the `swoop-caboose` helm chart first, which will also install Argo Workflows as a dependency.

To add the FilmDrop Helm Chart repository, do:

`helm repo add e84 https://element84.github.io/filmdrop-k8s-helm-charts`

First, follow the stepwise instructions under [Deployment](https://github.com/Element84/filmdrop-k8s-helm-charts/blob/main/charts/swoop-caboose/README.md#deployment), **until and including** the following command:

```
mc cp ./payload_workflow.json swoopminio/payloadtest/
```

Do not create any further resources such as secrets or submit any workflows.

Then, modify the `values.yaml` file with the `AWS Access Key ID`, `AWS Secret Access Key`, and `AWS Region` under the keys `s3.accessKeyId`, `s3.secretAccessKey`, `s3.region`, respectively. These values contain dummy data and should be modified to include your AWS IAM user credentials. Make sure you are using base-64 encoding for these values. Also, make sure that the correct values are provided for `minio.host`, `minio.port`, and `minio.bucket` as well. If you're running this example from a minio created by the MinIO helm chart, you should be able to use `minio.default` for the host and `9000` for the port.

Then, do:

`helm install workflow-config e84/workflow-config`

Once the chart has been deployed, you can do:

`kubectl get workflowtemplate` and

`kubectl get configmap`

to see that the workflow templates and SWOOP configmap were deployed.

This will deploy the Swoop configuration file as a configmap and the copy-assets workflow as an Argo Workflow onto the cluster. The workflow will run to completion and you should then see the output thumbnail asset in the bucket that you created in AWS S3.
