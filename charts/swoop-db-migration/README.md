## Database Migrations on K8s

Each individual helm chart is contained within a separate folder in the `charts` directory. Key helm charts for the database migrations are:

* `postgres`: this will deploy the `swoop-db` image from quay.io, which brings up a Postgres database server which is used to host the state database of SWOOP
* `swoop-bundle`: this is a 'bundled' helm chart that contains the three SWOOP components (API, Caboose, and Conductor) and the `swoop-db-migration` job as helm dependencies. Each of the three SWOOP components are their own helm chart as well.
* `swoop-db-init`: a helm chart for initializing Swoop-DB, this will run the database initialization script inside the Swoop-DB container
* `swoop-db-migration`: a helm chart for performing migrations in Swoop-DB, this will run the migration script inside a Swoop-DB container that has already been initialized by the database initialization script

The `swoop-db-init` chart will initialize a new database named `swoop` inside the database server with appropriate roles for database owner, read/write, and for the SWOOP components to interact with the database. The `swoop-db-migration` chart will then apply the migrations inside the initialized `swoop` database.

### Deployment

First, make sure that the `e84` repo has been added:

`helm repo add e84 https://element84.github.io/filmdrop-k8s-helm-charts`

Update the contents of the helm repo to get the latest versions of all charts:

`helm repo update e84`

You can list the contents of the repo (a list of all of its charts) by running:

`helm search repo e84`

Then, install the required helm charts, in this order:

`helm install swoopdb e84/postgres`

`helm install dbinit e84/swoop-db-init`

`helm install migration e84/swoop-db-migration`

This will create three helm releases named `swoopdb`, `dbinit`, and `migration`, respectively, which can be verified using `helm list`.

Note: the order in which these are installed is important. The `swoop-db-init` chart needs to have a running database server first, which is installed using the `postgres` chart, and the `swoop-db-migration` chart needs to have an initialized database first, which is installed by the `swoop-db-init` chart.

You can then do `kubectl get all` to view a list of all resources that were deployed onto the cluster with the above commands. The logs of the

You can then port-forward the `postgres` service onto a localhost port through Rancher Desktop or kubectl to view the migrated database in pgAdmin.

### Customization of deployment

The configuration of the K8s resources deployed by these helm charts can be changed by modifiying each chart's `values.yaml` file, which populates the values in the chart's templates.

The `values.yaml` files are located in the root folder of each chart, for example at `swoop-db-migration/values.yaml`, `swoop-db-init/values.yaml`, and `postgres/values.yaml`.

#### Values for migration helm chart

Three key values that need to be set in order to run the migration job are:

* `postgres.migration.rollback`: a boolean value indicating whether or not to rollback the database; if `true`, the database is rolled back; if `false`, the database is migrated
* `postgres.migration.version`: the migration version to which to migrate/rollback the database
* `postgres.migration.no_wait`: override option to skip waiting for active connections from application roles to close

## Uninstall swoop-db-migration

To uninstall the release, do `helm uninstall swoop-db-migration`.
