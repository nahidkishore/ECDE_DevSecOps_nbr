

# 🧹 **Kubernetes + Kyverno + Minikube Cleanup Guide**

This guide will help you clean up all resources after completing your admission controller / Kyverno hands-on training.

---

## 🟢 **Step 1: Delete Kyverno Policies & Resources**

First, delete the **ClusterPolicy** you created:

```bash
kubectl delete clusterpolicy block-latest-tag
```

Verify that no Kyverno policies remain:

```bash
kubectl get clusterpolicy
```

---

## 🟢 **Step 2: Uninstall Kyverno (if installed via Helm)**

If Kyverno was installed using Helm (recommended way), uninstall it:

```bash
helm uninstall kyverno -n kyverno
```

Then delete the namespace:

```bash
kubectl delete namespace kyverno
```

Verify that Kyverno resources are gone:

```bash
kubectl get all -n kyverno
```

👉 Should return nothing.

---

## 🟢 **Step 3: Delete Remaining Pods/Deployments**

Check pods in the default namespace:

```bash
kubectl get pods
```

Delete test pods:

```bash
kubectl delete pod nginx
kubectl delete pod nginx-v121 nginx-v122 --ignore-not-found
```

Delete all test deployments:

```bash
kubectl delete deploy --all
```

---

## 🟢 **Step 4: Clean Helm (if you used other charts)**

List Helm releases:

```bash
helm list -A
```

Uninstall unnecessary charts:

```bash
helm uninstall <release-name> -n <namespace>
```

---

## 🟢 **Step 5: Reset Minikube Cluster**

If you want to **reset the Minikube cluster** and start fresh:

```bash
minikube delete
```

This removes:

* Kubernetes cluster
* Configurations
* VMs or containers created by Minikube

Verify deletion:

```bash
minikube status
```

👉 Should show no active cluster.

---

## 🟢 **Step 6: (Optional) Clean Up Docker Images**

Minikube may have pulled Docker images. To free up space, you can prune unused containers and images:

```bash
docker system prune -af
```

⚠️ Warning: This will delete **all stopped containers and unused images**.

---

## ✅ **Final Verification**

After cleanup, check all namespaces:

```bash
kubectl get all --all-namespaces
```

👉 Only the default system namespaces should remain (`kube-system`, `default`, `kube-public`).

Check Minikube status:

```bash
minikube status
```


