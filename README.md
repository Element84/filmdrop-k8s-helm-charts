# filmdrop-k8s-helm-charts

## STAC-FastAPI

Stac-FastAPI is a Python library for building a STAC compliant FastAPI application. More information can be found at [https://github.com/stac-utils/stac-fastapi](https://github.com/stac-utils/stac-fastapi). It consists of a front-end FastAPI application that exposes a STAC catalog along with its Collections and Items, and a backend PostgreSQL database built using the PgSTAC framework (more information for which can be found [here](https://github.com/stac-utils/pgstac)). The backend database stores provides functionality for STAC Filters and CQL2 search along with utilities to help manage indexing and partitioning of STAC Collections and Items. The Helm chart deploys the two components onto two separate pods that are each exposed using separate Kubernetes services.

## SWOOP API

SWOOP is the STAC Workflow Open Orchestration Platform. More information can be found at [https://github.com/Element84/swoop](https://github.com/Element84/swoop). It consists of a front-end FastAPI application that  acts as an interface for clients to interact with the processing pipeline and state database. The helm chart for the swoop api contains a Postgres database and a Minio object storage. These Postgres database is intended to be used as the SWOOP state database, and Minio as the object storage for payload inputs and job result outputs. The Helm chart deploys the three components onto three separate pods that are each exposed using separate Kubernetes services.
