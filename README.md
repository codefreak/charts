# Code FREAK Helm Chart
![GitHub release (latest by date including pre-releases)](https://img.shields.io/github/v/release/codefreak/charts?include_prereleases&label=Chart&logo=helm)

Official Helm chart for installing Code FREAK on a Kubernetes cluster.

## Quickstart

```
helm repo add codefreak https://codefreak.github.io/charts
helm install my-codefreak-install codefreak/codefreak
```

## Configuration values
Please check out the [`values.yaml`](./charts/codefreak/values.yaml) for a list of supported values and their purposes.

## Prerequisites
* Helm
* Kubernetes Cluster (version >=1.19 recommended for stable Ingress support)
* Database: Postgres recommended. By default, an in-memory database will be used. Data will get lost between restarts!

## Resource requirements
The resource requirements will depend on the number of concurrent users you expect. For a production environment you should expect 1-2GiB RAM per concurrent user.

## Creating the JWT signing keypair
Code FREAK will use signed JWTs for authenticating users.
For this to work you have to create an RSA keypair once and store it in a secret that will be accessible from the backend
application.

```shell
# Create a key pair and convert the private key to the PKCS8
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
openssl pkcs8 -topk8 -nocrypt -in private.pem -out private-pkcs8.pem
# Create a secret containing both values
kubectl create secret generic my-secret-name --from-file=private=private-pkcs8.pem --from-file=public=public.pem
```

Afterwards, set the following values in your `values.yml`:

```yaml
workspaces:
  jwtKeypairSecretName: my-secret-name
  jwtKeypairSecretPrivateKey: private # default, can be omitted
  jwtKeypairSecretPublicKey: public # default, can be omitted
```

## Workspace namespace
Code FREAK will start additional Pods on demand, called "workspaces".
Workspaces are used to provide in-browser IDEs for students or evaluating their code.
For security reasons we recommend starting workspace Pods in a dedicated namespace.
To create a dedicated namespace you can set the following value in your `values.yml`:

```yaml
workspaces:
  namespace: my-workspace-namespace
```

If your user does not have access to create namespaces please set `workspaces.createNamespace` to `false` and ask
your cluster admin to create a dedicated namespace for you:

```yaml
workspaces:
  # Must be created manually
  namespace: my-workspace-namespace
  createNamespace: false
```
