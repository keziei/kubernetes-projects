# Kezie Iroha 
# Kezie Iroha 
## Certified Kubernetes Administrator (CKA) Exam Preparation Scenarios part 2 (k8 1.32)
## If you find my notes useful for your study, then hit the star ⭐️


## Table of Contents
1. [Lab Setup & Prerequisites](#lab-setup--prerequisites)
2. [Scenario 1: Control Plane Component Failure](#scenario-1-control-plane-component-failure)
3. [Scenario 2: etcd Backup & Restoration](#scenario-2-etcd-backup--restoration)
4. [Scenario 3: Advanced Kubectl Operations](#scenario-3-advanced-kubectl-operations)
5. [Scenario 4: Complex Scheduling Challenges](#scenario-4-complex-scheduling-challenges)
6. [Scenario 5: Network Policy & Service Debugging](#scenario-5-network-policy--service-debugging)
7. [Scenario 6: Storage & PV/PVC Troubleshooting](#scenario-6-storage--pvpvc-troubleshooting)
8. [Scenario 7: Cluster Upgrade & Node Maintenance](#scenario-7-cluster-upgrade--node-maintenance)
9. [Scenario 8: Security, RBAC & PSA Controls](#scenario-8-security-rbac--psa-controls)
10. [Scenario 9: Container Runtime Debugging](#scenario-9-container-runtime-debugging)
11. [Scenario 10: Custom Resource Definitions & Controllers](#scenario-10-custom-resource-definitions--controllers)
12. [Time Management Strategies](#time-management-strategies)
13. [Command Reference & Imperative Approaches](#command-reference--imperative-approaches)
14. [Version-Specific Notes (K8s 1.32)](#version-specific-notes-k8s-132)

---

## Lab Setup & Prerequisites

### Recommended Lab Resources
- **Minimum Node Specs**: 2 CPUs and 4GB RAM per node (8GB recommended for the control plane)
- **Kubernetes Version**: 1.32 with containerd runtime
- **Recommended Lab Structure**:
  - 1 control plane node (`k8s-master`)
  - 2+ worker nodes (`k8s-worker-1`, `k8s-worker-2`)
- **Container Runtime**: Ensure you're using containerd (not Docker) as per current K8s requirements

### Pre-Scenario Checklist
1. **Confirm cluster is running** with `kubectl get nodes`
2. **Verify container runtime**: `kubectl get nodes -o wide` should show containerd
3. **Create etcd snapshots** to practice restore scenarios:
   ```sh
   sudo ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd/snapshot.db \
     --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key
   ```
4. **Pull common images** in advance to save time:
   ```sh
   sudo crictl pull nginx:1.25
   sudo crictl pull busybox:1.36
   ```

### CKA Domain Coverage Matrix

| Scenario                     | Cluster Architecture | Workloads & Scheduling | Services & Networking | Storage | Troubleshooting | Security | Maintenance |
|------------------------------|----------------------|------------------------|-----------------------|---------|-----------------|----------|------------|
| 1. Control Plane Failure     | ✓                    |                        |                       |         | ✓               |          |            |
| 2. etcd Backup/Restore       | ✓                    |                        |                       |         | ✓               |          | ✓          |
| 3. Advanced Kubectl          |                      | ✓                      | ✓                     | ✓       |                 |          |            |
| 4. Complex Scheduling        |                      | ✓                      |                       |         | ✓               |          |            |
| 5. Network & Services        |                      |                        | ✓                     |         | ✓               |          |            |
| 6. Storage Troubleshooting   |                      |                        |                       | ✓       | ✓               |          |            |
| 7. Cluster/Node Maintenance  | ✓                    |                        |                       |         | ✓               |          | ✓          |
| 8. Security & RBAC           |                      |                        |                       |         |                 | ✓        |            |
| 9. Container Runtime Debug   | ✓                    |                        |                       |         | ✓               |          |            |
| 10. CRDs & Controllers       | ✓                    | ✓                      |                       |         |                 |          |            |

### Recommended Scenario Order & Approx. Times
1. **Scenario 3**: ~20 min (start with kubectl skills)
2. **Scenario 1**: ~30 min
3. **Scenario 4**: ~20 min
4. **Scenario 5**: ~25 min
5. **Scenario 6**: ~20 min
6. **Scenario 8**: ~25 min
7. **Scenario 9**: ~20 min
8. **Scenario 2**: ~30 min
9. **Scenario 7**: ~35 min
10. **Scenario 10**: ~25 min

> **Time Management Tip**: Add ~5 minutes buffer per scenario for reading & planning. The actual CKA exam has 17 questions in 120 minutes, so practice completing each scenario efficiently.

---

## Scenario 1: Control Plane Component Failure
**Estimated Time**: ~30 minutes

### Overview
Diagnose and fix component failures in the control plane, specifically targeting kube-apiserver, kube-scheduler, and kube-controller-manager. This tests **cluster architecture** and **troubleshooting** skills.

### Fault Injection
1. **Break kube-apiserver** by modifying its manifest:
   ```sh
   vagrant ssh k8s-master
   sudo sed -i 's/--etcd-servers=https:\/\/127.0.0.1:2379/--etcd-servers=https:\/\/127.0.0.1:2377/' /etc/kubernetes/manifests/kube-apiserver.yaml
   sudo systemctl restart kubelet
   ```
2. **Break kube-scheduler** by moving its manifest:
   ```sh
   sudo mv /etc/kubernetes/manifests/kube-scheduler.yaml /etc/kubernetes/kube-scheduler.yaml
   sudo systemctl restart kubelet
   ```

### Diagnostic Steps
1. **Check Static Pod Status**:
   ```sh
   sudo crictl ps -a | grep kube
   ls -la /etc/kubernetes/manifests/
   ```
2. **Check Kubelet Status and Logs**:
   ```sh
   sudo systemctl status kubelet
   sudo journalctl -u kubelet -f
   ```
3. **Check Component Logs**:
   ```sh
   sudo crictl logs $(sudo crictl ps -a | grep kube-apiserver | awk '{print $1}')
   ```

### Resolution Steps
1. **Fix API Server etcd Connection**:
   ```sh
   sudo sed -i 's/--etcd-servers=https:\/\/127.0.0.1:2377/--etcd-servers=https:\/\/127.0.0.1:2379/' /etc/kubernetes/manifests/kube-apiserver.yaml
   sudo systemctl restart kubelet
   ```
2. **Restore Scheduler Manifest**:
   ```sh
   sudo mv /etc/kubernetes/kube-scheduler.yaml /etc/kubernetes/manifests/kube-scheduler.yaml
   sudo systemctl restart kubelet
   ```

### Validation
```sh
kubectl get pods -n kube-system
kubectl get componentstatuses # While deprecated, still useful for basic checks
kubectl cluster-info
```

**Potential Pitfall**: Looking only at pod status without checking crictl for lower-level container issues.

---

## Scenario 2: etcd Backup & Restoration
**Estimated Time**: ~30 minutes

### Overview
Practice critical etcd operations: backup, data corruption, and full restoration. This tests **cluster architecture**, **maintenance**, and **disaster recovery** skills.

### Initial Backup
```sh
# Create baseline snapshot
sudo ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Create some test resources to verify restore later
kubectl create ns restore-test
kubectl create deployment nginx --image=nginx -n restore-test
kubectl create configmap testdata --from-literal=key1=value1 -n restore-test
```

### Fault Injection
```sh
# Corrupt etcd data
sudo systemctl stop etcd
sudo rm -rf /var/lib/etcd/member/snap/db
sudo systemctl start etcd
```

### Resolution Steps
1. **Stop etcd and kubelet**:
   ```sh
   sudo systemctl stop etcd
   sudo systemctl stop kubelet
   ```
2. **Restore from snapshot**:
   ```sh
   sudo ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
     --data-dir=/var/lib/etcd-restored \
     --name=k8s-master \
     --initial-cluster=k8s-master=https://127.0.0.1:2380 \
     --initial-cluster-token=etcd-cluster-1 \
     --initial-advertise-peer-urls=https://127.0.0.1:2380
   ```
3. **Update etcd data directory**:
   ```sh
   sudo mv /var/lib/etcd /var/lib/etcd.bak
   sudo mv /var/lib/etcd-restored /var/lib/etcd
   sudo chown -R etcd:etcd /var/lib/etcd
   ```
4. **Restart etcd and kubelet**:
   ```sh
   sudo systemctl start etcd
   sudo systemctl start kubelet
   ```

### Validation
```sh
kubectl get ns | grep restore-test
kubectl get all -n restore-test
kubectl get configmap -n restore-test
```

**Potential Pitfall**: Forgetting to set correct permissions on the restored data directory.

---

## Scenario 3: Advanced Kubectl Operations
**Estimated Time**: ~20 minutes

### Overview
Master the imperative kubectl commands and resource generation techniques that are essential for the time-constrained CKA exam.

### Tasks
1. **Create a deployment with specific parameters without YAML editing**:
   ```sh
   kubectl create deployment web --image=nginx:1.25 --replicas=3 --port=80
   ```
2. **Scale and expose the deployment**:
   ```sh
   kubectl scale deployment web --replicas=5
   kubectl expose deployment web --port=80 --target-port=80 --type=NodePort
   ```
3. **Create a ConfigMap and Secret from literals and files**:
   ```sh
   # Create a test file
   echo "database_url=mysql://user:password@dbhost:3306/db" > db.properties
   
   kubectl create configmap app-config --from-literal=app.name=myapp --from-literal=app.version=1.0 --from-file=db.properties
   kubectl create secret generic app-secrets --from-literal=api.key=SECRET123 --from-literal=api.token=TOKEN456
   ```
4. **Create a Job and CronJob imperatively**:
   ```sh
   kubectl create job oneshot --image=busybox:1.36 -- sleep 30
   kubectl create cronjob hourly-job --image=busybox:1.36 --schedule="0 * * * *" -- echo "Job running"
   ```
5. **Use kubectl patch to update resources**:
   ```sh
   kubectl patch deployment web -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","resources":{"requests":{"memory":"64Mi","cpu":"250m"}}}]}}}}'
   ```
6. **Create a multi-container pod with a shared volume**:
   ```sh
   kubectl run multi-pod --image=nginx:1.25 --dry-run=client -o yaml > multi-pod.yaml
   # Edit to add sidecar container and volume
   # kubectl apply -f multi-pod.yaml
   ```

### Additional Kubectl Shortcuts
```sh
# Compact output formats
kubectl get pods -o wide
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

# Quick pod debugging
kubectl debug nginx-pod -it --image=busybox:1.36 --target=nginx

# Temporary pod for testing
kubectl run test --rm -it --image=busybox:1.36 -- sh

# Generating YAML templates
kubectl run nginx --image=nginx --port=80 --labels=app=web --dry-run=client -o yaml > pod.yaml
kubectl create service clusterip nginx --tcp=80:80 --dry-run=client -o yaml > svc.yaml
```

**Potential Pitfall**: Relying too much on YAML editing instead of mastering imperative commands.

---

## Scenario 4: Complex Scheduling Challenges
**Estimated Time**: ~20 minutes

### Overview
Practice advanced scheduling concepts including taints/tolerations, node affinity/anti-affinity, and resource constraints.

### Setup Environment
```sh
# Add labels to nodes
kubectl label nodes k8s-worker-1 disk=ssd zone=east
kubectl label nodes k8s-worker-2 disk=hdd zone=west

# Create taints on worker nodes
kubectl taint nodes k8s-worker-1 app=critical:NoSchedule
kubectl taint nodes k8s-worker-2 environment=test:NoSchedule
```

### Tasks
1. **Create a pod that only runs on nodes with SSD**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: ssd-pod
   spec:
     containers:
     - name: nginx
       image: nginx:1.25
     nodeSelector:
       disk: ssd
   EOF
   ```
2. **Create a pod with toleration for the critical app taint**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: critical-pod
   spec:
     containers:
     - name: nginx
       image: nginx:1.25
     tolerations:
     - key: "app"
       operator: "Equal"
       value: "critical"
       effect: "NoSchedule"
   EOF
   ```
3. **Create a pod with node anti-affinity**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: anti-affinity-pod
   spec:
     containers:
     - name: nginx
       image: nginx:1.25
     affinity:
       nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
           - matchExpressions:
             - key: zone
               operator: NotIn
               values:
               - west
   EOF
   ```
4. **Create a pod with specific resource requirements**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: resource-pod
   spec:
     containers:
     - name: nginx
       image: nginx:1.25
       resources:
         requests:
           memory: "128Mi"
           cpu: "500m"
         limits:
           memory: "256Mi"
           cpu: "1"
   EOF
   ```
5. **Create a pod with topology spread constraints**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: spread-pod
     labels:
       app: web
   spec:
     topologySpreadConstraints:
     - maxSkew: 1
       topologyKey: zone
       whenUnsatisfiable: DoNotSchedule
       labelSelector:
         matchLabels:
           app: web
     containers:
     - name: nginx
       image: nginx:1.25
   EOF
   ```

### Diagnostic Tasks
1. **Identify why a pod is pending**:
   ```sh
   kubectl describe pod <pod-name>
   ```
2. **Find which pod is using the most resources**:
   ```sh
   kubectl top pods --all-namespaces
   ```

### Validation
```sh
kubectl get pods -o wide
```

**Potential Pitfall**: Not understanding the difference between nodeSelector and node affinity.

---

## Scenario 5: Network Policy & Service Debugging
**Estimated Time**: ~25 minutes

### Overview
Test understanding of Kubernetes networking concepts including Services, NetworkPolicies, and connectivity troubleshooting.

### Setup
```sh
# Create test namespaces
kubectl create namespace frontend
kubectl create namespace backend
kubectl create namespace database

# Deploy some pods
kubectl run web --image=nginx -n frontend --labels=app=web,tier=frontend
kubectl run api --image=nginx -n backend --labels=app=api,tier=backend
kubectl run db --image=nginx -n database --labels=app=db,tier=database

# Create services
kubectl expose pod web -n frontend --port=80 --name=web-svc
kubectl expose pod api -n backend --port=80 --name=api-svc
kubectl expose pod db -n database --port=80 --name=db-svc
```

### Fault Injection
1. **Add restrictive NetworkPolicies**:
   ```sh
   # Frontend can only talk to backend
   cat <<EOF | kubectl apply -f -
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: frontend-policy
     namespace: frontend
   spec:
     podSelector:
       matchLabels:
         tier: frontend
     policyTypes:
     - Egress
     egress:
     - to:
       - namespaceSelector:
           matchLabels:
             kubernetes.io/metadata.name: backend
         podSelector:
           matchLabels:
             tier: backend
   EOF

   # Backend can only talk to database
   cat <<EOF | kubectl apply -f -
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: backend-policy
     namespace: backend
   spec:
     podSelector:
       matchLabels:
         tier: backend
     policyTypes:
     - Ingress
     - Egress
     ingress:
     - from:
       - namespaceSelector:
           matchLabels:
             kubernetes.io/metadata.name: frontend
         podSelector:
           matchLabels:
             tier: frontend
     egress:
     - to:
       - namespaceSelector:
           matchLabels:
             kubernetes.io/metadata.name: database
         podSelector:
           matchLabels:
             tier: database
   EOF

   # Database can only receive from backend
   cat <<EOF | kubectl apply -f -
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: database-policy
     namespace: database
   spec:
     podSelector:
       matchLabels:
         tier: database
     policyTypes:
     - Ingress
     ingress:
     - from:
       - namespaceSelector:
           matchLabels:
             kubernetes.io/metadata.name: backend
         podSelector:
           matchLabels:
             tier: backend
   EOF
   ```

2. **Break the API service**:
   ```sh
   kubectl patch svc api-svc -n backend -p '{"spec":{"selector":{"app":"wrong-app"}}}'
   ```

### Diagnostic Tasks
1. **Test connectivity between pods**:
   ```sh
   # Create a test pod
   kubectl run tester --image=busybox:1.36 -n frontend --rm -it -- /bin/sh
   
   # Try to reach different services
   wget -qO- http://web-svc.frontend
   wget -qO- http://api-svc.backend
   wget -qO- http://db-svc.database
   ```
2. **Examine service selectors**:
   ```sh
   kubectl get svc api-svc -n backend -o yaml
   ```
3. **Check NetworkPolicy configuration**:
   ```sh
   kubectl describe networkpolicy -n backend
   ```

### Resolution
1. **Fix API service selector**:
   ```sh
   kubectl patch svc api-svc -n backend -p '{"spec":{"selector":{"app":"api"}}}'
   ```
2. **Modify NetworkPolicy if needed**:
   ```sh
   kubectl edit networkpolicy backend-policy -n backend
   ```

### Validation
```sh
kubectl run tester --image=busybox:1.36 -n frontend --rm -it -- wget -qO- http://api-svc.backend
```

**Potential Pitfall**: Not understanding how NetworkPolicies affect traffic between namespaces.

---

## Scenario 6: Storage & PV/PVC Troubleshooting
**Estimated Time**: ~20 minutes

### Overview
Test understanding of Kubernetes storage concepts including PersistentVolumes, StorageClasses, and CSI drivers.

### Setup
```sh
# Create StorageClass
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF

# Create PersistentVolume
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-worker-1
EOF

# Create PVC that should be bound
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
EOF
```

### Fault Injection
1. **Create a directory on worker node for local storage**:
   ```sh
   vagrant ssh k8s-worker-1
   sudo mkdir -p /mnt/data
   sudo chmod 777 /mnt/data
   ```
2. **Create a pod that will use this PVC but with a mistake**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: pv-pod
   spec:
     containers:
     - name: task-pv-container
       image: nginx:1.25
       volumeMounts:
       - mountPath: /usr/share/nginx/html
         name: pv-storage
     volumes:
     - name: pv-storage
       persistentVolumeClaim:
         claimName: wrong-local-claim
   EOF
   ```
3. **Create a misconfigured StorageClass with a nonexistent provisioner**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: broken-csi
   provisioner: example.com/doesnt-exist
   EOF
   ```
4. **Create a PVC using the broken StorageClass**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: broken-claim
   spec:
     accessModes:
     - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
     storageClassName: broken-csi
   EOF
   ```

### Diagnostic Tasks
1. **Check PV/PVC status**:
   ```sh
   kubectl get pv,pvc
   kubectl describe pvc local-claim
   kubectl describe pvc broken-claim
   ```
2. **Check pod events**:
   ```sh
   kubectl describe pod pv-pod
   ```
3. **Check StorageClass configuration**:
   ```sh
   kubectl get sc
   kubectl describe sc broken-csi
   ```

### Resolution
1. **Fix pod volume reference**:
   ```sh
   kubectl delete pod pv-pod
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: pv-pod
   spec:
     containers:
     - name: task-pv-container
       image: nginx:1.25
       volumeMounts:
       - mountPath: /usr/share/nginx/html
         name: pv-storage
     volumes:
     - name: pv-storage
       persistentVolumeClaim:
         claimName: local-claim
   EOF
   ```
2. **Create a pod with the correct PVC and add data to verify persistence**:
   ```sh
   kubectl exec -it pv-pod -- sh -c "echo 'Hello from PV storage' > /usr/share/nginx/html/index.html"
   ```

### Validation
```sh
kubectl get pv,pvc
kubectl get pod pv-pod
kubectl exec -it pv-pod -- cat /usr/share/nginx/html/index.html
```

**Potential Pitfall**: Not understanding volumeBindingMode in StorageClass and why some PVCs remain pending.

---

## Scenario 7: Cluster Upgrade & Node Maintenance
**Estimated Time**: ~35 minutes

### Overview
Practice the process of upgrading a Kubernetes cluster and performing node maintenance without disrupting application availability.

### Setup Application
```sh
kubectl create deployment nginx --image=nginx:1.25 --replicas=4
kubectl expose deployment nginx --port=80
```

### Tasks
1. **Simulate a node upgrade process**:
   - **Cordon the node**:
     ```sh
     kubectl cordon k8s-worker-1
     ```
   - **Drain the node safely**:
     ```sh
     kubectl drain k8s-worker-1 --ignore-daemonsets --delete-emptydir-data
     ```
   - **Simulate maintenance work**:
     ```sh
     vagrant ssh k8s-worker-1
     sudo dnf update
     # Simulate upgrade work
     sudo systemctl stop kubelet
     sleep 30
     sudo systemctl start kubelet
     exit
     ```
   - **Uncordon the node**:
     ```sh
     kubectl uncordon k8s-worker-1
     ```

2. **Upgrade the control plane (simulation)**:
   ```sh
   vagrant ssh k8s-master
   
   # Simulate upgrading kubeadm
   # Install dnf-plugins-core if not already installed
    sudo dnf install -y dnf-plugins-core

  # Lock kubeadm, kubelet, and kubectl versions (equivalent to apt-mark hold)
   sudo dnf versionlock add kubeadm kubelet kubectl
   sudo dnf update
   
   # In a real upgrade:
   # sudo dnf install -y kubeadm=1.32.x-00
   # sudo kubeadm upgrade plan
   # sudo kubeadm upgrade apply v1.32.x
   
   # Simulate upgrade
   sudo systemctl stop kubelet
   sleep 10
   sudo systemctl start kubelet
   
   # Then upgrade kubelet and kubectl
   # sudo dnf versionlock delete kubeadm kubelet kubectl
   # sudo dnf install -y kubelet=1.32.x-00 kubectl=1.32.x-00
   # sudo systemctl restart kubelet
   ```

3. **Upgrade worker nodes (simulation for k8s-worker-2)**:
   ```sh
   kubectl drain k8s-worker-2 --ignore-daemonsets
   
   vagrant ssh k8s-worker-2
   
   # In a real upgrade:
   # sudo dnf versionlock add kubelet
   # sudo dnf update
   # sudo dnf install -y kubeadm=1.32.x-00
   # sudo kubeadm upgrade node
   # sudo dnf versionlock delete kubelet
   # sudo dnf install -y kubelet=1.32.x-00
   # sudo systemctl restart kubelet
   
   # Simulate upgrade
   sudo systemctl stop kubelet
   sleep 10
   sudo systemctl start kubelet
   
   exit
   
   kubectl uncordon k8s-worker-2
   ```

### Troubleshooting Tasks
1. **Handle pod eviction failure**:
   ```sh
   # If a pod is not getting evicted, force delete it
   kubectl delete pod <pod-name> --force --grace-period=0
   ```
2. **Check for cluster health during upgrade**:
   ```sh
   kubectl get nodes
   kubectl cluster-info
   ```

### Validation
```sh
kubectl get nodes
kubectl version --short
```

**Potential Pitfall**: Not using the --ignore-daemonsets flag during drain, causing the operation to fail.

---

## Scenario 8: Security, RBAC & PSA Controls (continued)
**Estimated Time**: ~25 minutes

### Creating Test Pods in PSA Namespaces (continued)

```sh
# This should fail in the restricted namespace
cat <<EOF | kubectl apply -n restricted -f -
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    securityContext:
      privileged: true
EOF

# This should work in the restricted namespace
cat <<EOF | kubectl apply -n restricted -f -
apiVersion: v1
kind: Pod
metadata:
  name: restricted-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    securityContext:
      allowPrivilegeEscalation: false
      runAsNonRoot: true
      runAsUser: 1000
      capabilities:
        drop:
          - ALL
EOF

# This should work in the privileged namespace
cat <<EOF | kubectl apply -n privileged -f -
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    securityContext:
      privileged: true
EOF
```

4. **Test RBAC Authorization**:
   ```sh
   # Create cluster admin user (for demonstration only - not recommended in practice)
   kubectl create serviceaccount cluster-admin-sa
   kubectl create clusterrolebinding cluster-admin-binding \
     --clusterrole=cluster-admin \
     --serviceaccount=default:cluster-admin-sa
   
   # Verify permissions
   kubectl auth can-i '*' '*' --as=system:serviceaccount:default:cluster-admin-sa
   ```

5. **Create Secret and control access**:
   ```sh
   kubectl create secret generic api-creds \
     --from-literal=username=admin \
     --from-literal=password=supersecret \
     -n app-team1
   
   # Create role that allows access to this specific secret
   cat <<EOF | kubectl apply -f -
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     namespace: app-team1
     name: secret-reader
   rules:
   - apiGroups: [""]
     resources: ["secrets"]
     resourceNames: ["api-creds"]
     verbs: ["get"]
   EOF
   
   # Bind to service account
   kubectl create rolebinding app-sa-secret \
     --role=secret-reader \
     --serviceaccount=app-team1:app-sa \
     -n app-team1
   ```

### Diagnostic Tasks
1. **Test authorization for service accounts**:
   ```sh
   kubectl auth can-i list pods --as=system:serviceaccount:app-team1:app-sa -n app-team1
   kubectl auth can-i delete pods --as=system:serviceaccount:app-team1:app-sa -n app-team1
   kubectl auth can-i get secrets/api-creds --as=system:serviceaccount:app-team1:app-sa -n app-team1
   ```
2. **Verify PSA controls**:
   ```sh
   kubectl get ns restricted --show-labels
   ```

### Validation
```sh
kubectl get pod -n restricted
kubectl get pod -n privileged
kubectl get pod -n app-team1

# Examine failure events
kubectl get events -n restricted | grep -i error
```

**Potential Pitfall**: Not understanding how Pod Security Admission works at the namespace level or configuring roles with too broad permissions.

---

## Scenario 9: Container Runtime Debugging
**Estimated Time**: ~20 minutes

### Overview
Develop skills to diagnose and troubleshoot container runtime (containerd) issues, which is critical for CKA exam success in Kubernetes 1.32.

### Setup
```sh
# Deploy a pod for testing
kubectl run runtime-test --image=nginx:1.25
```

### Fault Injection
1. **Break containerd configuration on a worker node**:
   ```sh
   vagrant ssh k8s-worker-1
   
   # Back up containerd config
   sudo cp /etc/containerd/config.toml /etc/containerd/config.toml.bak
   
   # Modify config (introduce a syntax error)
   sudo bash -c 'echo "version = 2 # Incorrect setting" >> /etc/containerd/config.toml'
   
   # Restart containerd
   sudo systemctl restart containerd
   ```

2. **Deploy a pod with an incorrect image**:
   ```sh
   kubectl run broken-pod --image=nginx:nonexistent
   ```

### Diagnostic Tasks
1. **Examine containerd status**:
   ```sh
   sudo systemctl status containerd
   ```
2. **Check containerd logs**:
   ```sh
   sudo journalctl -u containerd -n 100
   ```
3. **Use crictl to inspect containers and images**:
   ```sh
   # List pods
   sudo crictl pods
   
   # List containers
   sudo crictl ps -a
   
   # List images
   sudo crictl images
   
   # Get pod details
   sudo crictl inspectp <pod-id>
   
   # Check container logs
   sudo crictl logs <container-id>
   ```
4. **Examine kubelet status**:
   ```sh
   sudo systemctl status kubelet
   ```
5. **Check kubelet logs**:
   ```sh
   sudo journalctl -u kubelet -n 100
   ```

### Resolution
1. **Fix containerd configuration**:
   ```sh
   sudo cp /etc/containerd/config.toml.bak /etc/containerd/config.toml
   sudo systemctl restart containerd
   ```
2. **Pull the correct image manually**:
   ```sh
   sudo crictl pull nginx:1.25
   ```
3. **Delete and recreate the broken pod**:
   ```sh
   kubectl delete pod broken-pod
   kubectl run broken-pod --image=nginx:1.25
   ```

### Additional containerd Commands
```sh
# Check container info
sudo crictl inspect <container-id>

# Execute command in container
sudo crictl exec -it <container-id> sh

# View container stats
sudo crictl stats

# Remove stopped containers
sudo crictl rm <container-id>

# Remove unused images
sudo crictl rmi --prune
```

### Validation
```sh
# Check pod status
kubectl get pods -o wide

# Verify containerd is running
sudo systemctl is-active containerd

# Check if images are properly pulled
sudo crictl images | grep nginx
```

**Potential Pitfall**: Not understanding how to use crictl commands to debug container issues or not checking both containerd and kubelet logs.

---

## Scenario 10: Custom Resource Definitions & Controllers
**Estimated Time**: ~25 minutes

### Overview
Practice working with Custom Resource Definitions (CRDs) and understanding how controllers use them, which is increasingly important in the Kubernetes ecosystem and CKA exam.

### Setup
1. **Create a simple CRD**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: apiextensions.k8s.io/v1
   kind: CustomResourceDefinition
   metadata:
     name: backups.cka.example.com
   spec:
     group: cka.example.com
     names:
       kind: Backup
       listKind: BackupList
       plural: backups
       singular: backup
       shortNames:
       - bkp
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
                 source:
                   type: string
                 schedule:
                   type: string
                 retention:
                   type: integer
                   minimum: 1
               required: ["source"]
             status:
               type: object
               properties:
                 phase:
                   type: string
                 lastBackupTime:
                   type: string
       additionalPrinterColumns:
       - name: Source
         type: string
         jsonPath: .spec.source
       - name: Schedule
         type: string
         jsonPath: .spec.schedule
       - name: Status
         type: string
         jsonPath: .status.phase
   EOF
   ```

2. **Create some example CR instances**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: cka.example.com/v1
   kind: Backup
   metadata:
     name: daily-etcd-backup
   spec:
     source: etcd
     schedule: "0 0 * * *"
     retention: 7
   EOF
   
   cat <<EOF | kubectl apply -f -
   apiVersion: cka.example.com/v1
   kind: Backup
   metadata:
     name: weekly-app-backup
   spec:
     source: app-data
     schedule: "0 0 * * 0"
     retention: 4
   EOF
   ```

3. **Create a simple controller (for demonstration purposes)**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: backup-controller
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     name: backup-controller
   rules:
   - apiGroups: ["cka.example.com"]
     resources: ["backups"]
     verbs: ["get", "list", "watch", "update", "patch"]
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:
     name: backup-controller
   subjects:
   - kind: ServiceAccount
     name: backup-controller
     namespace: default
   roleRef:
     kind: ClusterRole
     name: backup-controller
     apiGroup: rbac.authorization.k8s.io
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: backup-controller
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: backup-controller
     template:
       metadata:
         labels:
           app: backup-controller
       spec:
         serviceAccountName: backup-controller
         containers:
         - name: controller
           image: busybox:1.36
           command: ["sh", "-c", "while true; do echo 'Controller would process backups here'; sleep 30; done"]
   EOF
   ```

### Tasks
1. **Add a status update to a CR**:
   ```sh
   kubectl patch backup daily-etcd-backup --type=merge -p '{"status":{"phase":"Completed","lastBackupTime":"2025-03-04T10:00:00Z"}}'
   ```

2. **Query the custom resources**:
   ```sh
   kubectl get backups
   kubectl get bkp
   kubectl describe backup daily-etcd-backup
   ```

3. **Create an invalid CR (validation should fail)**:
   ```sh
   cat <<EOF | kubectl apply -f -
   apiVersion: cka.example.com/v1
   kind: Backup
   metadata:
     name: invalid-backup
   spec:
     schedule: "0 0 * * *"
     retention: 0
   EOF
   ```

### Diagnostic Tasks
1. **Check the CRD definition**:
   ```sh
   kubectl describe crd backups.cka.example.com
   ```
2. **Check controller logs**:
   ```sh
   kubectl logs -l app=backup-controller
   ```
3. **Check validation errors**:
   ```sh
   kubectl get events | grep invalid-backup
   ```

### Validation
```sh
kubectl get backups -o wide
kubectl api-resources | grep backup
```

**Potential Pitfall**: Not understanding CRD validation schema or not knowing how to correctly update CR status.

---

## Time Management Strategies

### Exam-Specific Approaches
1. **Rapid Assessment**: Scan all questions at the beginning to identify:
   - Quick wins (easy questions to answer first)
   - High-point questions (prioritize these next)
   - Complex questions (leave these for last)

2. **Command Efficiency**:
   - Use kubectl short aliases: `k` for `kubectl`
   - Set up auto-completion: `source <(kubectl completion bash)`
   - Create context-specific aliases: `alias kn='kubectl -n kube-system'`
   - Use the `-o` flag efficiently: `kubectl get pods -o wide`, `kubectl get pods -o yaml`

3. **Documentation Shortcuts**:
   - Bookmark key pages in the Kubernetes docs
   - Use `kubectl explain <resource>` for quick reference
   - Leverage `--dry-run=client -o yaml` to generate templates

4. **Parallel Troubleshooting**:
   - Use multiple terminal tabs/windows
   - Run long processes (like etcd backup) in background: `&`
   - Use `watch` for monitoring: `watch kubectl get pods`

5. **Triage Strategy**:
   - If stuck for >5 minutes, flag the question and move on
   - Return to difficult questions after completing easier ones
   - Always leave the last 10 minutes for review

### Practice Techniques
1. **Timed Drills**: Set a timer for each scenario and force yourself to move on when time expires
2. **Question Batching**: Do 3-4 scenarios in a row without breaks to build stamina
3. **Failure Scenarios**: Deliberately create issues in your cluster and then fix them
4. **Command Mastery**: Create flashcards for essential kubectl commands

---

## Command Reference & Imperative Approaches

### Essential kubectl Commands

#### Resource Creation
```sh
# Create resources imperatively
kubectl create deployment nginx --image=nginx:1.25 --replicas=3
kubectl create service clusterip nginx --tcp=80:80
kubectl expose deployment nginx --port=80 --target-port=80
kubectl create configmap app-config --from-file=config.properties
kubectl create secret generic db-creds --from-literal=username=admin --from-literal=password=secret

# Generate YAML templates
kubectl create deployment web --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl create job test-job --image=busybox --dry-run=client -o yaml -- echo "Hello" > job.yaml
```

#### Resource Management
```sh
# Patching resources
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'
kubectl patch svc nginx -p '{"spec":{"type":"NodePort"}}'

# Scaling resources
kubectl scale deployment nginx --replicas=3

# Rolling updates
kubectl set image deployment/nginx nginx=nginx:1.25
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx

# Labeling resources
kubectl label pods nginx-pod tier=frontend
kubectl label nodes k8s-worker-1 disk=ssd --overwrite
```

#### Troubleshooting
```sh
# Logs and events
kubectl logs pod/nginx
kubectl logs -f deployment/nginx
kubectl get events --sort-by=.metadata.creationTimestamp

# Describing resources
kubectl describe pod nginx
kubectl describe node k8s-worker-1

# Resource usage
kubectl top pods
kubectl top nodes

# Debugging pods
kubectl debug pod/nginx -it --image=busybox:1.36
kubectl attach pod/nginx -c nginx -i -t
```

#### Node Management
```sh
# Node operations
kubectl cordon k8s-worker-1
kubectl drain k8s-worker-1 --ignore-daemonsets
kubectl uncordon k8s-worker-1
```

### containerd/crictl Commands
```sh
# List pods, containers, images
sudo crictl pods
sudo crictl ps -a
sudo crictl images

# Container operations
sudo crictl pull nginx:1.25
sudo crictl logs <container-id>
sudo crictl exec -it <container-id> sh
sudo crictl inspect <container-id>

# Cleanup
sudo crictl rmi --prune
sudo crictl rm <container-id>
```

### Cluster Management
```sh
# etcd operations
sudo ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd/snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Certificate operations
sudo kubeadm certs check-expiration
sudo kubeadm certs renew all

# Static pod management
ls -la /etc/kubernetes/manifests/
```

---

## Version-Specific Notes (K8s 1.32)

1. **Container Runtime**: Docker is no longer supported; containerd is the standard.
   - All Docker CLI commands should be replaced with crictl equivalents
   - Example: `docker ps` → `crictl ps`

2. **Pod Security**:
   - PodSecurityPolicy (PSP) is completely removed
   - Pod Security Admission (PSA) is the standard replacement
   - PSA operates at namespace level with three profiles: privileged, baseline, restricted

3. **Storage**:
   - In-tree volume plugins are being replaced by CSI drivers
   - Volume expansion is a standard feature
   - CSI drivers provide more advanced features like snapshots

4. **Networking**:
   - EndpointSlices have replaced Endpoints for better scaling
   - IPv4/IPv6 dual-stack is standard
   - Gateway API is gaining functionality compared to Ingress

5. **API Changes**:
   - CRDs are now all apiextensions.k8s.io/v1
   - Beta APIs are increasingly being removed in favor of stable APIs
   - Service account token volumes should use projected volumes

6. **Kubelet**:
   - cgroupv2 is the preferred cgroup driver
   - SeccompDefault feature gate is stable
   - MemoryQoS feature is standard

7. **Other Features**:
   - OpenAPI v3 schema validation is standard
   - ValidatingAdmissionPolicy is stable (replacing webhooks in some cases)
   - StructuredAuthorizationConfiguration is available (Beta)

Always check the Kubernetes documentation for the most up-to-date information, as K8s 1.32 features may continue to evolve.

---

## Final Tips for CKA Success

1. **Master imperative commands** - they save precious time during the exam
2. **Practice using vi/vim editor shortcuts** - you will need to edit YAML files
3. **Get comfortable with containerd debugging** - crictl is essential
4. **Know your etcd operations cold** - backup/restore is a common task
5. **Focus on practical troubleshooting** - in real clusters, not just theory
6. **Time yourself religiously** - the exam is time-constrained
7. **Create cluster from scratch multiple times** - understand how components interact
8. **Memorize key paths** - /etc/kubernetes/manifests, /var/lib/etcd, etc.
9. **Practice with different CNI plugins** - understand networking fundamentals
10. **Embrace failure** - intentionally break your cluster to learn fixing it