# FilmDrop Kubernetes Helm Charts

This repository contains the packaging of FilmDrop helm charts with Kubernetes.

This FilmDrop repository manages the supporting infrastructure required to deploy the [STAC Workflow Open Orchestration Platform (SWOOP)](https://github.com/Element84/swoop). The helm charts in this repository are utilized via the [FilmDrop Kubernetes Terraform Infrastructure Modules](https://github.com/Element84/filmdrop-k8s-tf-modules) to deploy the FilmDrop resources on Kubernetes clusters.

## Components

### Local Path Provisioner

[Local Path Provisioner](https://github.com/rancher/local-path-provisioner) provides a way for the Kubernetes users to utilize the local storage in each node. Based on the user configuration, the Local Path Provisioner will create
`hostPath` based persistent volume on the node automatically. More information is available [here](./charts/local-path-provisioner/README.md).
<br></br>

### MinIO

[MinIO](https://min.io/) is an S3 compatible object storage for the usage of the [SWOOP API](https://github.com/Element84/swoop), [SWOOP Caboose](https://github.com/Element84/swoop-go), and [SWOOP Conductor](https://github.com/Element84/swoop-go). More information is available [here](./charts/minio/README.md).
<br></br>

### Postgres

[PostgreSQL](https://www.postgresql.org/) is an open source object-relational database system for the usage of the [SWOOP API](https://github.com/Element84/swoop), [SWOOP Caboose](https://github.com/Element84/swoop-go), and [SWOOP Conductor](https://github.com/Element84/swoop-go). More information is available [here](./charts/postgres/README.md).
<br></br>

### STAC-FastAPI

[Stac-FastAPI](https://github.com/stac-utils/stac-fastapi) is a Python library for building a STAC compliant FastAPI application. It consists of a front-end FastAPI application that exposes a STAC catalog along with its Collections and Items, and a backend PostgreSQL database built using the PgSTAC framework (more information for which can be found [here](https://github.com/stac-utils/pgstac)). The backend database stores provides functionality for STAC Filters and CQL2 search along with utilities to help manage indexing and partitioning of STAC Collections and Items. The Helm chart deploys the two components onto two separate pods that are each exposed using separate Kubernetes services. More information is available [here](./charts/stac-fastapi/README.md).
<br></br>

### SWOOP API

SWOOP is the [STAC Workflow Open Orchestration Platform](https://github.com/Element84/swoop). It consists of a front-end FastAPI application that  acts as an interface for clients to interact with the processing pipeline and state database. The helm chart for the swoop api will needa Postgres database and a Minio object storage that need to be installed separately. These Postgres database is intended to be used as the SWOOP state database, and Minio as the object storage for payload inputs and job result outputs. More information is available [here](./charts/swoop-api/README.md).
<br></br>

### SWOOP Bundle

SWOOP Bundle will install the followin the following components into a kubernetes cluster:
* [swoop-api](https://github.com/Element84/swoop)
* [argo-workflows](https://github.com/argoproj/argo-workflows/)
* [swoop-caboose](https://github.com/Element84/swoop-go)
* [swoop-conductor](https://github.com/Element84/swoop-go)
This helm chart is inteded to group all SWOOP components compatible with the same database schema version and it's related migration operations. More information is available [here](./charts/swoop-bundle/README.md).
<br></br>


### SWOOP Bundle

SWOOP Bundle will install the followin the following components into a kubernetes cluster:
* [swoop-api](https://github.com/Element84/swoop)
* [argo-workflows](https://github.com/argoproj/argo-workflows/)
* [swoop-caboose](https://github.com/Element84/swoop-go)
* [swoop-conductor](https://github.com/Element84/swoop-go)
This helm chart is inteded to group all SWOOP components compatible with the same database schema version and it's related migration operations. More information is available [here](./charts/swoop-bundle/README.md).
<br></br>

### SWOOP Caboose

[SWOOP Caboose](https://github.com/Element84/swoop-go) is the SWOOP component that handles the end of a workflow. SWOOP Caboose also installs [Argo Workflows](https://github.com/argoproj/argo-workflows/) as a dependency. More information is available [here](./charts/swoop-caboose/README.md).
<br></br>

### SWOOP Conductor

[SWOOP Conductor](https://github.com/Element84/swoop-go) is the SWOOP component that acts as an async executor application. More information is available [here](./charts/swoop-conductor/README.md).
<br></br>

### SWOOP DB Initialization

The swoop-db-init helm chart will initialize [swoop-db](https://github.com/Element84/swoop-db) with the creation of a SWOOP database and Roles for each SWOOP Component. More information is available [here](./charts/swoop-db-init/README.md).
<br></br>

### SWOOP DB Migration

The swoop-db-migration helm chart will migrate forward or backwards a schema version of the [swoop-db](https://github.com/Element84/swoop-db) in support of the SWOOP Components. More information is available [here](./charts/swoop-db-migration/README.md).
<br></br>
