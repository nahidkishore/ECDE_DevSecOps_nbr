# Kyverno Dynamic Admission Controller Hands-on Lab

This lab will guide you through installing and using Kyverno, a Kubernetes-native policy engine that allows you to create custom admission control policies.

## Prerequisites
- A Kubernetes cluster (v1.14+)
- kubectl configured to access your cluster
- Helm (v3+) for installation (optional)

## Part 1: Installation

### Option 1: Install with Helm (Recommended)

```bash
# Add the Kyverno Helm repository
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update

# Install in kyverno namespace with HA + metrics
helm upgrade --install kyverno kyverno/kyverno \
  --namespace kyverno --create-namespace \
  --set replicaCount=3 \
  --set resources.requests.cpu=200m \
  --set resources.requests.memory=256Mi \
  --set resources.limits.cpu=500m \
  --set resources.limits.memory=512Mi \
  --set metricsService.create=true \
  --wait

```

### Option 2: Install with kubectl

```bash
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.10.0/install.yaml
```

### Verify Installation

```bash
kubectl get pods -n kyverno
# Should show kyverno pods running

kubectl -n kyverno get pods
kubectl get mutatingwebhookconfigurations.admissionregistration.k8s.io | grep kyverno
kubectl get validatingwebhookconfigurations.admissionregistration.k8s.io | grep kyverno


kubectl get mutatingwebhookconfigurations,validatingwebhookconfigurations


# Should show kyverno webhooks
```

<img width="720" height="321" alt="Image" src="https://github.com/user-attachments/assets/0f319945-ec68-4208-8b39-93ca8d5744a0" />

<img width="719" height="429" alt="Image" src="https://github.com/user-attachments/assets/1e767713-b48d-49dd-b576-f1f601e2b8ce" />

<img width="720" height="426" alt="Image" src="https://github.com/user-attachments/assets/be759762-a051-48d1-9513-7b0a1888e451" />


## Part 2: Basic Policy Examples

### Example 1: Require Labels on Pods

Create a file named `require-labels.yaml`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
spec:
  validationFailureAction: Enforce
  rules:
  - name: check-for-labels
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "All pods must have 'app' and 'owner' labels"
      pattern:
        metadata:
          labels:
            app: "?*"
            owner: "?*"
```

Apply the policy:

```bash
kubectl apply -f require-labels.yaml
```

Test the policy:

```bash
# This should fail
kubectl run nginx --image=nginx

# This should pass
kubectl run nginx --image=nginx --labels="app=nginx,owner=team-a"
```


```bash
vim require-labels.yaml
   57  cat require-labels.yaml
   58  kubectl apply -f require-labels.yaml
   60  kubectl run nginx --image=nginx
   61  kubectl run nginx --image=nginx --labels="app=nginx,owner=team-a"
   63  kubectl get pods
```

<img width="571" height="334" alt="Image" src="https://github.com/user-attachments/assets/8bef172a-8eb7-42a0-ae8a-82ef8758dcc3" />

<img width="642" height="323" alt="Image" src="https://github.com/user-attachments/assets/2a6e80a7-18aa-46a2-acd4-f110767bb2b4" />



### Example 2: Block Latest Image Tags

Create `block-latest-tag.yaml`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: block-latest-tag
spec:
  validationFailureAction: Enforce
  rules:
  - name: block-latest-tag
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Using 'latest' tag is not allowed"
      pattern:
        spec:
          containers:
          - image: "!*:latest"
```

Apply and test:

```bash
kubectl apply -f block-latest-tag.yaml

# This should fail
kubectl run nginx --image=nginx:latest

# This should pass
kubectl run nginx --image=nginx:1.21
```


```bash
vim block-latest-tag.yaml
   65  ls
   66  cat block-latest-tag.yaml
   67  kubectl apply -f block-latest-tag.yaml
   68  kubectl run nginx --image=nginx:latest
   69  kubectl get clusterpolicy
   70  kubectl delete clusterpolicy require-labels
   71  kubectl get clusterpolicy
   72  kubectl run nginx-block-tag --image=nginx:latest
   73  kubectl run nginx-allow-tag --image=nginx:1.21
   74  kubectl get ns
   75  kubectl get pods
```

<img width="689" height="309" alt="Image" src="https://github.com/user-attachments/assets/ed503315-feda-480c-8a01-ffcf1c676d45" />

<img width="678" height="436" alt="Image" src="https://github.com/user-attachments/assets/10e3a1bc-9d06-48e2-8bf3-3333a990c0a4" />

### Example 3: Mutate Resources (Add Default Labels)

Create `add-default-labels.yaml`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-default-labels
spec:
  rules:
  - name: add-default-labels
    match:
      resources:
        kinds:
        - Pod
    mutate:
      patchStrategicMerge:
        metadata:
          labels:
            +(environment): development
            +(managed-by): kyverno
