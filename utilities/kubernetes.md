# Contents

- [basics](#basics)
    - [Ingress](#basics#Ingress)
- [frontend](#frontend)
- [deployment](#deployment)
    - [edit deployment](#deployment#edit deployment)
    - [restart deployment](#deployment#restart deployment)
    - [delete deployment](#deployment#delete deployment)
- [secrets](#secrets)
    - [edit secrets](#secrets#edit secrets)
    - [secret from file](#secrets#secret from file)
- [examples](#examples)
    - [gpu pod](#examples#gpu pod)
    - [postgres pod](#examples#postgres pod)
- [k3s](#k3s)
- [delete everything from namespace](#delete everything from namespace)
- [delete node](#delete node)
- [delete stuck namespace](#delete stuck namespace)

# basics
Kubernetes `Services` that are a set of labeled `Pods` are running on `Nodes`.  
They use internal network for communication and can be exposed to Internet with special  
services with type `NodePort` or `LoadBalancer`. For **HTTP/HTTPS** `Ingress` (w/ IngressController)  
is used.

## Ingress
Is an **API resource** used as _load balancer_, _SSL terminator_, _virtual hosting_.  
Besides **resource** an Ingress Controller is required, e.g. `ingress-nginx`.


# frontend
```bash
  kubectl proxy
```
connect to `http://localhost:8001/api/v1/namespaces/kubernator/services/kubernator/proxy/#/catalog`

# deployment

## edit deployment
`kubectl -n <namespace> edit deployment`  
replace  
```bash
   - command:
       - bash
```
with  
`-command: ['sleep', '500m']`  
to be able to connect to pod (instead it will fall into recreating loop)  

## restart deployment
* `kubectl -n <namespace> get deployment` => `<deployment_name>`
* `kubectl -n <namespace> rollout restart deployment <deployment-name>`

## delete deployment
To prevent pod from recreating the deployment must be deleted  
`kubectl -n <namespace> delete deploy <deploy_name>`

# secrets

## edit secrets
* `kubectl -n <namespace> get secrets`
* `kubectl -n <namespace> edit secret <secret>`
* paste secret in base64: `echo 'string' | base64`

## secret from file
* `kubectl -n <namespace> create secret generic <SECRET NAME> --from-file=<file>`
* add to deployment:
    ```yaml
    specs:
      containers:
          volumeMounts:
          - mountPath: /var/run/secrets/google
            name: <MOUNT NAME>
            readOnly: true
      volumes:
      - name: <MOUNT NAME>
        secret:
          defaultMode: 420
          secretName: <SECRET NAME>
    ```
**NOTE**: for some reason, ENV for the secret file must be defined in `ConfigMap`, not in `Secrets`

# examples

## gpu pod
```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: gpu-test
    namespace: experiments
  spec:
    containers:
    - command: 
      - sleep 
      - infinity
      name: my-gpu-container
      resources:
        limits:
         nvidia.com/gpu: 1
      image: nvidia/cuda:9.0-devel
      env:
      - name: NVIDIA_DRIVER_CAPABILITIES
        value: "all"
```
example: [GPU node performance](../hardware/gpu.md#GPU-node-performance-measurment)

## postgres pod
* create pod  
    `kubectl run -n <namespace> --image=postgres:9.2 tmp-psql`
* connect to database (from the pod)  
  `psql -h <database> -U postgres`
  
  
# k3s

* config file with auth (i.e. `kubeconfig`) is at `/etc/rancher/k3s/k3s.yaml`
  can be copied with: `sudo k3s kubectl config view --raw | tee <local config>`
* node token can be obtained from `/var/lib/rancher/k3s/server/node-token`


# delete everything from namespace
* `kubectl delete all --all -n <namespace>`
* `kubectl delete namespace <namespace>`


# delete node
- `kubectl drain <node-name> --ignore-daemonsets --delete-local-data`
- `kubectl delete node <node-name>`


# delete stuck namespace
```
NAMESPACE=
kubectl get namespace $NAMESPACE -o json > $NAMESPACE.json
sed -i -e 's/"kubernetes"//' $NAMESPACE.json
kubectl replace --raw "/api/v1/namespaces/$NAMESPACE/finalize" -f ./$NAMESPACE.json
```
