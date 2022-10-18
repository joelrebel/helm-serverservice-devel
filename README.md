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
❯ docker build -f Dockerfile.dev .
```

2. Tag the built image
```
> docker tag <new image sha> localhost:5000/server-service:<semver>-<devel branch name>
```

3. Load into KIND registry
```
kind load docker-image
localhost:5000/server-service:<semver>-<devel branch name
```

4. In this repository - update values.yaml to point to new image
```
serverservice:
  image:
    repository: localhost:5000/server-service
    tag: <semver>-<devel branch name>
```

5. Helm upgrade
```
helm upgrade serverservice-devel . -f values.yaml
```

### Check out make help for a list of available commands.

```
$ make help

Usage:
  make <target>

Targets:
  local-devel          install helm chart for the server service local development environment
  local-devel-upgrade  upgrade helm chart for local devel environment
  port-forward-hss     port forward hollow server service port (runs in foreground)
  port-forward-crdb    port forward crdb service port (runs in foreground)
  psql-crdb            connect to crdb with psql (requires port-forward-crdb)
  kubectl-ctx-kind     set kube ctx to kind cluster
  help                 Show help
```