```

Apply and test:

```bash
kubectl apply -f add-default-labels.yaml

# kubectl run nginx --image=nginx -o yaml --dry-run=client
kubectl run nginx-add-default-labels --image=nginx -o yaml --dry-run=client

# Check the output to see the added labels
```

```bash
cat add-default-labels.yaml
   85  kubectl apply -f add-default-labels.yaml
   86  kubectl get clusterpolicy
   87  kubectl run nginx-add-default-labels --image=nginx -o yaml --dry-run=client

# output

ubuntu@ip-172-31-38-4:~$ kubectl apply -f add-default-labels.yaml
clusterpolicy.kyverno.io/add-default-labels created
ubuntu@ip-172-31-38-4:~$ kubectl get clusterpolicy
NAME                 ADMISSION   BACKGROUND   READY   AGE   MESSAGE
add-default-labels   true        true         True    8s    Ready
ubuntu@ip-172-31-38-4:~$ kubectl run nginx-add-default-labels --image=nginx -o yaml --dry-run=client
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-add-default-labels
  name: nginx-add-default-labels
spec:
  containers:
  - image: nginx
    name: nginx-add-default-labels
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


```



## Part 3: Advanced Policy Examples

### Example 4: Validate Resource Requests/Limits

Create `require-resource-requests.yaml`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-resource-requests
spec:
  validationFailureAction: Enforce
  rules:
  - name: validate-resources
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "CPU and memory resource requests and limits are required"
      pattern:
        spec:
          containers:
          - resources:
              requests:
                memory: "?*"
                cpu: "?*"
              limits:
                memory: "?*"
                cpu: "?*"
```

Apply and test:

```bash
kubectl apply -f require-resource-requests.yaml

# This should fail
kubectl run nginx --image=nginx

# This should pass 
# its show request issue for kubernetes new versions
kubectl run nginx --image=nginx --requests="cpu=100m,memory=128Mi" --limits="cpu=200m,memory=256Mi"

```

Create `nginx-with-resources`:
```yml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-resources
  labels:
    app: nginx
    owner: dev-team
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```


```bash

kubectl get clusterpolicy
  101  kubectl apply -f require-resource-requests.yaml
  102  kubectl get clusterpolicy
  103  kubectl get pods
  104  kubectl run nginx-resource-requests --image=nginx

  mkdir resource-requests
  112  mv require-resource-requests.yaml resource-requests
  113  ls
  114  rm equire-resource-requests.yaml
  115  ls
  116  cd resource-requests/
  117  ls
  118  vim nginx-with-resources.yaml
  119  cat require-resource-requests.yaml
  120  cat nginx-with-resources.yaml
  121  kubectl apply -f require-resource-requests.yaml
  122  kubectl get clusterpolicy
  123  kubectl apply -f nginx-with-resources.yaml
  124  kubectl get pods
  125  kubectl delete pods nginx-with-resources
```

<img width="671" height="249" alt="Image" src="https://github.com/user-attachments/assets/a58c40ea-166c-4162-b983-2c742f254ead" />

<img width="633" height="422" alt="Image" src="https://github.com/user-attachments/assets/f6698321-2884-4bc7-81fe-ce8108075cc4" />

<img width="302" height="175" alt="Image" src="https://github.com/user-attachments/assets/6f265fe4-be59-4c56-99ab-78545770e279" />



<img width="657" height="382" alt="Image" src="https://github.com/user-attachments/assets/d0c774e1-a898-4a6b-b583-9b4dfb3954ad" />



### Example 5: Restrict Image Registries

Create `allowed-registries.yaml`:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: allowed-registries
spec:
  validationFailureAction: Enforce 
  rules:
  - name: check-image-registry
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Only images from approved registries (docker.io, ECR, ACR, GCR) are allowed"
      pattern:
        spec:
          containers:
          - image: "docker.io/* | *.dkr.ecr.*.amazonaws.com/* | *.azurecr.io/* | *.gcr.io/* | gcr.io/*"
```

Apply and test:

```bash
kubectl apply -f allowed-registries.yaml

# This should fail
kubectl run nginx --image=quay.io/nginx

# This should pass
kubectl run nginx --image=docker.io/nginx
```


# History 

```bash

cd Restrict_Image-Registries/
  136  ls
  137  vim allowed-registries.yaml
  138  cat allowed-registries.yaml
  139  kubectl get clusterpolicy
  140  kubectl delete clusterpolicy require-resource-requests
  141  kubectl get pods
  142  kubectl delete pods nginx-allow-tag nginx
  143  kubectl get pods
  144  cat allowed-registries.yaml
  145  kubectl apply -f allowed-registries.yaml
  146  kubectl get clusterpolicy
  147  kubectl run nginx --image=quay.io/nginx
  148  kubectl run nginx --image=docker.io/nginx
  149  kubectl get pods
  150  kubectl run nginx --image=test.io/nginx

