# Kubernetes & AWS Cloud Reliability Audit Checklist

## Reliability & Security Audit Components

### **Existing Checklist:**

#### **1. More than 1 replica is used**
- **Check:**
  - Run `kubectl get deploy -o=jsonpath='{.items[*].spec.replicas}'` to check the replica count.
  - Ensure all critical applications have more than one replica.
- **Improve:**
  - Use Horizontal Pod Autoscaler (HPA) to dynamically adjust replica counts.
  - Set `minReplicas` > 1 in deployment manifests.

#### **2. Pod Disruption Budgets (PDB) are in place and correctly configured**
- **Check:**
  - Run `kubectl get pdb -A` to list PDBs.
  - Ensure critical applications have PDBs configured to prevent downtime.
- **Improve:**
  - Define PDBs that allow minimal pod disruption.
  - Example YAML:
    ```yaml
    apiVersion: policy/v1
    kind: PodDisruptionBudget
    metadata:
      name: my-app-pdb
    spec:
      minAvailable: 1
      selector:
        matchLabels:
          app: my-app
    ```

#### **3. Topology Spread Constraints are in place and correctly configured**
- **Check:**
  - Run `kubectl get deployments -o yaml | grep topologySpreadConstraints`.
- **Improve:**
  - Define topology spread constraints to distribute workloads across nodes and availability zones.
  - Example YAML:
    ```yaml
    topologySpreadConstraints:
    - maxSkew: 1
      topologyKey: topology.kubernetes.io/zone
      whenUnsatisfiable: DoNotSchedule
      labelSelector:
        matchLabels:
          app: my-app
    ```

#### **4. Resource requests and limits are set and reasonable**
- **Check:**
  - Run `kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.containers[*].resources}{"\n"}{end}'`.
- **Improve:**
  - Define CPU & memory limits to prevent resource starvation:
    ```yaml
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
    ```

#### **5. Open Source Software (OSS) components used are properly configured**
- **Check:**
  - Verify Helm values are set correctly.
  - Check configurations in Helm releases: `helm list -A`.
- **Improve:**
  - Keep Helm charts updated.
  - Implement version control on Helm values.

#### **6. Useful metrics are collected from each OSS component**
- **Check:**
  - Verify Prometheus metrics with `kubectl get pods -n monitoring | grep prometheus`.
- **Improve:**
  - Install Prometheus & Grafana to visualize system health.
  - Enable metrics collection in applications.

#### **7. OpenTelemetry (OTEL) instrumentation is in place**
- **Check:**
  - Validate tracing setup in applications.
  - Check `otel-collector` logs.
- **Improve:**
  - Integrate OTEL SDKs in applications.
  - Deploy OTEL Collector.

#### **8. Liveness checks are in place**
- **Check:**
  - Run `kubectl get deploy -o=jsonpath='{.items[*].spec.template.spec.containers[*].livenessProbe}'`.
- **Improve:**
  - Define liveness probes to detect application failures.
    ```yaml
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    ```

#### **9. Other reliability concerns should be documented**
- **Check:**
  - Identify frequent issues in logs (`kubectl logs <pod-name>`).
  - Gather reports from monitoring dashboards.
- **Improve:**
  - Automate issue detection with alerts.

---

### **Additional Checklist Items for Security & Reliability**

#### **10. Role-Based Access Control (RBAC) is correctly configured**
- **Check:**
  - Run `kubectl get roles,rolebindings,clusterroles,clusterrolebindings -A`.
- **Improve:**
  - Implement least privilege principles in RBAC policies.

#### **11. Network Policies are enforced**
- **Check:**
  - Run `kubectl get networkpolicy -A`.
- **Improve:**
  - Restrict pod communication using network policies:
    ```yaml
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
      name: deny-all
      namespace: default
    spec:
      podSelector: {}
      policyTypes:
      - Ingress
      - Egress
    ```

#### **12. Secrets Management is properly handled**
- **Check:**
  - Run `kubectl get secrets -A`.
- **Improve:**
  - Use AWS Secrets Manager or Kubernetes Secrets.
  - Ensure secrets are encrypted at rest.

#### **13. Persistent Volumes (PV) & StorageClass configurations**
- **Check:**
  - Run `kubectl get pvc -A`.
- **Improve:**
  - Use dynamically provisioned storage classes.

#### **14. Cluster AutoScaling & Node AutoScaler**
- **Check:**
  - Ensure Karpenter or Cluster Autoscaler is deployed.
- **Improve:**
  - Optimize scaling policies for workload demand.

#### **15. Backup & Disaster Recovery Strategies**
- **Check:**
  - Ensure Velero or AWS Backup is configured.
- **Improve:**
  - Regularly test disaster recovery plans.

#### **16. Certificate Management & Expiry Monitoring**
- **Check:**
  - Run `kubectl get certs -A` (for cert-manager).
- **Improve:**
  - Automate certificate renewal using cert-manager.

#### **17. Image Security & Vulnerability Scanning**
- **Check:**
  - Ensure Trivy or Clair is scanning container images.
- **Improve:**
  - Regularly scan images for vulnerabilities.

#### **18. AWS Security Best Practices**
- **Check:**
  - Use AWS Security Hub and GuardDuty for threat detection.
- **Improve:**
  - Implement IAM least privilege policies.
  - Encrypt EBS volumes & S3 buckets.

---

## **Conclusion**
Regularly auditing Kubernetes & AWS infrastructure improves security, reliability, and efficiency. Use this checklist to maintain best practices and mitigate risks in cloud environments.
