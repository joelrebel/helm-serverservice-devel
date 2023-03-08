## k8s helm charts for a server service development instance.

Note: this deploys an *insecure* cockroachdb, serverservice instance in the
`default` k8s namespace to enable local [serverservice](https://github.com/metal-toolbox/hollow-serverservice/) development.


### Prerequisites

- Install docker KIND
- Setup a local KIND cluster with a registry using the script here: https://kind.sigs.k8s.io/docs/user/local-registry/
- export `KUBECONFIG=~/.kube/config_kind`

### Deploy helm chart.

- Run `make k8s-local-devel`


### Run a devel serverservice docker image

1. In your fork of the serverservice code repository - build the dev image with your changes.

```
export GIT_TAG="localhost:5000/serverservice:latest" && \
    GOOS=linux GOARCH=amd64 go build -o serverservice && \
    docker build -t "${GIT_TAG}" -f Dockerfile . && \
    docker push localhost:5000/serverservice:latest && kind load docker-image "${GIT_TAG}"
```


5. Helm upgrade
```
make local-devel-upgrade
```

## NATs setup

The chart configures a NATS Jetstream that serverservice sends messages on,
for the list of accounts configured check out [values.yaml](values.yaml).

Also check out the [cheatsheet](cheatsheet.md) to validate the Jetstream setup.


## Chaos mesh

The utility exposes a cool dashboard to run chaos experiments like dropping packets
from one app to another or such.


Install chaos mesh
```
helm install chaos-mesh  chaos-mesh/chaos-mesh -n=default --version 2.5.1 -f values.yaml
```


forward the dashboard port and run an experiment ! `http://localhost:2333/experiments`
```
make port-forward-chaos-dash
```

Uninstall chaos mesh
```
helm delete  chaos-mesh -n=default
```

### Check out make help for a list of available commands.

```
❯ make

Usage:
  make <target>

Targets:
  local-devel          install helm chart for the server service local development environment
  local-devel-upgrade  upgrade helm chart for local devel environment
  port-forward-hss     port forward hollow server service port (runs in foreground)
  port-forward-crdb    port forward crdb service port (runs in foreground)
  port-forward-chaos-dash port forward chaos-mesh dashboard (runs in foreground)
  psql-crdb            connect to crdb with psql (requires port-forward-crdb)
  kubectl-ctx-kind     set kube ctx to kind cluster
  help                 Show help
```
