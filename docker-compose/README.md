# Flexo MMS docker-compose

## What is this?
This docker-compose will start all services required for all current Flexo MMS microservices. It utilizes the following open source services in the backend:

- OpenLDAP - for Auth Service
- Apache Fuseki - Quadstore (GraphDB Option available)
- MinIO - for Store Service

With these initial services, the docker-compose will then start and connect the following Flexo MMS microservices:

- Flexo MMS Auth Service
- Flexo MMS Store Service
- Flexo MMS Layer 1 Service

All services will be on a bridged docker network named `flexo-mms-test-network`.

An initial trig file has been pre-generated under `mount/cluster.trig` and will be automatically added to Fuseki when it starts up. This includes policies that adds the default ldap users and group created to be admins.

For MinIO, a client can be installed to inspect objects https://min.io/docs/minio/linux/reference/minio-mc.html?ref=docs

## Default Flexo MMS Users and Groups
The following user / passwords are created by default:
- `user01` / `password1`
- `user02` / `password2`

## Usage
Install Docker Desktop from https://www.docker.com/

Run `docker-compose up` in this directory, once something like the following appears, the Flexo MMS api should be ready.

    layer1-service   | 2023-07-09T21:39:48,468Z [main] INFO  Application - Responding at http://0.0.0.0:8080

The default compose uses the Fuseki backend, a GraphDB example is also available, by doing `docker-compose -f docker-compose-graphdb.yml up` instead

### Setup GraphDB (if using GraphDB)

GraphDB requires more setup than Fuseki, but also offers more functionality and a UI at `http://localhost:7200`.

1. go to http://localhost:7200 to access the GraphDB UI
2. Under Setup > Repositories > Create New Repository > GraphDB Repository, use `openmbee` as Repository ID.
    1. the recommended settings are to use "No inference" from the Ruleset dropdown and check all of: "Enable content index", "Enable predicate list index", and "Enable full-text search (FTS) index".
    2. Leave all other settings as default and then hit `Create`
3. go to Import > Upload RDF Files > choose `mount/cluster.trig` file from this directory
    1. click Import > Import (can leave everything blank)
   
### Using Flexo MMS API
The first step will be the retrieve an authentication token from the auth-service, with `password1`. 

`curl -u user01 -X GET http://localhost:8082/login`

You can now use the token returned as a bearer token for all subsequent flexo-mms-layer1 api calls to http://localhost:8080, for api documentation, see https://www.openmbee.org/flexo-mms-layer1-openapi/

An example Postman collection is available in this directory that demonstrates basic api usage. Download the Postman app from https://www.postman.com/ and import the collection file to use. (for load model, the gnis-hawaii.ttl file in this directory can be used)

## Connecting Jupyter Notebook Quick Start

    docker run -p 8888:8888 --network=flexo-mms-test-network jupyter/scipy-notebook:latest

Example of using python `requests` lib to call the login url (note the host should be the host defined in the docker-compose file)

```python
import requests
from requests.auth import HTTPBasicAuth
response = requests.get("http://auth-service:8080/login", auth=HTTPBasicAuth('user01', 'password1'))
print(response.json())
```

## Potential Errors

You may see an error related to the `store-service` such that it doesn't start up (we believe this is due to issues with docker on m1/m2 mac). If this happens, the `store-service` is optional and can be taken out entirely. To remove it, remove the following lines and restart the compose:

- `FLEXO_MMS_STORE_SERVICE_URL=http://store-service:8080/store` in env/flexo-mms-layer1.env (env/flexo-mms-layer1-graphdb.env if using GraphDB)

- `store-service` under `depends_on:` in the docker-compose.yml file for layer1-service

## Shutdown
`Ctrl-C` from the terminal and run `docker-compose down` once all containers are shut down. (`docker-compose -f docker-compose-graphdb.yml down` for GraphDB)
