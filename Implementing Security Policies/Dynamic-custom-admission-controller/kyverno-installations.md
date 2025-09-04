
# resources:

https://kyverno.io/docs/installation/methods/
https://kyverno.io/docs/policy-types/cluster-policy/policy-rules/


# helm3 install

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```
# install kyverno
```bash
# Add repo
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

# verifications

```bash
 kubectl get pods -n kyverno
 
 kubectl get mutatingwebhookconfigurations.admissionregistration.k8s.io | grep kyverno

 kubectl get validatingwebhookconfigurations.admissionregistration.k8s.io | grep kyverno
 
 kubectl get mutatingwebhookconfigurations,validatingwebhookconfigurations

```