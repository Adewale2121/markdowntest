# **SaaS Support Team - Comprehensive Kubernetes Component Audit**

This document provides an audit of critical components in the Kubernetes environment, including best practices, verification commands, and remediation steps.

---

## **1. Replica Count Verification**
### **Check if more than 1 replica is used**
#### **Command:**
```sh
kubectl get deployment,statefulset -A | awk '{print $1, $2, $3}'
```
- Lists all deployments and stateful sets with their replica counts.

#### **Possible Issues & Fix:**
- **Issue:** Some components have only 1 replica.
- **Fix:** Increase the replica count:
  ```sh
  kubectl scale deployment <deployment-name> --replicas=2
  ```

---

## **2. Pod Disruption Budgets (PDBs)**
### **Check if PDBs are correctly configured**
#### **Command:**
```sh
kubectl get pdb -A
```
- Checks if PDBs are applied to ensure high availability.

#### **Possible Issues & Fix:**
- **Issue:** No PDBs or incorrect values.
- **Fix:** Apply a correct PDB:
  ```yaml
  apiVersion: policy/v1
  kind: PodDisruptionBudget
  metadata:
    name: my-pdb
  spec:
    minAvailable: 1
    selector:
      matchLabels:
        app: my-app
  ```

---

## **3. Horizontal Pod Autoscaler (HPA)**
### **Check if auto-scaling is enabled**
#### **Command:**
```sh
kubectl get hpa -A
```
- Lists HPAs for all deployments.

#### **Possible Issues & Fix:**
- **Issue:** HPA missing, leading to manual scaling.
- **Fix:** Apply an HPA to scale pods automatically:
  ```yaml
  apiVersion: autoscaling/v2
  kind: HorizontalPodAutoscaler
  metadata:
    name: my-app-hpa
  spec:
    minReplicas: 2
    maxReplicas: 5
    metrics:
      - type: Resource
        resource:
          name: cpu
          targetAverageUtilization: 70
  ```

---

## **4. Node Resource Availability**
### **Check if nodes have sufficient CPU and Memory**
#### **Command:**
```sh
kubectl top nodes
```
- Displays CPU and memory usage of nodes.

#### **Possible Issues & Fix:**
- **Issue:** Nodes running out of resources.
- **Fix:** Add new nodes, adjust pod resource limits, or enable cluster auto-scaling.

---

## **5. Persistent Volume Claims (PVC) Health**
### **Check PVC status**
#### **Command:**
```sh
kubectl get pvc -A
```
- Lists PVCs and their statuses.

#### **Possible Issues & Fix:**
- **Issue:** PVC stuck in `Pending` state.
- **Fix:** Ensure a matching Persistent Volume (PV) exists.
  ```sh
  kubectl get pv -A
  ```

---

## **6. Ingress Configuration & Health**
### **Check Ingress Controllers**
#### **Command:**
```sh
kubectl get ingress -A
```
- Lists all ingress rules.

#### **Possible Issues & Fix:**
- **Issue:** Ingress misconfigured, leading to connectivity issues.
- **Fix:** Ensure the correct ingress class is being used and DNS records are properly set.

---

## **7. Network Policies**
### **Check if network policies are enforced**
#### **Command:**
```sh
kubectl get networkpolicy -A
```
- Lists all network policies.

#### **Possible Issues & Fix:**
- **Issue:** No network policies, allowing unrestricted pod-to-pod communication.
- **Fix:** Define a restrictive network policy:
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: deny-all
  spec:
    podSelector: {}
    policyTypes:
      - Ingress
  ```

---

## **8. Secret Management**
### **Check if Kubernetes Secrets exist**
#### **Command:**
```sh
kubectl get secrets -A
```
- Lists secrets stored in Kubernetes.

#### **Possible Issues & Fix:**
- **Issue:** Sensitive data stored as plain text in ConfigMaps.
- **Fix:** Use Kubernetes secrets:
  ```sh
  kubectl create secret generic db-secret --from-literal=password=supersecurepassword
  ```

---

## **9. Service Account Permissions**
### **Check if service accounts have excessive privileges**
#### **Command:**
```sh
kubectl get serviceaccount -A
kubectl get clusterrolebinding -A
```
- Checks which service accounts have high privileges.

#### **Possible Issues & Fix:**
- **Issue:** Service accounts have `cluster-admin` privileges.
- **Fix:** Restrict permissions using RBAC:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: restricted-rolebinding
  subjects:
      - kind: ServiceAccount
        name: my-service-account
  roleRef:
    kind: Role
    name: restricted-role
    apiGroup: rbac.authorization.k8s.io
  ```

---

## **10. Pod Logs & Errors**
### **Check for errors in pod logs**
#### **Command:**
```sh
kubectl logs -l app=my-app --tail=50
```
- Retrieves the last 50 lines of logs for a specific application.

#### **Possible Issues & Fix:**
- **Issue:** Frequent errors or crashes.
- **Fix:** Debug logs and check resource limits.

---

## **Final Action Items**
1. **Document Findings**: Record identified issues and recommendations.
2. **Submit Merge Requests**: Fix minor configuration issues.
3. **Engage Teams**: Address larger concerns with responsible teams.
4. **Improve Monitoring**: Implement continuous observability best practices.

---

This expanded checklist provides a **comprehensive audit** for **Kubernetes reliability, security, and performance**. 🚀
