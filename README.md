# 🚀 Kubernetes Reliability and Resilience Checklist

## **1️⃣ Verify the Number of Replicas for Each Deployment**

### **🔍 How to Check**
```sh
kubectl get deployments -A -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,REPLICAS:.spec.replicas,AVAILABLE:.status.availableReplicas"
```
For a specific namespace:
```sh
kubectl get deployments -n <namespace>
```

### **✅ How to Improve**
- Ensure at least **two replicas** for high availability.
- Use **Horizontal Pod Autoscaler (HPA)**:
  ```yaml
  apiVersion: autoscaling/v2
  kind: HorizontalPodAutoscaler
  metadata:
    name: my-app-hpa
    namespace: my-namespace
  spec:
    minReplicas: 2
    maxReplicas: 10
    metrics:
      - type: Resource
        resource:
          name: cpu
          targetAverageUtilization: 70
  ```
- Use **Karpenter for auto-scaling node capacity**.

### **⚠️ Reliability Concerns**
- If **replicas = 1**, failure means downtime.
- Improper **HPA settings** may lead to over/under-scaling.

---

## **2️⃣ Validate Karpenter Auto-Scaling Configuration**

### **🔍 How to Check**
```sh
kubectl get pods -n karpenter
kubectl get provisioners -A
```
Check logs for Karpenter issues:
```sh
kubectl logs -l app.kubernetes.io/name=karpenter -n karpenter
```

### **✅ How to Improve**
- Ensure Karpenter **automatically provisions nodes**:
  ```yaml
  apiVersion: karpenter.k8s.aws/v1alpha5
  kind: Provisioner
  metadata:
    name: default
  spec:
    requirements:
      - key: "node.kubernetes.io/instance-type"
        operator: In
        values: ["m5.large", "m5.xlarge"]
  ```

### **⚠️ Reliability Concerns**
- **Slow scaling may cause pod failures** if nodes aren’t provisioned fast enough.
- **Over-scaling increases costs unnecessarily**.

---

## **3️⃣ Check for Pod Disruption Budgets (PDBs)**

### **🔍 How to Check**
```sh
kubectl get pdb -A
```
For detailed config:
```sh
kubectl describe pdb -n <namespace>
```

### **✅ How to Improve**
- Define **PDBs to prevent too many disruptions**:
  ```yaml
  apiVersion: policy/v1
  kind: PodDisruptionBudget
  metadata:
    name: my-app-pdb
    namespace: my-namespace
  spec:
    minAvailable: 1
    selector:
      matchLabels:
        app: my-app
  ```

### **⚠️ Reliability Concerns**
- **Missing PDBs can cause mass pod evictions**.
- **Too strict PDBs (e.g., `minAvailable: 100%`) prevent node draining**.

---

## **4️⃣ Verify Ingress Configuration**

### **🔍 How to Check**
```sh
kubectl get ingress -A
```
For detailed rules:
```sh
kubectl describe ingress <ingress-name> -n <namespace>
```

### **✅ How to Improve**
- Ensure **Ingress is properly configured with an ALB**:
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: my-app-ingress
    annotations:
      kubernetes.io/ingress.class: "alb"
  spec:
    rules:
      - host: my-app.example.com
        http:
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: my-app
                  port:
                    number: 80
  ```

### **⚠️ Reliability Concerns**
- **Misconfigured TLS certificates** can break traffic.
- **Missing health checks** may cause **traffic routing to failing pods**.

---

## **5️⃣ Validate Kyverno Policies**

### **🔍 How to Check**
```sh
kubectl get clusterpolicies -A
kubectl get policies -A
```
For policy logs:
```sh
kubectl logs -l kyverno.io/component=kyverno
```

### **✅ How to Improve**
- Ensure security policies like **image signing** and **enforcement of best practices**:
  ```yaml
  apiVersion: kyverno.io/v1
  kind: ClusterPolicy
  metadata:
    name: enforce-image-signing
  spec:
    validationFailureAction: Enforce
    rules:
      - name: check-image-signature
        match:
          resources:
            kinds:
              - Pod
        validate:
          message: "Container images must be signed"
  ```

### **⚠️ Reliability Concerns**
- **Too strict Kyverno policies can block deployments**.
- **Lack of policies exposes the cluster to security risks**.

---

## **6️⃣ Ensure Helm-Deployed OSS Components Are Properly Configured**

### **🔍 How to Check**
```sh
helm list -A
helm get values <release-name> -n <namespace>
```

### **✅ How to Improve**
- Ensure custom configurations are applied for **persistence, security, and monitoring**.

### **⚠️ Reliability Concerns**
- **Using default Helm values can lead to insecure setups**.
- **Misconfigured storage can lead to data loss**.

---

## **7️⃣ Verify GitLab and ArgoCD CI/CD Pipelines**

### **🔍 How to Check GitLab Pipelines**
```sh
kubectl get pods -n gitlab
kubectl logs -n gitlab -l app=gitlab-runner
```

### **🔍 How to Check ArgoCD Deployments**
```sh
kubectl get applications -n argocd
kubectl describe application <app-name> -n argocd
```

### **✅ How to Improve**
- Ensure **GitLab CI/CD integrates with Kubernetes**.
- Ensure **ArgoCD syncs Git repositories automatically**.

### **⚠️ Reliability Concerns**
- **Pipeline failures can block releases**.
- **ArgoCD misconfiguration may lead to drift between Git and Kubernetes**.

---

## **🔟 Additional Checks**
✅ **Backup & Disaster Recovery (Velero)**
```sh
kubectl get backup -n velero
```
✅ **AWS Secret Management**
```sh
aws secretsmanager list-secrets
```
✅ **Certificate Expiry Monitoring**
```sh
kubectl get secret -A
```

---

🚀 **This checklist ensures high availability, security, and resilience for Kubernetes workloads!**
