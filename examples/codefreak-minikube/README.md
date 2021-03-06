# Code FREAK on Minikube

This is a sample configuration how to run Code FREAK locally in Minikube for testing purposes.

## Prerequisites

1. Install minikube by following [the official instruction](https://minikube.sigs.k8s.io/docs/start/) 
2. Start the minikube cluster with `minikube start` (Warning: If you are using Mac please don't use the Docker driver as it's not compatible with the Ingress addon as of Nov. 2021)
3. Enable the ingress addon with `minikube addons enable ingress`
4. Verify that `kubectl cluster-info` shows information about you minikube cluster
5. Install Helm v3 (https://helm.sh/docs/intro/install/)

We will be using Ingress to expose Code FREAK and its workspaces.
Please set a static entry in your /etc/hosts file pointing to the IP returned by `minikube ip`
for the hosts "codefreak" and "codefreak-workspaces".

Example:
```
192.168.49.2    codefreak codefreak-workspaces
```

Please replace `192.168.49.2` with the IP returned by `minikube ip`.
Check that opening `http://codefreak` and `http://codefreak-workspaces` in your browser shows a `404`. If not, please make sure the Ingress controller is running (`kubectl -n ingress-nginx get pod,svc`) and you set the appropriate entries in `/etc/hosts`


## Install Code FREAK on minikube

Please create a `cf-minikube-values.yml` file with the following contents and adjust the value of `hostAliases[0].ip`.

```yml
ingress:
  enabled: true
  hosts:
    - host: codefreak
      paths:
        - path: /
          pathType: Prefix

hostAliases:
  # Please set this IP to the value of `minikube ip`
  - ip: "192.168.49.2"
    hostnames:
    - "codefreak-workspaces"

workspaces:
  baseUrlTemplate: http://codefreak-workspaces/{workspaceIdentifier}
  createNamespace: true
  namespace: my-workspace-namespace

```

Afterwards you can start the app using the following commands:

```
helm repo add codefreak https://codefreak.github.io/charts
helm repo update codefreak
helm install -f cf-minikube-values.yml my-codefreak-install codefreak/codefreak
```

You can check the progress using `kubectl -n default get pods --selector app.kubernetes.io/instance=my-codefreak-install`.
If the application is `Ready` the UI should be reachable at `http://codefreak` in your browser.