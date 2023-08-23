# Deployment

This chart deploys the SWOOP configuration file and workflow templates onto a K8s cluster. It will only deploy these resources, and will not run any workflows.

The SWOOP configuration file contains the high-level configuration parameters and values used by SWOOP components such as workflows, callbacks, conductors, and handlers, to run workflows on the cluster. The configuration file is located at `./swoop-config.yaml`.

The workflow templates contain more detailed configurations used by each individual workflow to run, and describe their tasks, input and output artifacts, any additional workflow templates referenced by the main workflow (if any), and secrets being used. The workflow templates are contained within the `./workflows/` directory.

The workflow-config helm chart uses Argo Workflows since the WorkflowTemplate resources are Argo-specific configurations.

To add the FilmDrop Helm Chart repository, do:

`helm repo add e84 https://element84.github.io/filmdrop-k8s-helm-charts`

Install the below helm charts **in order**:

`helm install minio e84/minio`

`helm install postgres e84/postgres`

`helm install swoop-db-init e84/swoop-db-init`

`helm install swoop-db-migration e84/swoop-db-migration`

`helm install swoop-caboose e84/swoop-caboose`

The `swoop-caboose` helm chart will also install Argo Workflows as a dependency.

Then, modify the `values.yaml` file with the `AWS Access Key ID`, `AWS Secret Access Key`, and `AWS Region` and/or `AWS Session Token` under the keys `s3.accessKeyId`, `s3.secretAccessKey`, `s3.region`, `s3.sessionToken`, respectively. These values contain dummy data and should be modified to include your AWS credentials. Also, update the `bucket` value with the name of the bucket that contains the `input.json` file and will also eventually contain the `output.json` file once the workflow has completed.  Make sure you are using base-64 encoding for these values.

Then, do:

`helm install workflow-config e84/workflow-config`

This will deploy the Swoop configuration file as a configmap and the copy-assets workflow template as an Argo WorkflowTemplate onto the cluster.

Once the chart has been deployed, you can do:

`kubectl get workflowtemplate` and

`kubectl get configmap`

to see that the workflow templates and SWOOP configmap were deployed.