# full execute command
ubuntu@ip-172-31-38-4:~$ cd Restrict_Image-Registries/
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ ls
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ vim allowed-registries.yaml
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ cat allowed-registries.yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: allowed-registries
spec:
  validationFailureAction: Enforce
  rules:
  - name: check-image-registry
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Only images from approved registries (docker.io, ECR, ACR, GCR) are allowed"
      pattern:
        spec:
          containers:
          - image: "docker.io/* | *.dkr.ecr.*.amazonaws.com/* | *.azurecr.io/* | *.gcr.io/* | gcr.io/*"
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl get clusterpolicy
NAME                        ADMISSION   BACKGROUND   READY   AGE   MESSAGE
require-resource-requests   true        true         True    18m   Ready
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl delete clusterpolicy require-resource-requests
clusterpolicy.kyverno.io "require-resource-requests" deleted
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
nginx             1/1     Running   0          64m
nginx-allow-tag   1/1     Running   0          48m
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl delete pods nginx-allow-tag nginx
pod "nginx-allow-tag" deleted
pod "nginx" deleted
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl get pods
No resources found in default namespace.
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ cat allowed-registries.yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: allowed-registries
spec:
  validationFailureAction: Enforce
  rules:
  - name: check-image-registry
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Only images from approved registries (docker.io, ECR, ACR, GCR) are allowed"
      pattern:
        spec:
          containers:
          - image: "docker.io/* | *.dkr.ecr.*.amazonaws.com/* | *.azurecr.io/* | *.gcr.io/* | gcr.io/*"
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl apply -f allowed-registries.yaml
clusterpolicy.kyverno.io/allowed-registries created
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl get clusterpolicy
NAME                 ADMISSION   BACKGROUND   READY   AGE   MESSAGE
allowed-registries   true        true         True    5s    Ready
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl run nginx --image=quay.io/nginx
Error from server: admission webhook "validate.kyverno.svc-fail" denied the request:

resource Pod/default/nginx was blocked due to the following policies

allowed-registries:
  check-image-registry: 'validation error: Only images from approved registries (docker.io,
    ECR, ACR, GCR) are allowed. rule check-image-registry failed at path /spec/containers/0/image/'
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl run nginx --image=docker.io/nginx
pod/nginx created
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          7s
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$ kubectl run nginx --image=test.io/nginx
Error from server: admission webhook "validate.kyverno.svc-fail" denied the request:

resource Pod/default/nginx was blocked due to the following policies

allowed-registries:
  check-image-registry: 'validation error: Only images from approved registries (docker.io,
    ECR, ACR, GCR) are allowed. rule check-image-registry failed at path /spec/containers/0/image/'
ubuntu@ip-172-31-38-4:~/Restrict_Image-Registries$
```




# Optional part (Ignore it)

## Part 4: Policy Reporting

Kyverno provides policy reports to view policy violations.

```bash
# View cluster policy reports
kubectl get clusterpolicyreports

# View policy reports for a specific namespace
kubectl get policyreports -n <namespace>

# Get details of a specific report
kubectl describe clusterpolicyreport <report-name>
```

## Part 5: Cleanup

To uninstall Kyverno:

```bash
# If installed with Helm
helm uninstall kyverno -n kyverno

# If installed with kubectl
kubectl delete -f https://github.com/kyverno/kyverno/releases/download/v1.10.0/install.yaml

# Delete any policies you created
kubectl delete clusterpolicy require-labels block-latest-tag add-default-labels require-resource-requests allowed-registries
```

## Next Steps

1. Explore more policy examples in the [Kyverno policies library](https://kyverno.io/policies/)
2. Learn about policy exceptions for cases where you need to bypass policies
3. Investigate generate policies that can create resources automatically
4. Explore Kyverno CLI for testing policies locally



# History

```bash

35  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
   36  chmod 700 get_helm.sh
   37  ./get_helm.sh
   38  helm version
   39  helm repo add kyverno https://kyverno.github.io/kyverno/
   40  helm repo update
   41  helm upgrade --install kyverno kyverno/kyverno   --namespace kyverno --create-namespace   --set replicaCount=3   --set resources.requests.cpu=200m   --set resources.requests.memory=256Mi   --set resources.limits.cpu=500m   --set resources.limits.memory=512Mi   --set metricsService.create=true   --wait
   42  kubectl get pods -n kyverno
   43  kubectl get mutatingwebhookconfigurations.admissionregistration.k8s.io | grep kyverno
   44  kubectl get validatingwebhookconfigurations.admissionregistration.k8s.io | grep kyverno
   45  kubectl get mutatingwebhookconfigurations,validatingwebhookconfigurations

```