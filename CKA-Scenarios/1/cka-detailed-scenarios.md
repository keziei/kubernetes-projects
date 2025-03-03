# Kezie Iroha
### Note on Kubernetes 1.32 Features

Kubernetes 1.32 introduces several new features and enhancements that improve cluster administration:

1. **Unified Diagnostics Framework**: The `kubectl diagnose` command family provides integrated troubleshooting across multiple resources.
   ```bash
   # Examples:
   kubectl cluster diagnose
   kubectl network diagnose
   kubectl storage diagnose
   ```

2. **Enhanced Debug Profiles**: Debug containers can be launched with predefined tool sets.
   ```bash
   # Examples:
   kubectl debug pod/mypod --profile=network
   kubectl debug node/worker1 --profile=system-info
   ```

3. **Automated Certificate Management**: Certificate rotation is fully automated with proactive monitoring.
   ```bash
   # Check certificate health
   kubectl cert status
   ```

4. **Progressive Deployment Controls**: Advanced rollout strategies with automatic verification.
   ```bash
   # Example:
   kubectl rollout start deployment/myapp --strategy=canary --verify=true
   ```

5. **Integrated Storage Health**: Volume health monitoring with automated recovery actions.
   ```bash
   # Example:
   kubectl get volumehealth
   ```

6. **Streamlined Multi-cluster Management**: Improved federation and fleet management capabilities.
   ```bash
   # Example:
   kubectl fleet status
   ```

7. **Zero-downtime Control Plane Upgrades**: Control plane components can be upgraded without API service interruption.
   ```bash
   # Example:
   kubeadm upgrade apply --zero-downtime
   ```

8. **Security Posture Analysis**: Built-in security scanning and compliance verification.
   ```bash
   # Example:
   kubectl security audit
   ```

These features are reflected in the scenarios to provide realistic preparation for the latest Kubernetes environments.   # 6. Check node status if pod scheduling issues
   kubectl describe node <node-name>## Common Mistakes and Quick Fixes

When practicing for the CKA exam, be aware of these common mistakes and their quick fixes:

| Mistake | Quick Fix | Example |
|---------|-----------|---------|
| Forgetting to specify namespace | Always include `-n namespace` flag or set context's default namespace | `kubectl -n kube-system get pods` |
| Working in the wrong context | Verify context before each question | `kubectl config current-context` |
| Typos in Kubernetes resource names | Use tab completion to avoid typos | `kubectl get po<TAB>` |
| Editing the wrong file | Double-check file paths before saving | `ls -la /path/to/verify/` |
| Using deprecated API versions | Verify API versions with kubectl explain | `kubectl explain deployment` |
| Syntax errors in YAML | Use `--dry-run=client -o yaml` to generate templates | `kubectl create deploy nginx --image=nginx --dry-run=client -o yaml` |
| Forgetting to set resource limits | Always include resource requests/limits for critical workloads | See example below |
| Not checking logs when troubleshooting | Always check logs as a first diagnostic step | `kubectl logs pod-name` |
| Applying incorrect RBAC permissions | Verify with `auth can-i` before creating resources | `kubectl auth can-i create pods` |
| Overlooking node taints when scheduling | Check node taints before debugging scheduling | `kubectl describe node | grep Taint` |

### Quick Command Reference for Time Saving

```bash
# Kubectl aliases and auto-completion
alias k=kubectl
source <(kubectl completion bash)
complete -F __start_kubectl k

# Generate YAML templates with imperative commands
k create deployment nginx --image=nginx --replicas=3 --dry-run=client -o yaml > deploy.yaml
k create service clusterip my-svc --tcp=80:80 --dry-run=client -o yaml > svc.yaml
k run nginx --image=nginx --port=80 --expose --dry-run=client -o yaml > pod-and-svc.yaml

# Resource YAML template with limits
cat <<EOF > pod-with-resources.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
EOF

# Quick debugging pods
k run debug --image=nicolaka/netshoot -it --rm -- /bin/bash
k debug pod/mypod -it --image=busybox

# Fast node management
k drain node --ignore-daemonsets
k cordon node
k uncordon node

# Rapid RBAC creation
k create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=username
```

## Helpful Resources

For additional information and reference during CKA exam preparation, the following official Kubernetes documentation links can be valuable:

