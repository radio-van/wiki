# Contents

- [get pods](#get pods)
- [frontend](#frontend)
- [create pod](#create pod)
    - [cli](#create pod#cli)
    - [frontend](#create pod#frontend)
- [edit deployment](#edit deployment)
- [edit secrets](#edit secrets)
- [postgres pod](#postgres pod)
- [gpu pod](#gpu pod)
- [delete pod](#delete pod)

# get pods
```bash
  kubectl -n <namespace> get pods
```

# frontend
```bash
  kubectl proxy
```
connect to `http://localhost:8001/api/v1/namespaces/kubernator/services/kubernator/proxy/#/catalog`

# create pod
## cli
`kubectl run -n <namespace> --image=<image> <pod_name>`
## frontend
`frontend/<namespace>/Pod/<+>`  

# edit deployment
`kubectl -n <namespace> edit deployment`  
replace  
```bash
   - command:
       - bash
```
with  
`-command: ['sleep', '500m']`  
to be able to connect to pod (instead it will fall into recreating loop)  

# edit secrets
* `kubectl -n <namespace> get secrets`
* `kubectl -n <namespace> edit secret <secret>`
* paste secret in base64: `echo 'string' | base64`

# postgres pod
* create pod  
    `kubectl run -n <namespace> --image=postgres:9.2 tmp-psql`
* connect to database (from the pod)  
  `psql -h <database> -U postgres`

# gpu pod
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

# delete pod
To prevent pod from recreating the deployment must be deleted  
`kubectl -n <namespace> delete deploy <deploy_name>`
