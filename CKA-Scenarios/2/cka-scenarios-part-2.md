# Kezie Iroha 
# Certified Kubernetes Administrator (CKA) Exam Preparation Scenarios
# Based on Kubernetes 1.32 Features

Below is a **fully expanded** set of scenarios **with no hidden sections**, covering **all 10** recommended scenarios in detail.

## Table of Contents
1. [Lab Setup & Prerequisites](#lab-setup--prerequisites)
2. [Scenario 1: Multi-Component Control Plane Crash](#scenario-1-multi-component-control-plane-crash)
3. [Scenario 2: Complex Scheduling Issues](#scenario-2-complex-scheduling-issues)
4. [Scenario 3: Advanced Networking](#scenario-3-advanced-networking)
5. [Scenario 4: Storage Chaos](#scenario-4-storage-chaos)
6. [Scenario 5: Security Incident - Privilege Escalation & RBAC](#scenario-5-security-incident---privilege-escalation--rbac)
7. [Scenario 6: Complex Cluster Upgrade & Node Crash](#scenario-6-complex-cluster-upgrade--node-crash)
8. [Scenario 7: Full Disaster Simulation](#scenario-7-full-disaster-simulation)
9. [Scenario 8: Database-Centric Postgres Operator](#scenario-8-database-centric-postgres-operator)
10. [Scenario 9 (Optional): Performance Diagnostics](#scenario-9-optional-performance-diagnostics)
11. [Scenario 10 (Optional): Multi-Tenancy Testing](#scenario-10-optional-multi-tenancy-testing)
12. [CRD & Ingress/Gateway API Enhancements](#crd--ingressgateway-api-enhancements)
13. [Time Management Strategies](#time-management-strategies)
14. [Command Reference & Imperative Approaches](#command-reference--imperative-approaches)
15. [Version-Specific Notes (1.28–1.32)](#version-specific-notes-128132)
16. [Final Notes](#final-notes)

---

## Lab Setup & Prerequisites

### Recommended Lab Resources & Preliminaries
- **Minimum Node Specs**: 2 CPUs and 4GB RAM per VM recommended.
- **Kubernetes Versions**: This guide specifically targets **v1.32** but notes changes for 1.28–1.31. With **dockershim removed** in newer releases, use **containerd** or **CRI-O**.
- **PodSecurityStandards**: Since **PodSecurityPolicy** was removed in v1.25, use **Pod Security Admission** or OPA/Gatekeeper (Scenario 5).
- **EndpointSlices**: Networking features (Scenario 3) rely on **EndpointSlices** in modern K8s.
- **CSI Drivers**: Scenario 4 references **in-tree** storage, but prefer or migrate to CSI-based drivers for 1.32.
- **etcd Snapshot Strategy**: Keep periodic etcd backups to practice restore scenarios.


### Pre-Scenario Checklist
1. **Confirm VMs** with `vagrant status`.
2. **Check etcd snapshots** for scenarios with corruption.
3. **Verify node resources** (CPU/RAM) meet scenario demands.
4. **Pull needed images** (e.g., `busybox`, `nginx`) in advance.
5. (Optional) **Set up cluster** with PodSecurity standards, CSI drivers, and an Ingress controller to fully emulate K8s v1.32.

### Recommended Scenario Order & Approx. Times
1. **Scenario 1**: ~30–45 min  
2. **Scenario 2**: ~20–30 min  
3. **Scenario 3**: ~25–35 min  
4. **Scenario 4**: ~20–30 min  
5. **Scenario 5**: ~20–30 min  
6. **Scenario 6**: ~30–45 min  
7. **Scenario 7**: ~60–90 min  
8. **Scenario 8**: ~30–60 min  
9. **Scenario 9** (Optional): time varies  
10. **Scenario 10** (Optional): time varies

> **Time Management**: Add ~10–15 minutes buffer per scenario for reading & planning. Tackle simpler ones first.

### Scenario Coverage Matrix (CKA Domains)

| Scenario                               | Cluster Arch/Config | Workloads & Scheduling | Services & Networking | Storage | Troubleshooting | Security | Maintenance |
|----------------------------------------|----------------------|------------------------|-----------------------|---------|-----------------|----------|------------|
| 1. Control Plane Crash                 | ✓                   |                        |                       | ✓       | ✓               |          | ✓          |
| 2. Complex Scheduling                  |                      | ✓                      |                       |         | ✓               |          |            |
| 3. Advanced Networking                 |                      |                        | ✓                     |         | ✓               |          |            |
| 4. Storage Chaos                       |                      |                        |                       | ✓       | ✓               |          |            |
| 5. Security Incident                   |                      |                        |                       |         | ✓               | ✓        |            |
| 6. Complex Upgrade & Node Crash        | ✓                   | ✓                      |                       | ✓       | ✓               |          | ✓          |
| 7. Full Disaster Simulation            | ✓                   | ✓                      | ✓                     | ✓       | ✓               | ✓        | ✓          |
| 8. Postgres Operator (DB-Centric)      |                      |                        | ✓                     | ✓       | ✓               | ✓        | ✓          |
| 9. Performance Diagnostics (Optional)  |                      |                        |                       |         | ✓               |          |            |
| 10. Multi-Tenancy (Optional)           |                      |                        | ✓                     |         | ✓               | ✓        |            |


## Common Pitfalls & Recovery Tips
1. **Failure to Validate**: Always run validation commands after each fix.  
2. **Resource Starvation**: Under-provisioning leads to etcd or control-plane crashes.  
3. **Incorrect Snapshots**: Using outdated etcd snapshots.  
4. **Version Skew**: Mismatched K8s or container runtimes.  
5. **RBAC Gaps**: ServiceAccounts or Roles with no restrictions.  
6. **In-Tree vs. CSI**: Migrate to CSI for real-world readiness.  
7. **PodSecurityPolicy Removal**: Use PodSecurityStandards or OPA.  
8. **CRD Upgrades**: Pay attention to CRD versions in scenario expansions.

## Structured Validation Protocols

To **standardize** scenario outcomes, adopt a consistent validation approach:
1. **Check Node & Pod Health**:
   ```sh
   kubectl get nodes -o wide
   kubectl get pods -A -o wide
   ```
2. **Check Logs**:
   ```sh
   kubectl logs <pod> -n <namespace>
   journalctl -xeu kubelet
   crictl logs <container-id>
   ```
3. **Check API Connectivity**:
   ```sh
   curl -k https://localhost:6443/healthz
   kubectl cluster-info
   ```
4. **Confirm Resources** (e.g., `kubectl describe` for pods, pvc, networkpolicy):
   ```sh
   kubectl describe pod <pod> -n <namespace>
   ```
5. **Look for OOMKills** or CrashLoopBackOff:
   ```sh
   kubectl get pods --field-selector=status.phase!=Running -A
   ```
6. **Monitor** using top or custom metrics:
   ```sh
   kubectl top nodes
   kubectl top pods -n <namespace>
   ```

*(Use these steps **after** each fix or change to confirm everything’s healthy.)*

---

## Scenario 1: Multi-Component Control Plane Crash
**Estimated Time**: ~30–45 minutes

### Overview
Simulate a chain-reaction failure in the control plane (kube-apiserver, kube-scheduler, and kube-controller-manager) on **`k8s-master`**, requiring advanced diagnosis and an **etcd** snapshot recovery. This scenario tests **cluster architecture**, **troubleshooting**, and **maintenance**.

### Enhanced etcd Backup Steps (Kubernetes 1.32 / etcd v3.5+)

```sh
# Create a snapshot with explicit endpoints
sudo ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd/snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Example restore with explicit membership settings
sudo ETCDCTL_API=3 etcdctl snapshot restore /var/lib/etcd/snapshot.db \
  --data-dir=/var/lib/etcd/restored \
  --name=k8s-master \
  --initial-cluster=k8s-master=https://127.0.0.1:2380 \
  --initial-cluster-token=etcd-cluster-1 \
  --initial-advertise-peer-urls=https://127.0.0.1:2380

# Set correct ownership
sudo chown -R etcd:etcd /var/lib/etcd/restored
sudo mv /var/lib/etcd/member /var/lib/etcd/member.bak
sudo mv /var/lib/etcd/restored/member /var/lib/etcd/
```

Use the steps above if you want a more **robust** approach than simply appending data to simulate corruption.

### Fault Injection (on `k8s-master`)
1. **Break kube-scheduler and kube-controller-manager**:
   ```sh
   vagrant ssh k8s-master
   sudo mv /etc/kubernetes/manifests/kube-scheduler.yaml /etc/kubernetes/manifests/kube-scheduler.yaml.bak
   sudo mv /etc/kubernetes/manifests/kube-controller-manager.yaml /etc/kubernetes/manifests/kube-controller-manager.yaml.bak
   sudo systemctl restart kubelet
   ```
2. **Corrupt the etcd data** (simplified approach):
   ```sh
   echo 'corruption' | sudo tee -a /var/lib/etcd/member/snap/db
   sudo systemctl restart etcd
   ```
3. **Force incorrect etcd endpoint** in kube-apiserver:
   ```sh
   sudo sed -i 's/--etcd-servers=https:\/\/127.0.0.1:2379/--etcd-servers=https:\/\/127.0.0.1:9999/' /etc/kubernetes/manifests/kube-apiserver.yaml
   sudo systemctl restart kubelet
   ```

### Diagnostic Steps
- **Check etcd Health**:
  ```sh
  export ETCDCTL_API=3
  etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    endpoint health
  ```
- **Check Control Plane Pods**:
  ```sh
  crictl ps --all
  ls /etc/kubernetes/manifests
  journalctl -u kubelet -f
  ```
- **Check API Server Logs**:
  ```sh
  crictl pods | grep kube-apiserver
  crictl logs <apiserver-container-id>
  ```

### Recovery Steps
1. **Restore etcd from Snapshot** (using either the advanced or simple approach):
   ```sh
   sudo systemctl stop etcd
   sudo mv /var/lib/etcd/member /var/lib/etcd/member.bak
   sudo etcdctl snapshot restore /var/lib/etcd/snapshot.db --data-dir=/var/lib/etcd/member
   sudo systemctl start etcd
   ```
2. **Fix the kube-apiserver manifest**:
   ```sh
   sudo sed -i 's/--etcd-servers=https:\/\/127.0.0.1:9999/--etcd-servers=https:\/\/127.0.0.1:2379/' /etc/kubernetes/manifests/kube-apiserver.yaml
   ```
3. **Restore the scheduler/controller-manager manifests**:
   ```sh
   sudo mv /etc/kubernetes/manifests/kube-scheduler.yaml.bak /etc/kubernetes/manifests/kube-scheduler.yaml
   sudo mv /etc/kubernetes/manifests/kube-controller-manager.yaml.bak /etc/kubernetes/manifests/kube-controller-manager.yaml
   sudo systemctl restart kubelet
   ```

### Validation
```sh
kubectl get pods -n kube-system
kubectl get nodes
etcdctl --endpoints=https://127.0.0.1:2379 endpoint health
```

**Potential Pitfall**: Using the **wrong** snapshot or skipping backups.

---

## Scenario 2: Complex Scheduling Issues
**Estimated Time**: ~20–30 minutes

### Overview
Combine **node taints**, **affinity/anti-affinity**, and **resource quotas** to diagnose unschedulable pods. This scenario tests **workloads & scheduling** as well as **troubleshooting**.

### Fault Injection
1. **Taint a worker node** (e.g., `k8s-worker-1`):
   ```sh
   kubectl taint nodes k8s-worker-1 critical=1:NoSchedule
   ```
2. **Create a namespace** with a strict quota:
   ```sh
   kubectl create ns quota-ns
   cat <<EOF | kubectl apply -n quota-ns -f -
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: compute-quota
   spec:
     hard:
       requests.cpu: "2"
       requests.memory: 2Gi
   EOF
   ```
3. **Deploy a pod** violating taint & resource quota:
   ```sh
   cat <<EOF | kubectl apply -n quota-ns -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: complex-pod
   spec:
     affinity:
       nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
             - matchExpressions:
                 - key: kubernetes.io/hostname
                   operator: In
                   values:
                     - k8s-worker-1
     containers:
       - name: busybox
         image: busybox:1.36
         command: ["sleep", "3600"]
         resources:
           requests:
             cpu: "4"
             memory: "4Gi"
   EOF
   ```

### Diagnostic Steps
- **Describe Pod**:
  ```sh
  kubectl describe pod complex-pod -n quota-ns
  ```
- **Check Taints**:
  ```sh
  kubectl describe node k8s-worker-1
  ```
- **Check Resource Quota**:
  ```sh
  kubectl get resourcequota -n quota-ns
  ```

### Recovery Steps
1. **Reduce** resource requests to fit the quota.
2. **Remove the taint** or add a toleration:
   ```sh
   kubectl taint nodes k8s-worker-1 critical-
   ```

### Validation
```sh
kubectl get pods -n quota-ns -o wide
```

**Potential Pitfall**: Failing both node affinity & taint constraints.

---

## Scenario 3: Advanced Networking
**Estimated Time**: ~25–35 minutes

### Overview
Test how **NetworkPolicies**, **EndpointSlices** (K8s 1.32), and potential **IP conflicts** can disrupt cross-node communication. This scenario emphasizes **services & networking** and **troubleshooting**.

### Multi-tier Variation
Create a **tiered application** to demonstrate more realistic NetworkPolicies:
```sh
kubectl create ns tiered-app
kubectl run frontend --image=nginx -n tiered-app --labels=tier=frontend
kubectl run api --image=nginx -n tiered-app --labels=tier=api
kubectl run db --image=postgres -n tiered-app --labels=tier=database

# NP for frontend -> api
cat <<EOF | kubectl apply -n tiered-app -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
spec:
  podSelector:
    matchLabels:
      tier: frontend
  policyTypes:
  - Ingress
  - Egress
  ingress:
    - from:
      - ipBlock:
          cidr: 0.0.0.0/0
  egress:
    - to:
      - podSelector:
          matchLabels:
            tier: api
EOF

# NP for api -> db
cat <<EOF | kubectl apply -n tiered-app -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
spec:
  podSelector:
    matchLabels:
      tier: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            tier: frontend
  egress:
    - to:
      - podSelector:
          matchLabels:
            tier: database
EOF

# NP for db
cat <<EOF | kubectl apply -n tiered-app -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
  - Ingress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            tier: api
EOF
```

### Fault Injection
1. **Restrictive NP** + **IP conflict**:
   ```sh
   kubectl create ns net-test
   kubectl run web --image=nginx -n net-test --port=80 --labels="app=web"
   kubectl expose pod web -n net-test --port=80 --target-port=80

   cat <<EOF | kubectl apply -n net-test -f -
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: deny-all
   spec:
     podSelector: {}
     policyTypes:
       - Ingress
       - Egress
     ingress: []
     egress: []
   EOF

   # IP conflict on k8s-worker-2
   vagrant ssh k8s-worker-2
   sudo mv /etc/cni/net.d/10-flannel.conflist /etc/cni/net.d/10-flannel.conflist.bak
   sudo systemctl restart kubelet
   ```

### Validation
```sh
kubectl run test-again --rm -it --image=busybox:1.36 -- wget -qO- http://web.net-test
```

**Potential Pitfall**: Overlooking that both **ingress** and **egress** might be blocked by default.

---

## Scenario 4: Storage Chaos
**Estimated Time**: ~20–30 minutes

### Overview
Investigate why PVCs remain in Pending or Lost state in a 3-node lab, due to a **misbehaving volume provisioner**. In K8s 1.32, prefer **CSI** drivers.

### Enhanced CSI Volume Expansion Example
```sh
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expandable-sc
provisioner: my-csi.driver
allowVolumeExpansion: true
parameters:
  type: gp2
EOF

# Basic PVC
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: expandable-pvc
spec:
  storageClassName: expandable-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

# Expand it
kubectl patch pvc expandable-pvc -p '{"spec":{"resources":{"requests":{"storage":"2Gi"}}}}'
```

### Fault Injection
- **Break the default StorageClass**:
  ```sh
  kubectl patch storageclass <default-class> -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
  ```
- **Deploy a StorageClass** referencing a nonexistent provisioner, or remove the CSI driver.
- **Create a PVC** using the broken StorageClass.

### Validation
```sh
kubectl get pvc
kubectl describe pvc
kubectl describe storageclass
```

**Potential Pitfall**: Turning off the default storage class unintentionally.

---

## Scenario 5: Security Incident - Privilege Escalation & RBAC
**Estimated Time**: ~20–30 minutes

### Overview
Test PodSecurityStandards, Admission Controllers, plus advanced RBAC. Also consider **Audit Logging** for better traceability.

### Fault Injection
1. **Deploy malicious container** mounting `/`:
   ```sh
   kubectl apply -f - <<EOF
   apiVersion: v1
   kind: Pod
   metadata:
     name: exploit-pod
   spec:
     containers:
     - name: exploit
       image: busybox
       command: ["sleep", "3600"]
       volumeMounts:
         - mountPath: /host
           name: host-root
     volumes:
       - name: host-root
         hostPath:
           path: /
   EOF
   ```
2. **Attempt RBAC privilege escalation**:
   ```sh
   kubectl create serviceaccount privileged-user
   kubectl create clusterrolebinding privileged-binding --clusterrole=cluster-admin --serviceaccount=default:privileged-user

   # Then test:
   kubectl auth can-i delete pods --as=system:serviceaccount:default:privileged-user
   ```

### Additional Steps
- **Enable Audit Logging**:
  ```sh
  # /etc/kubernetes/audit-policy.yaml
  apiVersion: audit.k8s.io/v1
  kind: Policy
  rules:
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods"]
  ```
  Then set `--audit-policy-file=/etc/kubernetes/audit-policy.yaml` in `/etc/kubernetes/manifests/kube-apiserver.yaml`.

- **Create a restricted RBAC**:
  ```sh
  kubectl create serviceaccount restricted-user
  kubectl create role view-only --verb=get,list --resource=pods
  kubectl create rolebinding restricted-binding --role=view-only --serviceaccount=default:restricted-user
  ```

### Validation
```sh
kubectl auth can-i list pods --as=system:serviceaccount:default:restricted-user
kubectl auth can-i delete pods --as=system:serviceaccount:default:restricted-user
```

**Potential Pitfall**: Overlooking cluster-wide PodSecurity settings or forgetting to block cluster-admin.

---

## Scenario 6: Complex Cluster Upgrade & Node Crash
**Estimated Time**: ~30–45 minutes

### Overview
Simulate a partial upgrade on **`k8s-master`** (control plane only), while **`k8s-worker-1`** and **`k8s-worker-2`** remain un-upgraded, then crash a worker during the process. Mind **container runtime version alignment** to avoid mismatches. Also test **Kubelet Certificate Rotation**.

### Fault Injection
1. **Upgrade kubeadm** on `k8s-master`:
   ```sh
   vagrant ssh k8s-master
   sudo apt-get update && sudo apt-get install -y kubeadm=1.32.0-00
   sudo kubeadm upgrade plan
   sudo kubeadm upgrade apply v1.32.0
   ```
2. **Do NOT upgrade kubelet/kubectl** on `k8s-master` or on the workers.
3. **Crash** the worker `k8s-worker-1`:
   ```sh
   vagrant ssh k8s-worker-1
   sudo systemctl stop kubelet && sudo kill -9 $(pidof kubelet)
   ```

### Diagnostic Steps
- **Check Version Skew**:
  ```sh
  kubectl version --short
  ```
- **Check Node States**:
  ```sh
  kubectl get nodes -o wide
  ```

### Kubelet Certificate Rotation
```sh
sudo kubeadm certs check-expiration
sudo kubeadm certs renew all
sudo openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep "Not After"
```

### Recovery Steps
1. **Upgrade** the rest of the control plane components on `k8s-master`:
   ```sh
   sudo apt-get install -y kubelet=1.32.0-00 kubectl=1.32.0-00
   sudo systemctl restart kubelet
   ```
2. **Upgrade** `k8s-worker-1` and `k8s-worker-2` similarly.
3. **Restart** the crashed worker if needed:
   ```sh
   sudo systemctl start kubelet
   ```

### Validation
```sh
kubectl get nodes -o wide
kubectl version --short
```

**Potential Pitfall**: Upgrading kubeadm but forgetting the container runtime version.

---

## Scenario 7: Full Disaster Simulation
**Estimated Time**: ~60–90 minutes

### Overview
Simulate a worst-case scenario in your 3-node cluster: **`k8s-master`** control plane certs are corrupted, plus **`k8s-worker-1`** and **`k8s-worker-2`** both go offline, while etcd data is damaged. Tests **cluster architecture**, **troubleshooting**, **security**, and **maintenance**.

### Fault Injection
1. **Corrupt the CA** on `k8s-master`:
   ```sh
   vagrant ssh k8s-master
   sudo mv /etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/ca.crt.bak
   sudo rm /etc/kubernetes/pki/ca.key
   ```
2. **Shut down the worker nodes**:
   ```sh
   vagrant halt k8s-worker-1
   vagrant halt k8s-worker-2
   ```
3. **Corrupt etcd**:
   ```sh
   echo "extra data" | sudo tee -a /var/lib/etcd/member/snap/db
   sudo systemctl restart etcd
   ```

### Diagnostic Steps
- **Check PKI**:
  ```sh
  openssl x509 -in /etc/kubernetes/pki/ca.crt -text -noout
  ```
- **etcd Logs**:
  ```sh
  journalctl -u etcd -f
  ```
- **Node States**:
  ```sh
  kubectl get nodes
  ```

### Recovery Steps
1. **Generate new PKI** or restore from a backup:
   ```sh
   sudo kubeadm init phase certs all --apiserver-advertise-address=<ip>
   ```
   - May require re-approving kubelet certs or rejoining nodes with new CA.
2. **Restore etcd Snapshot**:
   ```sh
   sudo etcdctl snapshot restore /path/to/etcd-backup.db --data-dir=/var/lib/etcd/member
   ```
3. **Bring workers online**:
   ```sh
   vagrant up k8s-worker-1
   vagrant up k8s-worker-2
   kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
   ```

### Validation
```sh
kubectl get nodes
kubectl get pods -A
```

**Potential Pitfall**: Skipping re-approval of kubelet certificates.

---

## Scenario 8: Database-Centric Postgres Operator
**Estimated Time**: ~30–60 minutes

### Overview
Focus on **database** reliability in Kubernetes. Use a **Postgres operator** (e.g., CrunchyData, Zalando, or Bitnami) to simulate operator misconfigurations, volume corruption, and failover.

### Fault Injection
- **Corrupt the primary's PVC**:
  ```sh
  vagrant ssh k8s-worker-1
  sudo dd if=/dev/urandom of=/var/lib/kubelet/pods/<podID>/volumes/kubernetes.io~csi/pvc-<uid>/data bs=1M count=10
  ```
- **Simulate replication lag** via network throttling:
  ```sh
  sudo tc qdisc add dev eth0 root netem delay 200ms
  ```

### Diagnostic Steps
- **Check Operator Logs**:
  ```sh
  kubectl logs deploy/my-postgres-operator
  ```
- **Check Postgres Pod Logs**:
  ```sh
  kubectl logs sts/my-db-postgresql-0
  ```
- **Check Replication**:
  ```sh
  kubectl exec -it <postgres-pod> -- psql -U postgres -c 'SELECT * FROM pg_stat_replication;'
  ```
- **Monitor Performance** with built-in metrics, `kubectl top`, or external tools.

### Recovery Steps
1. **Operator Self-Healing**: Confirm if the operator initiates failover.
2. **Remove/Recreate** the corrupt volume.
3. **Backup/Restore** the DB with WAL archiving or `pg_basebackup`.
4. **Remove Throttling**:
  ```sh
  sudo tc qdisc del dev eth0 root
  ```

### Validation
```sh
kubectl get pods -l app.kubernetes.io/component=primary
kubectl exec sts/my-db-postgresql-0 -- psql -U postgres -c "SELECT 1;"
```

**Potential Pitfall**: Not implementing WAL archiving or operator backups, leading to irrecoverable data loss.

---

## (Optional) Scenario 9: Performance Diagnostics
**Time Varies**

### Overview
Add a **performance** dimension to your DBRE practice. Deploy Prometheus and Grafana to visualize cluster and database metrics under stress.

### Steps
1. **Install Prometheus**:
   ```sh
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm install my-prom prometheus-community/prometheus
   ```
2. **Install Grafana**:
   ```sh
   helm repo add grafana https://grafana.github.io/helm-charts
   helm install my-graf grafana/grafana --set adminPassword='admin'
   ```
3. **Scrape DB metrics** or operator metrics.
4. **Simulate Load** on the DB with `pgbench` or custom scripts.

### Validation
- Check **Prometheus** targets.
- Observe CPU/Memory usage under load.
- Identify performance bottlenecks.

**Potential Pitfall**: Overloading the cluster if resources are minimal.

---

## (Optional) Scenario 10: Multi-Tenancy Testing
**Time Varies**

### Overview
Demonstrate resource and network isolation across multiple namespaces.

### Steps
1. **Create namespaces** for different tenants:
   ```sh
   kubectl create ns tenant-a
   kubectl create ns tenant-b
   ```
2. **Resource Quotas & LimitRanges**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: tenant-a-quota
     namespace: tenant-a
   spec:
     hard:
       requests.cpu: "4"
       requests.memory: 4Gi
       limits.cpu: "8"
       limits.memory: 8Gi
       pods: "10"
   EOF

   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: LimitRange
   metadata:
     name: tenant-a-limits
     namespace: tenant-a
   spec:
     limits:
     - default:
         memory: 512Mi
         cpu: 500m
       defaultRequest:
         memory: 256Mi
         cpu: 200m
       type: Container
   EOF
   ```
3. **Network Policy** to block cross-tenant traffic:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: deny-cross-tenant
     namespace: tenant-a
   spec:
     podSelector: {}
     policyTypes:
     - Ingress
     - Egress
     ingress:
     - from:
       - namespaceSelector:
           matchLabels:
             kubernetes.io/metadata.name: tenant-a
     egress:
     - to:
       - namespaceSelector:
           matchLabels:
             kubernetes.io/metadata.name: tenant-a
   EOF
   ```

### Validation
```sh
kubectl run test-tenant --rm -it --image=busybox:1.36 --namespace=tenant-a -- wget -qO- http://<podInTenantB>
```

**Potential Pitfall**: Overlooking cross-tenant references or failing to label namespaces properly.

---

## CRD & Ingress/Gateway API Enhancements

To further **align** with modern K8s usage and exam coverage:

1. **CustomResourceDefinitions**:
   - Create a simple CRD (e.g., `Foo`) and a controller or operator that handles it.
   - Show how CRDs can be versioned, migrated, and validated.
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: apiextensions.k8s.io/v1
   kind: CustomResourceDefinition
   metadata:
     name: foos.myorg.io
   spec:
     group: myorg.io
     names:
       kind: Foo
       plural: foos
       singular: foo
     scope: Namespaced
     versions:
       - name: v1
         served: true
         storage: true
         schema:
           openAPIV3Schema:
             type: object
             properties:
               spec:
                 type: object
                 properties:
                   replicas:
                     type: integer
   EOF
   ```
   - Write a small scenario where a CRD-based resource is created incorrectly, requiring troubleshooting.

2. **Ingress & Gateway API**:
   - Deploy an **Ingress controller** (e.g., NGINX Ingress) or **Gateway API** controller.
   - Create an Ingress or Gateway to route traffic to a simple backend.
   - Break or misconfigure the Ingress/Gateway to practice debugging.
   ```sh
   kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
   # or any other gateway / ingress solution
   ```

Example small Ingress:
```sh
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: my-app.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
EOF
```

3. **Structured Validation** of Ingress:
   ```sh
   kubectl get ingress -A
   kubectl describe ingress test-ingress
   curl -H 'Host: my-app.local' http://<nodeIP>/
   ```

---

## Time Management Strategies

- **Parallel Troubleshooting**: Launch logs/`journalctl` in separate terminals.
- **Prioritize** high-value tasks first if exam time is constrained.
- **Use Imperative Commands** for speed, then refine with YAML if needed.
- **Set Timers**: If a scenario demands 30 min, move on if you’re stuck.

---

## Command Reference & Imperative Approaches

- **Context Switching**:
  ```sh
  kubectl --context=my-cluster get pods
  ```
- **Imperative Resource Creation**:
  ```sh
  kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
  kubectl create configmap app-config --from-file=config.json
  kubectl label node k8s-worker-1 disk=ssd --overwrite
  ```
- **System Checks**:
  ```sh
  curl -k https://localhost:6443/healthz
  journalctl -xeu kubelet
  crictl ps
  ```

---

## Version-Specific Notes (1.28–1.32)

1. **PodSecurityPolicy** removed post-1.25.  
2. **EndpointSlices** are default.  
3. **In-tree Storage** replaced by CSI.  
4. **Gateway API** evolving; Ingress is still common.  
5. **CRD Upgrades** with `apiextensions.k8s.io/v1`.

---