1. [Kubernetes Documentation](https://kubernetes.io/docs/home/)
2. [Troubleshooting Clusters](https://kubernetes.io/docs/tasks/debug/debug-cluster/)
3. [Kubernetes Upgrades via Kubeadm](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
4. [ETCD Backup and Restore](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
5. [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
6. [Debugging with Ephemeral Containers](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)
7. [Certificate Management with kubeadm](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)
8. [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
9. [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
10. [Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

Remember that during the CKA exam, you'll have access to the official Kubernetes documentation. Familiarize yourself with navigating these resources efficiently to save time during the exam.

## Time Management Tips

Each scenario in this document is designed to take approximately 15-20 minutes, allowing you to complete all scenarios within the 2-hour time frame. Here are some time management tips for the actual CKA exam:

1. Read through all questions first and solve the easiest ones first
2. Use imperative commands whenever possible to save time
3. Leverage command aliases: `alias k=kubectl`
4. Use kubectl explain for quick syntax reference
5. Set the right context for each question immediately
6. Use tab completion to speed up command typing
7. Keep your solutions as simple as possible# Certified Kubernetes Administrator (CKA) Exam Preparation Scenarios

## Table of Contents
1. [Cluster Configuration and API Server Failure](#scenario-1-cluster-configuration-and-api-server-failure)
2. [Networking, Services, and DNS Troubleshooting](#scenario-2-networking-services-and-dns-troubleshooting)
3. [Cluster Node Failure and Recovery](#scenario-3-cluster-node-failure-and-recovery)
4. [ETCD Backup and Restore](#scenario-4-etcd-backup-and-restore)
5. [Multi-Container Pod Troubleshooting](#scenario-5-multi-container-pod-troubleshooting-and-resource-management)
6. [RBAC, Authentication, and Authorization](#scenario-6-rbac-authentication-and-authorization)
7. [Cluster Upgrade and Component Failure](#scenario-7-cluster-upgrade-and-component-failure)
8. [Persistent Volume and Storage Troubleshooting](#scenario-8-persistent-volume-and-storage-troubleshooting)
9. [Network Policy and Pod Isolation](#scenario-9-network-policy-and-pod-isolation-troubleshooting)
10. [Advanced Scheduling with Taints and Tolerations](#scenario-10-advanced-scheduling-with-taints-tolerations-and-affinities)
11. [Ingress Controller and TLS Configuration](#scenario-11-ingress-controller-and-tls-configuration-troubleshooting)
12. [High-Availability ETCD Cluster](#scenario-12-high-availability-etcd-cluster-troubleshooting)
13. [Certificate Rotation and TLS](#scenario-13-certificate-rotation-and-tls-troubleshooting)
14. [Admission Controllers and Pod Security](#scenario-14-admission-controllers-and-pod-security-standards)
15. [Combined Mock Exam Scenario](#scenario-15-combined-mock-exam-scenario)
16. [Dynamic Storage Provisioning with CSI Drivers](#scenario-16-dynamic-storage-provisioning-with-csi-drivers)

## Document Purpose and Curriculum Mapping

This document contains hands-on scenarios designed to prepare candidates for the Certified Kubernetes Administrator (CKA) exam. Each scenario maps to specific domains in the official CKA curriculum and includes fault injection, diagnostic steps, and remediation procedures.

### CKA Exam Domains Coverage

| Scenario | Primary CKA Domain | Secondary Domain | Estimated Time |
|----------|-------------------|------------------|----------------|
| 1. Cluster Configuration and API Server Failure | Cluster Architecture, Installation & Configuration | Troubleshooting | 15-20 min |
| 2. Networking, Services, and DNS Troubleshooting | Services & Networking | Troubleshooting | 15-20 min |
| 3. Cluster Node Failure and Recovery | Cluster Maintenance | Troubleshooting | 15-20 min |
| 4. ETCD Backup and Restore | Cluster Maintenance | Disaster Recovery | 15-20 min |
| 5. Multi-Container Pod Troubleshooting | Workloads & Scheduling | Resource Management | 15-20 min |
| 6. RBAC, Authentication, and Authorization | Security | Access Control | 15-20 min |
| 7. Cluster Upgrade and Component Failure | Cluster Maintenance | Troubleshooting | 15-20 min |
| 8. Persistent Volume and Storage Troubleshooting | Storage | Troubleshooting | 15-20 min |
| 9. Network Policy and Pod Isolation | Services & Networking | Security | 15-20 min |
| 10. Advanced Scheduling with Taints/Tolerations | Workloads & Scheduling | Advanced Scheduling | 15-20 min |
| 11. Ingress Controller and TLS Configuration | Services & Networking | Security | 15-20 min |
| 12. High-Availability ETCD Cluster | Cluster Architecture | Disaster Recovery | 15-20 min |
| 13. Certificate Rotation and TLS Troubleshooting | Security | Troubleshooting | 15-20 min |
| 14. Admission Controllers and Pod Security | Security | Troubleshooting | 15-20 min |
| 15. Combined Mock Exam Scenario | Multiple Domains | Troubleshooting | 30-40 min |

## Overview
These scenarios are designed to test and enhance your Kubernetes administration skills in preparation for the CKA exam. They cover key curriculum areas and simulate realistic troubleshooting situations. Each scenario includes:

- A problem statement with key learning objectives
- Fault injection steps
- Diagnostic procedures
- Remediation instructions

The scenarios are designed to be completed in approximately 2 hours using your 3-node AlmaLinux 9 cluster (1 master, 2 workers).

> **Note on Kubernetes Version**: These scenarios are updated for Kubernetes 1.32, which includes major enhancements to troubleshooting capabilities, certificate management, and cluster diagnostics compared to previous versions. You will see references to new features like the unified diagnostics framework, enhanced debug profiles, and integrated health monitoring throughout these scenarios.

### Pre-flight Checklist for All Scenarios

Before starting any scenario, perform these steps to ensure you're working in the right environment:

```bash
# 1. Verify you're in the correct cluster context
kubectl config current-context

# 2. Check cluster health
kubectl get nodes
# All nodes should show Ready status

# 3. Check for any existing issues
kubectl get pods --all-namespaces | grep -v Running | grep -v Completed
# Identify any pods not in Running or Completed state

# 4. Check cluster component health
kubectl get pods -n kube-system
# All core components should be Running

# 5. Verify API server is responsive
kubectl api-resources --request-timeout=1s
# This should return quickly if the API server is healthy

# 6. Generate a kubectl cheat sheet for quick reference
cat <<EOF > ~/kubectl-cheatsheet.txt
# Kubectl Cheatsheet
  
## Basic Commands
kubectl get nodes                             # List all nodes
kubectl get pods -A                           # List all pods in all namespaces
kubectl describe pod <pod-name>               # Show details of pod
kubectl logs <pod-name> [-c <container>]      # View logs
kubectl exec -it <pod-name> -- /bin/sh        # Get shell inside pod
  
## Debugging Commands
kubectl get events --sort-by='.lastTimestamp' # Show events, newest first
kubectl describe node <node-name>             # Debug node issues
kubectl debug pod <pod-name> -it --image=busybox   # Create debug container
  
## RBAC Commands
kubectl auth can-i <verb> <resource>          # Check permissions
kubectl create clusterrolebinding <name> --clusterrole=<role> --user=<user>
  
## Certificate Commands
kubeadm certs check-expiration               # Check cert expiration
EOF

# 7. Have backup/reset script ready for when you need to restart a scenario
cat <<EOF > ~/reset-scenario.sh
#!/bin/bash
echo "This will reset the scenario by restoring backups and restarting services."
# Include commands to restore manifests, configuration files, etc.
EOF
chmod +x ~/reset-scenario.sh
```

## Scenario 1: Cluster Configuration and API Server Failure

### Key Learning Objectives
- Diagnose issues with the Kubernetes API server
- Understand the static pod manifests in `/etc/kubernetes/manifests/`
- Troubleshoot authorization mode configuration errors
- Use system logs to identify control plane failures
- Learn the recovery process for API server failures

### Problem Statement
The Kubernetes API server has stopped responding. Users are unable to interact with the cluster, and applications cannot be deployed. Your task is to diagnose and resolve the issue.

### Fault Injection
Execute on the master node:
```bash
# Create backup of API server manifest
cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
# Inject configuration error in apiserver manifest
sed -i 's/--authorization-mode=Node,RBAC/--authorization-mode=Node,RBAC,NonExistentMode/' /etc/kubernetes/manifests/kube-apiserver.yaml
# Create incorrect certificate path
sudo mkdir -p /etc/kubernetes/broken-certs/
```

### Diagnostic Steps
1. Verify API server status:
   ```bash
   kubectl get nodes
   # This should fail with connection refused error
   ```

2. Check API server pod status:
   ```bash
   crictl pods | grep api-server
   # The pod might be in a crash loop
   ```

3. Examine API server logs:
   ```bash
   crictl logs $(crictl pods | grep api-server | awk '{print $1}')
   # OR if pod not found:
   journalctl -xeu kubelet | grep api-server
   ```

4. Check API server manifest for errors:
   ```bash
   cat /etc/kubernetes/manifests/kube-apiserver.yaml
   ```

3. If necessary, use enhanced debug functionality:
   ```bash
   # Kubernetes 1.32 includes integrated debugging tools
   kubectl debug node/k8s-master -it --image=busybox
   # Inside the debug container, access node filesystem at /host
   
   # For advanced debugging with more tools
   kubectl debug node/k8s-master -it --image=nicolaka/netshoot --privileged
   
   # For runtime information and metrics
   kubectl debug node/k8s-master --profile=system-info
   ```

### Remediation
1. Restore the correct API server configuration:
   ```bash
   cp /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
   # OR edit the file to remove incorrect mode:
   sed -i 's/--authorization-mode=Node,RBAC,NonExistentMode/--authorization-mode=Node,RBAC/' /etc/kubernetes/manifests/kube-apiserver.yaml
   ```

2. Restart kubelet if necessary:
   ```bash
   systemctl restart kubelet
   ```

3. Verify recovery:
   ```bash
   # Wait for API server to start
   watch -n1 kubectl get nodes
   ```

## Scenario 2: Networking, Services, and DNS Troubleshooting

### Key Learning Objectives
- Diagnose CoreDNS configuration issues
- Understand DNS resolution in Kubernetes pods
- Learn how ConfigMaps affect CoreDNS behavior
- Use targeted debugging to isolate network services issues
- Apply configuration changes to restore DNS functionality

### Problem Statement
Applications are reporting DNS resolution failures and connectivity issues between pods. Your task is to diagnose and resolve the network infrastructure problems.

### Fault Injection
Execute on the master node:
```bash
# Corrupt CoreDNS ConfigMap
kubectl get configmap coredns -n kube-system -o yaml > /tmp/coredns-backup.yaml
kubectl edit configmap coredns -n kube-system
# Change Corefile to add a syntax error; for example, change:
#   kubernetes cluster.local in-addr.arpa ip6.arpa {
# to:
#   kubernetes cluster.local in-addr.arpa ip6.arpa X@ {

# Delete one of the CoreDNS pods to force configmap reload
kubectl delete pod -l k8s-app=kube-dns -n kube-system --limit=1
```

### Diagnostic Steps
1. Deploy a test pod and check DNS functionality:
   ```bash
   kubectl run busybox --image=busybox:1.28 -- sleep 3600
   kubectl exec -it busybox -- nslookup kubernetes.default
   # This will fail with DNS resolution errors
   
   # Check pod DNS configuration
   kubectl exec -it busybox -- cat /etc/resolv.conf
   # Verify nameserver points to CoreDNS service IP
   ```

2. Check CoreDNS pods status:
   ```bash
   kubectl get pods -n kube-system -l k8s-app=kube-dns
   # You'll likely see pods in CrashLoopBackOff or Error state
   ```

3. Check CoreDNS logs:
   ```bash
   kubectl logs -n kube-system -l k8s-app=kube-dns
   # This will show Corefile parsing errors
   ```

4. Examine the CoreDNS ConfigMap:
   ```bash
   kubectl get configmap coredns -n kube-system -o yaml
   # Look for syntax errors in the Corefile
   ```

### Remediation
1. Restore the correct CoreDNS ConfigMap:
   ```bash
   kubectl apply -f /tmp/coredns-backup.yaml
   # OR edit the configmap to fix the syntax:
   kubectl edit configmap coredns -n kube-system
   # Correct the syntax error by removing the 'X@'
   ```

2. Restart CoreDNS pods:
   ```bash
   kubectl delete pods -n kube-system -l k8s-app=kube-dns
   ```

3. Verify DNS resolution is working:
   ```bash
   kubectl exec -it busybox -- nslookup kubernetes.default
   kubectl exec -it busybox -- nslookup google.com
   ```

## Scenario 3: Cluster Node Failure and Recovery

### Key Learning Objectives
- Identify and troubleshoot node failures
- Master the node drainage process for maintenance
- Understand kubelet configuration and its impact on node health
- Learn proper procedures for removing nodes from a cluster
- Verify workload rescheduling after node recovery

### Problem Statement
One of your worker nodes has become unresponsive and is not accepting new pod deployments. Your task is to diagnose the issue, recover the node if possible, or safely evict and reschedule pods if necessary.

### Fault Injection
Execute on worker1:
```bash
# Corrupt kubelet configuration
sudo mv /etc/kubernetes/kubelet.conf /etc/kubernetes/kubelet.conf.bak
# Restart kubelet to trigger the failure
sudo systemctl restart kubelet
```

### Diagnostic Steps
1. Check the status of nodes:
   ```bash
   kubectl get nodes
   # The worker1 node will show as NotReady
   ```

2. Examine pods affected by the node failure:
   ```bash
   kubectl get pods --all-namespaces -o wide | grep worker1
   ```

3. Check node description for events:
   ```bash
   kubectl describe node worker1
   ```

4. SSH to the problematic node and check kubelet status:
   ```bash
   ssh worker1
   sudo systemctl status kubelet
   sudo journalctl -u kubelet | tail -n 50
   ```

5. Check for missing configuration files:
   ```bash
   sudo ls -la /etc/kubernetes/
   ```

### Remediation
1. Restore kubelet configuration on worker1:
   ```bash
   sudo mv /etc/kubernetes/kubelet.conf.bak /etc/kubernetes/kubelet.conf
   sudo systemctl restart kubelet
   ```

2. If node recovery isn't possible, safely drain the node:
   ```bash
   kubectl drain worker1 --ignore-daemonsets --delete-emptydir-data
   ```

3. If the node is permanently unavailable, remove it from the cluster:
   ```bash
   kubectl delete node worker1
   ```

4. Check pod rescheduling to other nodes:
   ```bash
   kubectl get pods --all-namespaces -o wide
   ```

4. Verify node recovery:
   ```bash
   kubectl get nodes
   # worker1 should show as Ready
   ```

## Scenario 4: ETCD Backup and Restore

### Problem Statement
A critical mistake has been made, resulting in the accidental deletion of important deployments and services. You need to restore the etcd backup to recover the cluster state.

### Fault Injection
First, create a backup and then simulate a disaster:
```bash
# 1. Create etcd backup
sudo mkdir -p /tmp/etcd-backup
# Get the etcd pod name
ETCDPOD=$(kubectl get pods -n kube-system | grep etcd | cut -d' ' -f1)
# Create etcd snapshot
sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /tmp/etcd-backup/etcd-snapshot.db

# 2. Create some test resources to delete later
kubectl create namespace recovery-test
kubectl create deployment nginx --image=nginx -n recovery-test
kubectl create service clusterip nginx --tcp=80:80 -n recovery-test
kubectl scale deployment nginx --replicas=3 -n recovery-test

# 3. Simulate disaster by deleting the test namespace
kubectl delete namespace recovery-test
```

### Diagnostic Steps
1. Verify that the namespace and resources are gone:
   ```bash
   kubectl get ns
   kubectl get deployment -n recovery-test
   kubectl get service -n recovery-test
   # These should show "not found" errors
   ```

2. Inspect available etcd backups:
   ```bash
   ls -la /tmp/etcd-backup/
   ```

3. Check etcd backup integrity:
   ```bash
   sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status /tmp/etcd-backup/etcd-snapshot.db
   
   # Verify ETCD version to ensure compatibility
   sudo ETCDCTL_API=3 etcdctl version
   ```

### Remediation
1. Stop the API server and etcd:
   ```bash
   # Move static pod manifests to temporarily stop these components
   sudo mkdir -p /tmp/manifests-backup
   sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/manifests-backup/
   sudo mv /etc/kubernetes/manifests/etcd.yaml /tmp/manifests-backup/
   ```

2. Restore from backup:
   ```bash
   # Restore snapshot to a new directory
   sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     snapshot restore /tmp/etcd-backup/etcd-snapshot.db \
     --data-dir=/var/lib/etcd-backup
   
   # Backup current etcd data directory
   sudo mv /var/lib/etcd /var/lib/etcd.old
   
   # Replace with restored data
   sudo mv /var/lib/etcd-backup /var/lib/etcd
   
   # Fix permissions
   sudo chown -R etcd:etcd /var/lib/etcd
   ```

3. Restart etcd and API server:
   ```bash
   sudo mv /tmp/manifests-backup/etcd.yaml /etc/kubernetes/manifests/
   sudo mv /tmp/manifests-backup/kube-apiserver.yaml /etc/kubernetes/manifests/
   ```

4. Verify restored resources:
   ```bash
   # Wait for API server to come back
   sleep 60
   kubectl get ns
   kubectl get deployment -n recovery-test
   kubectl get service -n recovery-test
   kubectl get pods -n recovery-test
   ```

## Scenario 5: Multi-Container Pod Troubleshooting and Resource Management

### Problem Statement
A critical application involves a multi-container pod with a sidecar logs collector. The pod is not starting properly due to resource constraints and container dependencies. Your task is to diagnose and fix the issues.

### Fault Injection
Create a problematic pod with resource issues:
```bash
cat <<EOF > /tmp/problematic-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
  namespace: default
spec:
  containers:
  - name: app-container
    image: nginx
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "512Mi"
        cpu: "500m"
    startupProbe:
      httpGet:
        path: /nonexistent-path
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3
  - name: sidecar-container
    image: busybox
    command: ["/bin/sh", "-c", "cat /non-existent-file; tail -f /var/log/nginx/access.log"]
    volumeMounts:
    - name: logs-volume
      mountPath: /var/log/nginx
    resources:
      requests:
        memory: "1Gi"
        cpu: "1"
      limits:
        memory: "1Gi"
        cpu: "1"
  volumes:
  - name: logs-volume
    emptyDir: {}
EOF

kubectl apply -f /tmp/problematic-pod.yaml
```

### Diagnostic Steps
1. Check pod status:
   ```bash
   kubectl get pod app-with-sidecar
   # Will show either Pending or CrashLoopBackOff
   ```

2. Describe the pod to check events:
   ```bash
   kubectl describe pod app-with-sidecar
   # Will show resource constraints or container crash issues
   ```

3. Check container logs:
   ```bash
   kubectl logs app-with-sidecar -c app-container
   kubectl logs app-with-sidecar -c sidecar-container
   # The sidecar container will show errors about missing file
   ```

4. Check node resource availability:
   ```bash
   kubectl describe nodes | grep -A 10 "Allocated resources"
   
   # Check resource usage
   kubectl top pods
   kubectl top nodes
   ```

### Remediation
1. Edit the pod specification to fix the issues:
   ```bash
   kubectl delete pod app-with-sidecar
   # Edit the YAML file to fix resource requirements and container issues
   ```

2. Create a corrected pod configuration:
   ```bash
   cat <<EOF > /tmp/fixed-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: app-with-sidecar
     namespace: default
   spec:
     containers:
     - name: app-container
       image: nginx
       resources:
         requests:
           memory: "128Mi"
           cpu: "100m"
         limits:
           memory: "256Mi"
           cpu: "200m"
       startupProbe:
         httpGet:
           path: /
           port: 80
         initialDelaySeconds: 5
         periodSeconds: 5
         failureThreshold: 3
     - name: sidecar-container
       image: busybox
       command: ["/bin/sh", "-c", "sleep 10; while true; do tail -f /var/log/nginx/access.log; sleep 5; done"]
       volumeMounts:
       - name: logs-volume
         mountPath: /var/log/nginx
       resources:
         requests:
           memory: "64Mi"
           cpu: "50m"
         limits:
           memory: "128Mi"
           cpu: "100m"
     volumes:
     - name: logs-volume
       emptyDir: {}
   EOF

   kubectl apply -f /tmp/fixed-pod.yaml
   ```

3. Verify the pod is running:
   ```bash
   kubectl get pod app-with-sidecar
   kubectl logs app-with-sidecar -c sidecar-container
   kubectl logs app-with-sidecar -c app-container
   ```

## Scenario 6: RBAC, Authentication, and Authorization

### Problem Statement
A new developer needs access to the cluster but with limited permissions. However, they're reporting that they can't access any resources. Your task is to diagnose and fix the RBAC configuration issue.

### Fault Injection
Create a problematic RBAC setup:
```bash
# Create a namespace for development
kubectl create namespace development

# Create a service account
kubectl create serviceaccount developer -n development

# Create a role with incorrect apiGroups
cat <<EOF > /tmp/developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: developer-role
rules:
- apiGroups: [""] # This should include "apps" for deployments
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
EOF

kubectl apply -f /tmp/developer-role.yaml

# Create a rolebinding with incorrect roleRef
cat <<EOF > /tmp/developer-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: development
subjects:
- kind: ServiceAccount
  name: developer
  namespace: development
roleRef:
  kind: ClusterRole # This should be Role, not ClusterRole
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f /tmp/developer-rolebinding.yaml

# Create a kubeconfig for the developer
DEVELOPER_TOKEN=$(kubectl create token developer -n development)
CLUSTER_NAME="kubernetes"
SERVER_URL=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
CA_DATA=$(kubectl config view --minify --flatten -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')

cat <<EOF > /tmp/developer.kubeconfig
apiVersion: v1
kind: Config
clusters:
- name: ${CLUSTER_NAME}
  cluster:
    server: ${SERVER_URL}
    certificate-authority-data: ${CA_DATA}
contexts:
- name: developer@${CLUSTER_NAME}
  context:
    cluster: ${CLUSTER_NAME}
    user: developer
    namespace: development
current-context: developer@${CLUSTER_NAME}
users:
- name: developer
  user:
    token: ${DEVELOPER_TOKEN}
EOF
```

### Diagnostic Steps
1. Test the developer's access:
   ```bash
   # Test access using the developer's kubeconfig
   kubectl --kubeconfig=/tmp/developer.kubeconfig get pods
   kubectl --kubeconfig=/tmp/developer.kubeconfig create deployment nginx --image=nginx
   # These should fail with authorization errors
   
   # Verify permissions using auth can-i
   kubectl --kubeconfig=/tmp/developer.kubeconfig auth can-i get pods --namespace=development
   kubectl --kubeconfig=/tmp/developer.kubeconfig auth can-i create deployments --namespace=development
   ```

2. Check the role configuration:
   ```bash
   kubectl get role developer-role -n development -o yaml
   # Notice the missing apiGroup for deployments
   ```

3. Check the role binding:
   ```bash
   kubectl get rolebinding developer-binding -n development -o yaml
   # Notice the incorrect kind (ClusterRole instead of Role)
   ```

4. Verify the service account:
   ```bash
   kubectl describe serviceaccount developer -n development
   ```

### Remediation
1. Fix the role configuration:
   ```bash
   cat <<EOF > /tmp/fixed-developer-role.yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     namespace: development
     name: developer-role
   rules:
   - apiGroups: ["", "apps"]
     resources: ["pods", "services", "deployments"]
     verbs: ["get", "list", "watch", "create", "update", "delete"]
   EOF

   kubectl apply -f /tmp/fixed-developer-role.yaml
   ```

2. Fix the role binding:
   ```bash
   cat <<EOF > /tmp/fixed-developer-rolebinding.yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: developer-binding
     namespace: development
   subjects:
   - kind: ServiceAccount
     name: developer
     namespace: development
   roleRef:
     kind: Role
     name: developer-role
     apiGroup: rbac.authorization.k8s.io
   EOF

   kubectl apply -f /tmp/fixed-developer-rolebinding.yaml
   ```

3. Generate a new kubeconfig for the developer:
   ```bash
   DEVELOPER_TOKEN=$(kubectl create token developer -n development)
   CLUSTER_NAME="kubernetes"
   SERVER_URL=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
   CA_DATA=$(kubectl config view --minify --flatten -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')

   cat <<EOF > /tmp/developer.kubeconfig
   apiVersion: v1
   kind: Config
   clusters:
   - name: ${CLUSTER_NAME}
     cluster:
       server: ${SERVER_URL}
       certificate-authority-data: ${CA_DATA}
   contexts:
   - name: developer@${CLUSTER_NAME}
     context:
       cluster: ${CLUSTER_NAME}
       user: developer
       namespace: development
   current-context: developer@${CLUSTER_NAME}
   users:
   - name: developer
     user:
       token: ${DEVELOPER_TOKEN}
   EOF
   ```

4. Verify the developer's access:
   ```bash
   # Test access using the developer's kubeconfig
   kubectl --kubeconfig=/tmp/developer.kubeconfig get pods
   kubectl --kubeconfig=/tmp/developer.kubeconfig create deployment nginx --image=nginx
   kubectl --kubeconfig=/tmp/developer.kubeconfig get deployments
   ```

## Scenario 7: Cluster Upgrade and Component Failure

### Problem Statement
Your organization requires an upgrade of the Kubernetes cluster from v1.31 to v1.32. During the upgrade, you encounter component failures that need to be addressed while maintaining cluster functionality.

> **Note**: This scenario uses the latest Kubernetes version (v1.32), which includes enhanced upgrade mechanisms and streamlined component management.

### Fault Injection
Simulate a problematic upgrade scenario:
```bash
# First, create some workloads to maintain during upgrade
kubectl create deployment important-app --image=nginx --replicas=3
kubectl create service clusterip important-app --tcp=80:80

# Simulate component version mismatch on worker2
ssh worker2 "sudo sed -i 's/v1.31/v1.30/' /etc/kubernetes/kubelet.conf"
ssh worker2 "sudo systemctl restart kubelet"

# On master, prepare for kubelet upgrade but introduce error
ssh k8s-master "sudo mv /etc/systemd/system/kubelet.service.d/10-kubeadm.conf /etc/systemd/system/kubelet.service.d/10-kubeadm.conf.bak"
```

### Diagnostic Steps
1. Check current component versions:
   ```bash
   kubectl version --short
   kubectl get nodes -o custom-columns=NAME:.metadata.name,VERSION:.status.nodeInfo.kubeletVersion
   ```

2. Check node status:
   ```bash
   kubectl get nodes
   kubectl describe node worker2
   ```

3. Check kubelet configuration on worker2:
   ```bash
   ssh worker2 "sudo systemctl status kubelet"
   ssh worker2 "sudo journalctl -u kubelet | tail -n 50"
   ssh worker2 "sudo cat /etc/kubernetes/kubelet.conf"
   ```

4. Check the kubelet service configuration on master:
   ```bash
   ssh k8s-master "sudo ls -la /etc/systemd/system/kubelet.service.d/"
   ssh k8s-master "sudo cat /usr/lib/systemd/system/kubelet.service"
   ```

### Remediation
1. Fix kubelet configuration on worker2:
   ```bash
   ssh worker2 "sudo sed -i 's/v1.30/v1.32/' /etc/kubernetes/kubelet.conf"
   ssh worker2 "sudo systemctl restart kubelet"
   ```

2. Restore kubelet service configuration on master:
   ```bash
   ssh k8s-master "sudo cp /etc/systemd/system/kubelet.service.d/10-kubeadm.conf.bak /etc/systemd/system/kubelet.service.d/10-kubeadm.conf"
   ssh k8s-master "sudo systemctl daemon-reload"
   ssh k8s-master "sudo systemctl restart kubelet"
   ```

3. Perform a proper upgrade using kubeadm:
   ```bash
   # On master node
   ssh k8s-master "sudo kubeadm upgrade plan"
   ssh k8s-master "sudo kubeadm upgrade apply v1.32.0 -y"
   
   # Update kubelet on master
   ssh k8s-master "sudo dnf upgrade -y kubelet kubectl"
   ssh k8s-master "sudo systemctl restart kubelet"
   
   # Upgrade worker nodes one at a time
   # For worker1
   kubectl drain worker1 --ignore-daemonsets
   ssh worker1 "sudo kubeadm upgrade node"
   ssh worker1 "sudo dnf upgrade -y kubelet"
   ssh worker1 "sudo systemctl restart kubelet"
   kubectl uncordon worker1
   
   # For worker2
   kubectl drain worker2 --ignore-daemonsets
   ssh worker2 "sudo kubeadm upgrade node"
   ssh worker2 "sudo dnf upgrade -y kubelet"
   ssh worker2 "sudo systemctl restart kubelet"
   kubectl uncordon worker2
   ```

4. Verify the upgrade:
   ```bash
   kubectl version --short
   kubectl get nodes -o custom-columns=NAME:.metadata.name,VERSION:.status.nodeInfo.kubeletVersion
   
   # Check important workloads are still functional
   kubectl get deployment important-app
   kubectl get service important-app
   kubectl get pods -l app=important-app
   ```

## Scenario 8: Persistent Volume and Storage Troubleshooting

### Key Learning Objectives
- Configure and troubleshoot Persistent Volumes and Persistent Volume Claims
- Understand storage provisioning and binding modes
- Diagnose access mode mismatches between PVs and PVCs
- Resolve storage permissions issues on host nodes
- Configure storage for stateful applications

### Problem Statement
A stateful application deployment is failing because its PersistentVolumeClaims are stuck in a Pending state. Your task is to diagnose and resolve storage configuration issues preventing the application from starting.

> **Note**: For dynamic provisioning with StorageClasses and CSI drivers, see Scenario 16: Dynamic Storage Provisioning.

### Fault Injection
Create a storage configuration with deliberate mismatches:
```bash
# Create storage class with incorrect provisioner
cat <<EOF > /tmp/broken-storage-class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF

kubectl apply -f /tmp/broken-storage-class.yaml

# Create persistent volume with incorrect access mode
cat <<EOF > /tmp/local-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany  # This should be ReadWriteOnce for local volumes
  persistentVolumeReclaimPolicy: Delete
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
          - worker1
EOF

kubectl apply -f /tmp/local-pv.yaml

# Create directory on worker1 with incorrect permissions
ssh worker1 "sudo mkdir -p /mnt/data && sudo chmod 700 /mnt/data"

# Create a stateful application that requires storage
cat <<EOF > /tmp/stateful-app.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  selector:
    matchLabels:
      app: database
  serviceName: "database"
  replicas: 1
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: database
        image: postgres:13
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_PASSWORD
          value: "password123"
        volumeMounts:
        - name: database-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: database-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "local-storage"
      resources:
        requests:
          storage: 1Gi
EOF

kubectl apply -f /tmp/stateful-app.yaml
```

### Diagnostic Steps
1. Check the StatefulSet status:
   ```bash
   kubectl get statefulset database
   kubectl get pods -l app=database
   # Pod will likely be stuck in ContainerCreating or Pending
   ```

2. Check PVC status:
   ```bash
   kubectl get pvc
   # The PVC will be stuck in Pending
   kubectl describe pvc database-data-database-0
   # Look for events indicating why binding failed
   ```

3. Examine the PV and StorageClass configuration:
   ```bash
   kubectl get pv local-pv -o yaml
   kubectl get sc local-storage -o yaml
   # Notice the access mode mismatch between PV and PVC
   ```

4. Check node storage configuration:
   ```bash
   ssh worker1 "ls -la /mnt/data"
   # Note the restricted permissions
   ```

### Remediation
1. Fix the PV access modes to match PVC requirements:
   ```bash
   kubectl delete pv local-pv
   
   cat <<EOF > /tmp/fixed-local-pv.yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: local-pv
   spec:
     capacity:
       storage: 1Gi
     accessModes:
       - ReadWriteOnce  # Fixed to match PVC requirements
     persistentVolumeReclaimPolicy: Delete
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
             - worker1
   EOF
   
   kubectl apply -f /tmp/fixed-local-pv.yaml
   ```

2. Fix directory permissions on the worker node:
   ```bash
   ssh worker1 "sudo chmod 755 /mnt/data"
   ssh worker1 "sudo chown 999:999 /mnt/data"  # 999 is the postgres user in the container
   ```

3. Delete the pod to trigger recreation with correct storage:
   ```bash
   kubectl delete pod database-0
   ```

4. Verify the application is now running:
   ```bash
   kubectl get pods -l app=database
   kubectl get pvc
   kubectl get pv
   # Check that the database pod is running and the PVC is bound
   ```

## Scenario 9: Network Policy and Pod Isolation Troubleshooting

### Key Learning Objectives
- Configure and troubleshoot NetworkPolicy resources
- Understand ingress and egress traffic control in Kubernetes
- Diagnose connectivity issues between namespaces and services
- Learn how to apply the principle of least privilege to pod networking
- Test and verify NetworkPolicy effects using debugging techniques

### Problem Statement
A microservices application has been deployed with restrictive NetworkPolicies for security. However, communication between frontend and backend services is failing. Your task is to diagnose and fix the network connectivity issues while maintaining security.

> **Note**: In Kubernetes 1.32, NetworkPolicy features include enhanced observability, policy recommendation engines, and conflict detection capabilities.

### Fault Injection
Create application components with restrictive network policies:
```bash
# Create namespaces for frontend and backend
kubectl create namespace frontend
kubectl create namespace backend

# Deploy backend service
cat <<EOF > /tmp/backend-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-app
  namespace: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
  namespace: backend
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 80
EOF

kubectl apply -f /tmp/backend-app.yaml

# Deploy frontend app
cat <<EOF > /tmp/frontend-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-app
  namespace: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: busybox
        command: ["/bin/sh", "-c", "while true; do wget -q -O- http://backend-svc.backend.svc.cluster.local; sleep 5; done"]
EOF

kubectl apply -f /tmp/frontend-app.yaml

# Create restrictive network policy for backend
cat <<EOF > /tmp/restrictive-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: backend
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF

kubectl apply -f /tmp/restrictive-policy.yaml
```

### Diagnostic Steps
1. Check frontend pod logs for connection errors:
   ```bash
   kubectl get pods -n frontend
   FRONTEND_POD=$(kubectl get pod -n frontend -l app=frontend -o jsonpath='{.items[0].metadata.name}')
   kubectl logs -n frontend $FRONTEND_POD
   # Should show connection refused or timeout errors
   
   # Use ephemeral debug container to troubleshoot
   kubectl debug -n frontend $FRONTEND_POD --image=nicolaka/netshoot -it -- /bin/bash
   # Inside debug container:
   # curl -v backend-svc.backend.svc.cluster.local
   ```

2. Verify the network policies in place:
   ```bash
   kubectl get networkpolicy -n backend
   kubectl describe networkpolicy -n backend deny-all-ingress
   # Notice it's blocking all incoming traffic
   ```

3. Test connectivity directly from frontend pod:
   ```bash
   kubectl exec -it -n frontend $FRONTEND_POD -- wget -q -O- --timeout=5 http://backend-svc.backend.svc.cluster.local
   # Should fail with a timeout or connection refused
   ```

4. Check DNS resolution:
   ```bash
   kubectl exec -it -n frontend $FRONTEND_POD -- nslookup backend-svc.backend.svc.cluster.local
   # This should work as DNS is not affected by network policies
   ```

### Remediation
1. Create a targeted network policy allowing frontend access:
   ```bash
   cat <<EOF > /tmp/allow-frontend-policy.yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: allow-frontend-to-backend
     namespace: backend
   spec:
     podSelector:
       matchLabels:
         app: backend
     policyTypes:
     - Ingress
     ingress:
     - from:
       - namespaceSelector:
           matchLabels:
             kubernetes.io/metadata.name: frontend
         podSelector:
           matchLabels:
             app: frontend
       ports:
       - protocol: TCP
         port: 80
   EOF
   
   kubectl apply -f /tmp/allow-frontend-policy.yaml
   ```

2. Add namespace label to make the selector work:
   ```bash
   kubectl label namespace frontend kubernetes.io/metadata.name=frontend
   ```

3. Verify connectivity is restored:
   ```bash
   # Run the network diagnostics tool
   kubectl network diagnose pod/$FRONTEND_POD -n frontend --target-service backend-svc.backend
   # Should show successful connection
   
   # Watch the frontend pod logs
   kubectl logs -n frontend $FRONTEND_POD -f
   # Should now show successful connections
   
   # Or directly test connectivity
   kubectl exec -it -n frontend $FRONTEND_POD -- wget -q -O- --timeout=5 http://backend-svc.backend.svc.cluster.local
   # Should return the nginx welcome page
   
   # Verify NetworkPolicy rules
   kubectl describe networkpolicy allow-frontend-to-backend -n backend
   
   # Check NetworkPolicy metrics (new in 1.32)
   kubectl get networkpolicymetrics -n backend
   ```

## Scenario 10: Advanced Scheduling with Taints, Tolerations, and Affinities

### Key Learning Objectives
- Configure and troubleshoot taints and tolerations for node selection
- Understand pod and node affinity/anti-affinity rules
- Use advanced scheduling constraints to distribute workloads
- Diagnose pod scheduling failures
- Implement high-availability deployment patterns using topology constraints

### Problem Statement
A high-priority application needs to be scheduled with specific node placement requirements, but the pods are either not scheduling or landing on the wrong nodes. Your task is to diagnose scheduling constraints and ensure proper workload placement.

### Fault Injection
Set up a cluster with taints and scheduling constraints:
```bash
# Taint worker1 to repel regular workloads
kubectl taint nodes worker1 dedicated=high-priority:NoSchedule

# Taint worker2 with a taint that has no matching toleration
kubectl taint nodes worker2 environment=production:NoSchedule

# Create a deployment without proper tolerations
cat <<EOF > /tmp/priority-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: priority-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: priority-app
  template:
    metadata:
      labels:
        app: priority-app
    spec:
      containers:
      - name: app
        image: nginx
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
EOF

kubectl apply -f /tmp/priority-app.yaml

# Create a regular deployment to take up resources
cat <<EOF > /tmp/regular-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: regular-app
spec:
  replicas: 10
  selector:
    matchLabels:
      app: regular-app
  template:
    metadata:
      labels:
        app: regular-app
    spec:
      containers:
      - name: app
        image: nginx
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
EOF

kubectl apply -f /tmp/regular-app.yaml
```

### Diagnostic Steps
1. Check pod distribution across nodes:
   ```bash
   kubectl get pods -o wide
   # Notice priority-app pods may be pending or all on master node
   ```

2. Check node taints:
   ```bash
   kubectl describe nodes worker1 | grep Taint
   kubectl describe nodes worker2 | grep Taint
   ```

3. Check why pods aren't scheduling:
   ```bash
   PENDING_POD=$(kubectl get pod -l app=priority-app -o jsonpath='{.items[?(@.status.phase=="Pending")].metadata.name}' | head -n1)
   if [ -n "$PENDING_POD" ]; then
     kubectl describe pod $PENDING_POD | grep -A10 Events
   fi
   ```

4. Examine existing pod placements:
   ```bash
   kubectl get pods -l app=regular-app -o wide
   kubectl get pods -l app=priority-app -o wide
   ```

### Remediation
1. Update priority-app deployment with appropriate tolerations and node affinity:
   ```bash
   kubectl delete deployment priority-app
   
   cat <<EOF > /tmp/fixed-priority-app.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: priority-app
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: priority-app
     template:
       metadata:
         labels:
           app: priority-app
       spec:
         tolerations:
         - key: "dedicated"
           operator: "Equal"
           value: "high-priority"
           effect: "NoSchedule"
         affinity:
           nodeAffinity:
             preferredDuringSchedulingIgnoredDuringExecution:
             - weight: 100
               preference:
                 matchExpressions:
                 - key: kubernetes.io/hostname
                   operator: In
                   values:
                   - worker1
         containers:
         - name: app
           image: nginx
           resources:
             requests:
               memory: "64Mi"
               cpu: "100m"
             limits:
               memory: "128Mi"
               cpu: "200m"
   EOF
   
   kubectl apply -f /tmp/fixed-priority-app.yaml
   ```

2. Verify pod placement:
   ```bash
   kubectl get pods -l app=priority-app -o wide
   # Pods should now be scheduled on worker1
   ```

3. Add a pod anti-affinity to ensure high availability:
   ```bash
   kubectl delete deployment priority-app
   
   cat <<EOF > /tmp/ha-priority-app.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: priority-app
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: priority-app
     template:
       metadata:
         labels:
           app: priority-app
       spec:
         tolerations:
         - key: "dedicated"
           operator: "Equal"
           value: "high-priority"
           effect: "NoSchedule"
         - key: "environment"
           operator: "Equal"
           value: "production"
           effect: "NoSchedule"
         affinity:
           podAntiAffinity:
             preferredDuringSchedulingIgnoredDuringExecution:
             - weight: 100
               podAffinityTerm:
                 labelSelector:
                   matchExpressions:
                   - key: app
                     operator: In
                     values:
                     - priority-app
                 topologyKey: "kubernetes.io/hostname"
         containers:
         - name: app
           image: nginx
           resources:
             requests:
               memory: "64Mi"
               cpu: "100m"
             limits:
               memory: "128Mi"
               cpu: "200m"
   EOF
   
   kubectl apply -f /tmp/ha-priority-app.yaml
   ```

4. Verify distribution across nodes:
   ```bash
   kubectl get pods -l app=priority-app -o wide
   # Pods should be distributed across worker1 and worker2 for HA
   ```

## Scenario 13: Certificate Rotation and TLS Troubleshooting

### Problem Statement
The cluster's kubelets have failing TLS connections to the API server. Upon investigation, you discover that the kubelet client certificates have expired. Your task is to troubleshoot the TLS errors and rotate the kubelet client certificates to restore proper node-to-control-plane communication.

### Fault Injection
Set up a scenario with expired certificates:
```bash
# First, backup the current certificate for reference
ssh worker1 "sudo cp /var/lib/kubelet/pki/kubelet-client-current.pem /tmp/kubelet-client-backup.pem"

# Create a short-lived certificate on worker1
ssh worker1 "sudo kubeadm cert renew kubelet-client --config /tmp/kubeadm-config.yaml"

# Artificially expire the certificate by modifying the timestamp
ssh worker1 "sudo sed -i 's/NotAfter:/NotAfter: 210101000000Z # /' /var/lib/kubelet/pki/kubelet-client-current.pem"

# Restart kubelet to pick up the "expired" certificate
ssh worker1 "sudo systemctl restart kubelet"
```

### Diagnostic Steps
1. Check the node status and kubelet healthiness:
   ```bash
   kubectl get nodes
   # worker1 will show as NotReady

   kubectl describe node worker1
   # Look for conditions and errors related to kubelet communication
   ```

2. Examine kubelet logs on the problematic node:
   ```bash
   ssh worker1 "sudo journalctl -u kubelet | tail -n 50"
   # Look for TLS certificate errors
   ```

3. Check certificate expiration:
   ```bash
   ssh worker1 "sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout | grep -A2 Validity"
   # Note the certificate validity period
   ```

4. Check API server connection from worker node:
   ```bash
   ssh worker1 "curl -v https://k8s-master:6443/healthz --cacert /etc/kubernetes/pki/ca.crt --cert /var/lib/kubelet/pki/kubelet-client-current.pem --key /var/lib/kubelet/pki/kubelet-client-current.pem"
   # Should show TLS/certificate errors
   ```

### Remediation
1. Rotate the kubelet client certificate:
   ```bash
   # In a kubeadm-managed cluster, we can renew cert with kubeadm
   ssh worker1 "sudo kubeadm cert renew kubelet-client"
   
   # Alternative manual approach (if needed):
   ssh worker1 "sudo cp /tmp/kubelet-client-backup.pem /var/lib/kubelet/pki/kubelet-client-current.pem"
   ```

2. Restart kubelet to use the new certificate:
   ```bash
   ssh worker1 "sudo systemctl restart kubelet"
   ```

3. Verify the node has rejoined the cluster:
   ```bash
   kubectl get nodes
   # worker1 should show as Ready
   
   # Verify certificate validity
   ssh worker1 "sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout | grep -A2 Validity"
   ```

4. Set up automatic certificate rotation for future renewals:
   ```bash
   # In Kubernetes 1.32, certificate rotation is enabled by default with advanced features
   # Check that the automatic rotation is functioning
   ssh worker1 "sudo systemctl status kubelet | grep certificate-rotation"
   
   # If needed, verify the rotation settings in the kubelet configuration
   ssh worker1 "sudo grep -A5 rotation /var/lib/kubelet/config.yaml"
   
   # Certificates should appear in the auto rotation list
   ssh worker1 "sudo kubeadm certs check-expiration | grep kubelet-client"
   ```

## Scenario 14: Admission Controllers and Pod Security Standards

### Problem Statement
Your organization has implemented Pod Security Standards using Kubernetes Admission Controllers. Development teams are reporting that their workloads are being blocked from deployment. Your task is to diagnose the admission controller issues, identify policy violations, and adjust the deployment to comply with security standards.

### Fault Injection
Create a namespace with restrictive Pod Security Standards and a non-compliant pod:
```bash
# Create a namespace with enforced Pod Security Standards
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: restricted-ns
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
EOF

# Create a non-compliant pod with privileged settings
cat <<EOF > /tmp/privileged-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-violation-pod
  namespace: restricted-ns
spec:
  containers:
  - name: privileged-container
    image: nginx
    securityContext:
      privileged: true
      runAsUser: 0
    volumeMounts:
    - name: host-path
      mountPath: /host
  volumes:
  - name: host-path
    hostPath:
      path: /
EOF

# Try to deploy the non-compliant pod
kubectl apply -f /tmp/privileged-pod.yaml
# This should be rejected by admission controllers
```

### Diagnostic Steps
1. Check if the pod was created:
   ```bash
   kubectl get pods -n restricted-ns
   # The pod should not exist due to admission control
   ```

2. Review the error message from pod creation:
   ```bash
   kubectl apply -f /tmp/privileged-pod.yaml
   # Should show detailed errors about security violations
   ```

3. Check namespace pod security standards:
   ```bash
   kubectl get namespace restricted-ns -o yaml
   # Look for pod-security.kubernetes.io labels
   ```

4. Check if admission controllers are enabled:
   ```bash
   kubectl exec -n kube-system kube-apiserver-k8s-master -- kube-apiserver -h | grep admission-plugins
   # Or check the API server configuration:
   kubectl get pod -n kube-system kube-apiserver-k8s-master -o yaml | grep admission-control
   ```

### Remediation
1. Create a compliant pod specification:
   ```bash
   cat <<EOF > /tmp/compliant-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: security-compliant-pod
     namespace: restricted-ns
   spec:
     securityContext:
       runAsNonRoot: true
       runAsUser: 1000
       runAsGroup: 3000
       fsGroup: 2000
       seccompProfile:
         type: RuntimeDefault
     containers:
     - name: restricted-container
       image: nginx
       securityContext:
         allowPrivilegeEscalation: false
         capabilities:
           drop:
           - ALL
         runAsNonRoot: true
         runAsUser: 1000
         seccompProfile:
           type: RuntimeDefault
       resources:
         limits:
           cpu: "100m"
           memory: "128Mi"
         requests:
           cpu: "50m"
           memory: "64Mi"
   EOF
   
   kubectl apply -f /tmp/compliant-pod.yaml
   ```

2. Verify the compliant pod is allowed:
   ```bash
   kubectl get pods -n restricted-ns
   ```

3. For development/testing, create a namespace with less restrictive policies:
   ```bash
   cat <<EOF | kubectl apply -f -
   apiVersion: v1
   kind: Namespace
   metadata:
     name: dev-ns
     labels:
       pod-security.kubernetes.io/enforce: baseline
       pod-security.kubernetes.io/audit: restricted
       pod-security.kubernetes.io/warn: restricted
   EOF
   ```

4. Document the Pod Security Standards requirements for development teams:
   ```bash
   cat <<EOF > /tmp/pod-security-standards.md
   # Pod Security Standards Requirements
   
   ## Restricted Namespace Requirements
   - Must run as non-root
   - Must use a read-only root filesystem
   - Must drop ALL capabilities
   - Must disable privileged mode and privilege escalation
   - Must use RuntimeDefault seccomp profile
   - Cannot use hostPath volumes or other host namespaces
   
   ## Baseline Namespace Requirements
   - Cannot use privileged mode
   - Cannot mount hostPath volumes to sensitive directories
   - Must drop dangerous capabilities
   
   See full documentation at: https://kubernetes.io/docs/concepts/security/pod-security-standards/
   EOF
   ```

## Scenario 15: Combined Mock Exam Scenario

### Key Learning Objectives
- Diagnose and resolve multiple concurrent issues in a Kubernetes cluster
- Apply a systematic troubleshooting methodology for complex problems
- Manage time effectively under pressure
- Combine knowledge from multiple CKA domains
- Verify resolutions for interdependent issues

### Problem Statement
Your production Kubernetes cluster is experiencing multiple issues simultaneously. Users are reporting that applications are unavailable, deployments are failing, and nodes are showing various statuses. You need to methodically identify and resolve all issues while minimizing downtime.

> **Time Estimate**: 30-45 minutes (depending on your familiarity with the concepts)
>
> **Note**: This scenario showcases Kubernetes 1.32's enhanced troubleshooting features, including the unified diagnostics framework, multi-resource debugging, and cluster health analyzers.

### Fault Injection
Execute the following commands to prepare the mock exam scenario:
```bash
# 1. Create a script to inject multiple faults
cat <<EOF > /tmp/mock-exam-setup.sh
#!/bin/bash
set -e

echo "Setting up Mock Exam scenario..."

# Break API server on master
echo "1. Breaking API server configuration..."
sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/kube-apiserver.yaml.bak
sudo sed -i 's/--etcd-servers=/--etcd-servers=https:\/\/127.0.0.1:2380,/' /etc/kubernetes/manifests/kube-apiserver.yaml

# Disrupt networking on worker1
echo "2. Disrupting network on worker1..."
ssh worker1 "sudo iptables -I INPUT -p tcp --dport 10250 -j DROP"

# Break RBAC for system:kube-scheduler
echo "3. Breaking scheduler RBAC permissions..."
cat <<EOT | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-scheduler
subjects:
- kind: User
  name: system:kube-scheduler
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: wrong-scheduler-permissions
  apiGroup: rbac.authorization.k8s.io
EOT

# Create resource quotas that will conflict with deployments
echo "4. Creating restrictive ResourceQuota..."
kubectl create namespace restricted-space
cat <<EOT | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: restricted-space
spec:
  hard:
    pods: "1"
    requests.cpu: "200m"
    requests.memory: "100Mi"
    limits.cpu: "300m"
    limits.memory: "200Mi"
EOT

# Deploy a non-functional application in the restricted namespace
echo "5. Deploying resource-intensive application..."
kubectl run heavy-app --image=nginx -n restricted-space --requests=cpu=500m,memory=500Mi --limits=cpu=700m,memory=700Mi

# Create a persistent volume with incorrect permissions
echo "6. Creating broken PV/PVC configuration..."
kubectl create namespace data-services
ssh worker1 "sudo mkdir -p /mnt/data && sudo chmod 700 /mnt/data"
cat <<EOT | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: broken-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Delete
  hostPath:
    path: /mnt/data
EOT

cat <<EOT | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
  namespace: data-services
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOT

cat <<EOT | kubectl apply -f -
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
  namespace: data-services
spec:
  selector:
    matchLabels:
      app: database
  serviceName: "database"
  replicas: 1
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: database
        image: postgres:13
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_PASSWORD
          value: "password123"
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: app-data
EOT

# Create NetworkPolicy that blocks essential traffic
echo "7. Creating restrictive NetworkPolicy..."
cat <<EOT | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: kube-system
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOT

echo "Mock Exam is set up with multiple issues to resolve."
EOF

# Make the script executable and run it
chmod +x /tmp/mock-exam-setup.sh
/tmp/mock-exam-setup.sh
```

### Diagnostic Steps

#### 1. Initial Assessment
```bash
# Check cluster status
kubectl get nodes
# This will likely fail due to API server issues

# Use the Kubernetes 1.32 cluster diagnostics tool
kubectl cluster diagnose
# This will attempt to identify control plane issues even with API server down

# Check node health directly from each node
ssh k8s-master "sudo k8s-node-health-check"
ssh worker1 "sudo k8s-node-health-check"
ssh worker2 "sudo k8s-node-health-check"
# The integrated node health check tool provides status even when API server is unavailable
```

#### 2. Diagnose API Server Issues
```bash
# Check API server status on master
ssh k8s-master "sudo crictl pods | grep api-server"
ssh k8s-master "sudo journalctl -u kubelet | grep api-server | tail -50"

# Check API server configuration
ssh k8s-master "sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml"
# Look for issues with etcd configuration
```

#### 3. Check Pod and Application Status
Once API server is fixed:
```bash
kubectl get pods --all-namespaces
# Look for pods in problematic states (Pending, CrashLoopBackOff, etc.)

kubectl describe pods -n restricted-space heavy-app
# Check for resource quota issues

kubectl describe pods -n data-services
# Check for PV/PVC binding issues
```

#### 4. Check Network Connectivity
```bash
# Test worker1 connectivity
ssh worker1 "curl -k https://k8s-master:6443/healthz"
# Check iptables rules
ssh worker1 "sudo iptables -L -n"
```

#### 5. Check RBAC and Permissions
```bash
kubectl get clusterrolebindings | grep scheduler
kubectl describe clusterrolebinding system:kube-scheduler
# Look for incorrect role references
```

#### 6. Check Network Policies
```bash
kubectl get networkpolicies --all-namespaces
kubectl describe networkpolicy -n kube-system default-deny-all
# Check if critical services are being blocked
```

### Remediation

#### 1. Fix API Server Configuration
```bash
# Restore API server configuration from backup
ssh k8s-master "sudo cp /tmp/kube-apiserver.yaml.bak /etc/kubernetes/manifests/kube-apiserver.yaml"
# Wait for API server to recover
sleep 60
```

#### 2. Fix Network Connectivity on worker1
```bash
# Remove the blocking iptables rule
ssh worker1 "sudo iptables -D INPUT -p tcp --dport 10250 -j DROP"
```

#### 3. Fix Scheduler RBAC Permissions
```bash
# Delete the incorrect binding and create the correct one
kubectl delete clusterrolebinding system:kube-scheduler
kubectl create clusterrolebinding system:kube-scheduler --clusterrole=system:kube-scheduler --user=system:kube-scheduler
```

#### 4. Fix Resource Quota Issues
```bash
# Adjust the resource quota to accommodate the application
kubectl delete pod heavy-app -n restricted-space
kubectl edit resourcequota compute-quota -n restricted-space
# Increase the resource limits or
kubectl delete resourcequota compute-quota -n restricted-space
```

#### 5. Fix PV/PVC Issues
```bash
# Fix permissions on the storage path
ssh worker1 "sudo chmod 755 /mnt/data"
# Delete and recreate PV with correct access modes
kubectl delete pv broken-pv
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fixed-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  hostPath:
    path: /mnt/data
EOF
# Restart the statefulset
kubectl rollout restart statefulset -n data-services database
```

#### 6. Fix NetworkPolicy Issues
```bash
# Remove the overly restrictive NetworkPolicy
kubectl delete networkpolicy -n kube-system default-deny-all
```

### Verification
```bash
# Verify nodes are Ready
kubectl get nodes
# All nodes should show Ready status

# Verify pods are running
kubectl get pods --all-namespaces | grep -v Running
# Check for non-running pods

# Verify the StatefulSet in data-services
kubectl get statefulset -n data-services
kubectl get pods -n data-services
# Should show Running status

# Test connectivity between services
kubectl run test-pod --image=busybox -- sleep 3600
kubectl exec -it test-pod -- wget -q -O- --timeout=5 http://kubernetes.default
# Should return a response
```

### Key Exam Tips
During the actual CKA exam, when facing multiple issues:
1. Fix control plane issues first (API server, etcd, scheduler)
2. Restore cluster communication pathways (networking, node connectivity)
3. Resolve RBAC and security issues
4. Address application-specific problems (resource conflicts, storage issues)
5. Always verify fixes before moving to the next issue

## Scenario 16: Dynamic Storage Provisioning with CSI Drivers

### Key Learning Objectives
- Configure and troubleshoot dynamic storage provisioning
- Understand Container Storage Interface (CSI) drivers
- Create and manage StorageClasses for automated PV provisioning
- Diagnose common storage provisioning failures
- Configure volume expansion and storage class parameters

### Problem Statement
Your team needs to implement dynamic storage provisioning for applications. The current static provisioning approach is causing delays in deployment and scaling. You need to configure a CSI driver, set up appropriate StorageClasses, and ensure applications can automatically get the storage they need.

> **Note**: For static provisioning with manually created PVs, see Scenario 8: Persistent Volume and Storage Troubleshooting.
>
> In Kubernetes 1.32, the CSI framework has been enhanced with automatic migration tools, volume health monitoring, and snapshot scheduling capabilities.

### Preparation
This scenario requires setting up a CSI driver. We'll use a local-path provisioner as it's simple to deploy in a test environment:

```bash
# Install the enhanced local-path provisioner for Kubernetes 1.32
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v2.0.0/deploy/local-path-storage.yaml

# Verify the installation
kubectl get pods -n local-path-storage
```

### Fault Injection
Create a scenario with misconfigured storage:

```bash
# Create a namespace for our test application
kubectl create namespace storage-test

# Create a restrictive StorageClass that doesn't match our needs
cat <<EOF > /tmp/broken-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: restricted-storage
provisioner: k8s.io/minikube-hostpath
parameters:
  type: pd-standard
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: false
reclaimPolicy: Delete
EOF

kubectl apply -f /tmp/broken-storageclass.yaml

# Create a PVC that tries to use our non-working StorageClass
cat <<EOF > /tmp/test-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
  namespace: storage-test
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: restricted-storage
EOF

kubectl apply -f /tmp/test-pvc.yaml

# Create a pod that tries to use this PVC
cat <<EOF > /tmp/storage-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-app
  namespace: storage-test
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: dynamic-pvc
EOF

kubectl apply -f /tmp/storage-pod.yaml
```

### Diagnostic Steps
1. Check the PVC status:

```bash
kubectl get pvc -n storage-test
# Example output:
# NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS          AGE
# dynamic-pvc   Pending                                      restricted-storage    30s

kubectl describe pvc dynamic-pvc -n storage-test
# Look for error messages such as:
# Events:
#   Type     Reason              Age                From                         Message
#   ----     ------              ----               ----                         -------
#   Warning  ProvisioningFailed  15s (x3 over 45s)  persistentvolume-controller  storageclass.storage.k8s.io "restricted-storage" not found
```

2. Check the pod status:

```bash
kubectl get pod storage-app -n storage-test
# Example output:
# NAME          READY   STATUS    RESTARTS   AGE
# storage-app   0/1     Pending   0          1m

kubectl describe pod storage-app -n storage-test
# Example relevant output:
# Events:
#   Type     Reason            Age                From               Message
#   ----     ------            ----               ----               -------
#   Warning  FailedScheduling  45s (x3 over 1m)   default-scheduler  persistentvolumeclaim "dynamic-pvc" not found
#   Warning  FailedMount       30s (x3 over 1m)   kubelet            MountVolume.SetUp failed for volume "data-volume" : persistentvolumeclaim "dynamic-pvc" not bound
```

3. Check available StorageClasses and the problematic one:

```bash
kubectl get storageclass
# Example output:
# NAME                 PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
# restricted-storage   k8s.io/minikube-hostpath       Delete          WaitForFirstConsumer   false                  2m
# local-path           rancher.io/local-path          Delete          WaitForFirstConsumer   false                  5m
# standard (default)   k8s.io/minikube-hostpath       Delete          Immediate              false                  10m

# Check the details of our StorageClass
kubectl describe storageclass restricted-storage
# Example output showing the issue:
# Name:            restricted-storage
# IsDefaultClass:  No
# Annotations:     <none>
# Provisioner:     k8s.io/minikube-hostpath  # Wrong provisioner - not installed in this cluster
# Parameters:      type=pd-standard
# ReclaimPolicy:   Delete
# VolumeBindingMode:  WaitForFirstConsumer
# AllowVolumeExpansion:  false
# Events:
#   Type     Reason    Age                From                Message
#   ----     ------    ----               ----                -------
#   Warning  NotFound  15s (x3 over 45s)  persistentvolume-controller  no volume plugin matched
```

4. Check if the provisioner is running:

```bash
# If we're using local-path-provisioner:
kubectl get pods -n local-path-storage
# Example output:
# NAME                                     READY   STATUS    RESTARTS   AGE
# local-path-provisioner-7884d988bc-xntzl  1/1     Running   0          10m

kubectl logs -n local-path-storage -l app=local-path-provisioner
# Look for errors like:
# E0303 12:34:56.789012 1 controller.go:987] failed to provision volume with StorageClass "restricted-storage": no volume plugin matched
# E0303 12:34:56.789012 1 controller.go:987] failed to find plugin k8s.io/minikube-hostpath, plugin not registered
```

5. Check if any PVs have been created:

```bash
kubectl get pv
# Should not show any PVs for our claim since provisioning failed
```

### OS-Level Storage Issues

When troubleshooting storage problems in Kubernetes, remember to check OS-level configurations that might affect volume mounting:

1. **SELinux issues**:
   ```bash
   # Check if SELinux is enforcing
   ssh worker1 sudo getenforce
   # If it returns "Enforcing", it might be blocking volume access
   
   # Temporarily set to permissive for testing
   ssh worker1 sudo setenforce 0
   
   # For a permanent fix, create the correct SELinux context
   ssh worker1 sudo chcon -Rt svirt_sandbox_file_t /mnt/data
   ```

2. **Directory permissions**:
   ```bash
   # Check directory permissions
   ssh worker1 ls -la /mnt/data
   
   # Fix permissions if needed - typical container UIDs need read/write
   ssh worker1 sudo chmod 755 /mnt/data
   ```

3. **AppArmor/SecComp profiles** (if enabled):
   ```bash
   # Check if profiles are loaded
   ssh worker1 sudo aa-status
   
   # Check Docker/containerd configuration for default profiles
   ssh worker1 cat /etc/docker/daemon.json
   ```

4. **Disk space issues**:
   ```bash
   # Check available disk space
   ssh worker1 df -h
   
   # Check inode usage (sometimes overlooked)
   ssh worker1 df -i
   ```

These OS-level checks are especially important for local storage provisioners.

### Remediation
1. Fix the StorageClass to use the correct provisioner:

```bash
cat <<EOF > /tmp/fixed-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-local-storage
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
allowVolumeExpansion: true
EOF

kubectl apply -f /tmp/fixed-storageclass.yaml
```

2. Update the PVC to use the working StorageClass:

```bash
kubectl delete pvc dynamic-pvc -n storage-test

cat <<EOF > /tmp/fixed-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
  namespace: storage-test
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast-local-storage
EOF

kubectl apply -f /tmp/fixed-pvc.yaml
```

3. Recreate the pod to use the new PVC:

```bash
kubectl delete pod storage-app -n storage-test
kubectl apply -f /tmp/storage-pod.yaml
```

4. Verify that the PVC is bound and the pod is running:

```bash
kubectl get pvc -n storage-test
# Should show Bound status

kubectl get pod storage-app -n storage-test
# Should show Running status

# Examine the automatically created PV
kubectl get pv
kubectl describe pv $(kubectl get pv | grep dynamic-pvc | awk '{print $1}')
```

5. Test data persistence:

```bash
# Write some data to the volume
kubectl exec -n storage-test storage-app -- sh -c "echo 'This is persistent data' > /usr/share/nginx/html/index.html"

# Delete and recreate the pod
kubectl delete pod storage-app -n storage-test
kubectl apply -f /tmp/storage-pod.yaml

# Verify data persists
kubectl exec -n storage-test storage-app -- cat /usr/share/nginx/html/index.html
# Should output: This is persistent data
```

### Remediation
1. Fix the StorageClass to use the correct provisioner:

```bash
cat <<EOF > /tmp/fixed-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-local-storage
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
allowVolumeExpansion: true
EOF

kubectl apply -f /tmp/fixed-storageclass.yaml

# Example output:
# storageclass.storage.k8s.io/fast-local-storage created
```

2. Update the PVC to use the working StorageClass:

```bash
kubectl delete pvc dynamic-pvc -n storage-test

cat <<EOF > /tmp/fixed-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
  namespace: storage-test
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast-local-storage
EOF

kubectl apply -f /tmp/fixed-pvc.yaml

# Example output:
# persistentvolumeclaim/dynamic-pvc created
```

3. Recreate the pod to use the new PVC:

```bash
kubectl delete pod storage-app -n storage-test
kubectl apply -f /tmp/storage-pod.yaml

# Example output:
# pod/storage-app deleted
# pod/storage-app created
```

4. Verify that the PVC is bound and the pod is running:

```bash
kubectl get pvc -n storage-test
# Example output:
# NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
# dynamic-pvc   Bound    pvc-3f2a7a51-0e9e-48a9-a1d9-23e5d86e4b3b   1Gi        RWO            fast-local-storage  35s

kubectl get pod storage-app -n storage-test
# Example output:
# NAME          READY   STATUS    RESTARTS   AGE
# storage-app   1/1     Running   0          28s

# Examine the automatically created PV
kubectl get pv
# Example output:
# NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS        REASON   AGE
# pvc-3f2a7a51-0e9e-48a9-a1d9-23e5d86e4b3b   1Gi        RWO            Delete           Bound    storage-test/dynamic-pvc   fast-local-storage           42s

kubectl describe pv $(kubectl get pv | grep dynamic-pvc | awk '{print $1}')
# Example output includes:
# Source:
#    Type:      HostPath (bare host directory volume)
#    Path:      /var/lib/rancher/k3s/storage/pvc-3f2a7a51-0e9e-48a9-a1d9-23e5d86e4b3b_storage-test_dynamic-pvc
#    HostPathType:  DirectoryOrCreate
```

5. Test data persistence:

```bash
# Write some data to the volume
kubectl exec -n storage-test storage-app -- sh -c "echo 'This is persistent data' > /usr/share/nginx/html/index.html"

# Delete and recreate the pod
kubectl delete pod storage-app -n storage-test
kubectl apply -f /tmp/storage-pod.yaml

# Verify data persists
kubectl exec -n storage-test storage-app -- cat /usr/share/nginx/html/index.html
# Example output:
# This is persistent data
```

### Expanding Volumes
Demonstrate volume expansion capability:

```bash
# Edit the PVC to request more storage
kubectl edit pvc dynamic-pvc -n storage-test
# Change spec.resources.requests.storage from 1Gi to 2Gi

# Check the PVC status
kubectl get pvc -n storage-test
# Example output:
# NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
# dynamic-pvc   Bound    pvc-3f2a7a51-0e9e-48a9-a1d9-23e5d86e4b3b   2Gi        RWO            fast-local-storage  5m

# For local-path-provisioner, the resize might happen immediately
# Verify the new size
kubectl describe pvc dynamic-pvc -n storage-test
# Look for:
# Status:
#    Phase:          Bound
#    Capacity:       2Gi
```

### Pitfalls to Avoid in Storage Provisioning

When working with dynamic storage provisioning in Kubernetes, be aware of these common pitfalls:

1. **Mismatched Provisioner Names**: Using a provisioner name that doesn't match any installed storage driver in the cluster.
   ```
   # Error example:
   Warning  ProvisioningFailed  15s (x3 over 45s)  persistentvolume-controller  no volume plugin matched
   ```

2. **Incorrect Access Modes**: Requesting an access mode the storage provider doesn't support (e.g., ReadWriteMany for drivers that only support ReadWriteOnce).
   ```
   # Error example:
   Warning  ProvisioningFailed  30s  persistentvolume-controller  AccessMode [ReadWriteMany] isn't supported by storage class
   ```

3. **Permission Issues on Host**: The provisioner may fail to create volumes due to SELinux or insufficient permissions.
   ```
   # Error in provisioner logs:
   Failed to create directory /mnt/data/pvc-xxx: permission denied
   
   # Check SELinux status:
   $ getenforce
   Enforcing
   
   # Potential solution:
   $ sudo setenforce 0  # Temporarily disable SELinux
   # Or create proper SELinux context for the directory
   ```

4. **Volume Expansion Not Enabled**: Attempting to expand a volume when the StorageClass has `allowVolumeExpansion: false`.
   ```
   # Error example:
   error: persistentvolumeclaims "dynamic-pvc" could not be patched: volume expansion is disabled for this PVC
   ```

5. **Reclaim Policy Confusion**: Setting `Delete` as the reclaim policy means data will be lost when the PVC is deleted.
   ```
   # To change reclaim policy:
   $ kubectl patch storageclass fast-local-storage -p '{"reclaimPolicy":"Retain"}'
   ```

6. **Failing to Set StorageClassName**: If no StorageClassName is specified, the default StorageClass is used (if one exists), which might not be suitable.
   ```
   # To check default StorageClass:
   $ kubectl get sc
   NAME                   PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
   standard (default)     k8s.io/minikube-hostpath   Delete          Immediate              false                  10m
   ```

7. **Zone/Region Mismatches**: In cloud environments, creating volumes in zones where no appropriate nodes exist.
   ```
   # Error example:
   Failed to provision volume: no nodes matching the topology [topology.kubernetes.io/zone=us-east-1a]
   ```

8. **Capacity Limits**: Requesting more storage than your storage class or provider allows.
   ```
   # Error example:
   Failed to provision volume with StorageClass "gp2": volume size exceeded
   ```

9. **CSI Driver Installation Issues**: The CSI driver might not be properly installed or might have incorrect permissions.
   ```
   # Check CSI driver pods:
   $ kubectl get pods -n kube-system | grep csi
   
   # Check CSI driver logs:
   $ kubectl logs -n kube-system csi-provisioner-xyz
   ```

10. **Overlooking VolumeMounts**: Confusion between volumes and volumeMounts in pod specifications.
    ```
    # Common error: referencing non-existent PVC
    Warning  FailedMount  30s (x3 over 1m)  kubelet  MountVolume.SetUp failed: pvc "wrong-name" not found
    ```

### Example Complete Solution

Here's the full YAML solution to fix the dynamic provisioning issues:

```yaml
---
# 1. Create the correct StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-local-storage
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
allowVolumeExpansion: true
---
# 2. Create the PVC with the correct StorageClass
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
  namespace: storage-test
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast-local-storage
---
# 3. Create the Pod using the PVC
apiVersion: v1
kind: Pod
metadata:
  name: storage-app
  namespace: storage-test
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: dynamic-pvc
```

### Cross-References with Other Storage Scenarios

If you need to troubleshoot more basic storage issues related to static provisioning, refer to:
- **Scenario 8: Persistent Volume and Storage Troubleshooting** - Covers static PV/PVC binding, access mode mismatches, and volume permissions

When working with stateful applications that need storage:
- **Scenario 4: ETCD Backup and Restore** - Provides insights into managing storage for critical cluster components
- **Scenario 12: High-Availability ETCD Cluster** - Shows advanced multi-node storage configurations

## Key Exam Focus Areas to Practice

1. **Cluster Architecture and Setup**
   - Master and worker node components
   - etcd backup and restore
   - Cluster upgrades

2. **Workloads and Scheduling**
   - Deployments and pods
   - Multi-container pods
   - Resource requirements and limits
   - Taints and tolerations
   - Node/Pod affinity and anti-affinity
   - Advanced scheduling constraints

3. **Services and Networking**
   - Service DNS resolution
   - Network troubleshooting
   - CoreDNS configuration
   - NetworkPolicies and pod isolation

4. **Storage**
   - EmptyDir volumes
   - PersistentVolumes and claims
   - StorageClasses
   - Volume binding modes and access modes

5. **Security**
   - RBAC configuration
   - Service accounts and tokens
   - Kubeconfig files
   - Certificate management and rotation
   - Pod Security Standards
   - Admission Controllers

6. **Troubleshooting**
   - Node failures
   - API server issues
   - Kubelet configuration problems
   - Log analysis
   - Ephemeral debug containers
   - NetworkPolicy verification
   - Certificate and TLS issues

7. **Cluster Maintenance**
   - Node draining and cordoning
   - Component version management
   - Safely moving workloads

## Exam Tips

1. **Time Management**: Use kubectl explain and --help for syntax help rather than searching online
2. **Efficiency**: Use kubectl with -o yaml --dry-run=client > file.yaml to generate templates
3. **Shortcuts**: Set up aliases like `k=kubectl` and use tab completion
4. **Contexts**: Always check your current context with `kubectl config current-context`
5. **Documentation**: Remember you can access Kubernetes docs during the exam
6. **Imperative Commands**: Master kubectl create/run for quick resource creation
7. **Troubleshooting Flow**: Always follow systematic approach - check pods, logs, events, describe resources. Use this debugging sequence:
   ```bash
   # 1. Check pod status
   kubectl get pods [-n namespace]
   
   # 2. Check pod details
   kubectl describe pod <pod-name> [-n namespace]
   
   # 3. Check logs
   kubectl logs <pod-name> [-c container-name] [-n namespace]
   
   # 4. Check events
   kubectl get events --sort-by='.lastTimestamp' [-n namespace]
   
   # 5. Use enhanced debugging for running workloads
   kubectl debug <pod-name> --image=nicolaka/netshoot -it --profile=troubleshooting
   
   # For specific container targeting
   kubectl debug <pod-name> --target=<container-name> --image=busybox -it
   
   # For resource analysis
   kubectl debug <pod-name> --image=nicolaka/netshoot --profile=resource-monitor
   ```