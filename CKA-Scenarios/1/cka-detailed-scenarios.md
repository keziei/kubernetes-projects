# Kezie Iroha 
# Certified Kubernetes Administrator (CKA) Exam Preparation Scenarios
# Based on Kubernetes 1.32 Features

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
17. [Advanced Debugging with Ephemeral Containers](#scenario-17-advanced-debugging-with-ephemeral-containers)
18. [Appendix: Emergency Commands Reference](#appendix-emergency-commands-reference)
19. [Common Diagnostic Steps](#common-diagnostic-steps)
20. [CKA Exam Quick Start](#cka-exam-quick-start)

## Common Troubleshooting Patterns

Before diving into specific scenarios, here are fundamental troubleshooting patterns that apply across multiple Kubernetes issues:

### 1. The Kubernetes Troubleshooting Workflow

```
┌────────────────┐     ┌────────────────┐     ┌────────────────┐     ┌────────────────┐
│  1. OBSERVE    │────>│  2. DIAGNOSE   │────>│  3. REMEDIATE  │────>│  4. VALIDATE   │
└────────────────┘     └────────────────┘     └────────────────┘     └────────────────┘
    - Get status         - Check logs           - Edit configs         - Check status
    - List resources     - Describe objects     - Restart services     - Test functionality
    - Check events       - Inspect configs      - Scale/recreate       - Monitor logs
```

### 2. Resource-Specific Troubleshooting Commands

| Resource Type | Observation | Diagnosis | Common Issues |
|---------------|-------------|-----------|---------------|
| **Nodes** | `kubectl get nodes` | `kubectl describe node <name>` <br> `kubectl top node` | NotReady status, kubelet issues, resource pressure |
| **Pods** | `kubectl get pods -o wide` | `kubectl describe pod <name>` <br> `kubectl logs <pod>` | CrashLoopBackOff, ImagePullBackOff, Pending status |
| **Services** | `kubectl get svc` | `kubectl describe svc <name>` <br> `kubectl get endpoints <name>` | No endpoints, selector issues, port mismatches |
| **Deployments** | `kubectl get deploy` | `kubectl describe deploy <name>` <br> `kubectl rollout status deploy <name>` | Unavailable replicas, failed conditions, strategy issues |
| **Storage** | `kubectl get pv,pvc` | `kubectl describe pv/pvc <name>` | Pending binding, access mode mismatches, wrong storage class |
| **Networking** | `kubectl get netpol` | `kubectl describe netpol <name>` | Overly restrictive policies, ingress/egress rules |

### 3. Log Location Reference

| Component | Log Location | Command |
|-----------|-------------|---------|
| kubelet | `/var/log/kubelet.log` | `journalctl -u kubelet` |
| API Server | container logs | `kubectl logs -n kube-system kube-apiserver-<master>` |
| Controller Manager | container logs | `kubectl logs -n kube-system kube-controller-manager-<master>` |
| Scheduler | container logs | `kubectl logs -n kube-system kube-scheduler-<master>` |
| etcd | container logs | `kubectl logs -n kube-system etcd-<master>` |
| Container Runtime | varies | `journalctl -u containerd` or `crictl logs` |
| System | `/var/log/syslog` or `/var/log/messages` | `journalctl` |

### 4. Critical Configuration Files

| Component | Config Location | Purpose |
|-----------|-----------------|---------|
| kubelet | `/var/lib/kubelet/config.yaml` | Kubelet configuration |
| kubeadm | `/etc/kubernetes/admin.conf` | Cluster admin configuration |
| API Server | `/etc/kubernetes/manifests/kube-apiserver.yaml` | Static pod manifest |
| Controller Manager | `/etc/kubernetes/manifests/kube-controller-manager.yaml` | Static pod manifest |
| Scheduler | `/etc/kubernetes/manifests/kube-scheduler.yaml` | Static pod manifest |
| etcd | `/etc/kubernetes/manifests/etcd.yaml` | Static pod manifest |
| PKI | `/etc/kubernetes/pki/` | Certificate files |

## Document Purpose and Curriculum Mapping

This document contains hands-on scenarios designed to prepare candidates for the Certified Kubernetes Administrator (CKA) exam. Each scenario maps to specific domains in the official CKA curriculum and includes fault injection, diagnostic steps, and remediation procedures.

### CKA Exam Domains Coverage

| Scenario | Primary CKA Domain | Secondary Domain | Estimated Time* |
|----------|-------------------|------------------|----------------|
| 1. Cluster Configuration and API Server Failure | Cluster Architecture, Installation & Configuration | Troubleshooting | 15-25 min |
| 2. Networking, Services, and DNS Troubleshooting | Services & Networking | Troubleshooting | 15-25 min |
| 3. Cluster Node Failure and Recovery | Cluster Maintenance | Troubleshooting | 15-25 min |
| 4. ETCD Backup and Restore | Cluster Maintenance | Disaster Recovery | 15-25 min |
| 5. Multi-Container Pod Troubleshooting | Workloads & Scheduling | Resource Management | 15-25 min |
| 6. RBAC, Authentication, and Authorization | Security | Access Control | 15-25 min |
| 7. Cluster Upgrade and Component Failure | Cluster Maintenance | Troubleshooting | 20-30 min |
| 8. Persistent Volume and Storage Troubleshooting | Storage | Troubleshooting | 15-25 min |
| 9. Network Policy and Pod Isolation | Services & Networking | Security | 15-25 min |
| 10. Advanced Scheduling with Taints/Tolerations | Workloads & Scheduling | Advanced Scheduling | 15-25 min |
| 11. Ingress Controller and TLS Configuration | Services & Networking | Security | 20-30 min |
| 12. High-Availability ETCD Cluster | Cluster Architecture | Disaster Recovery | 20-30 min |
| 13. Certificate Rotation and TLS Troubleshooting | Security | Troubleshooting | 15-25 min |
| 14. Admission Controllers and Pod Security | Security | Troubleshooting | 15-25 min |
| 15. Combined Mock Exam Scenario | Multiple Domains | Troubleshooting | 30-45 min |
| 16. Dynamic Storage Provisioning with CSI | Storage | Troubleshooting | 20-30 min |
| 17. Advanced Debugging with Ephemeral Containers | Troubleshooting | Workloads & Scheduling | 15-25 min |

*Time estimates can vary significantly based on your experience level and familiarity with the concepts.

**Note on Kubernetes Version**: These scenarios are updated for Kubernetes 1.32, which includes major enhancements to troubleshooting capabilities, certificate management, and cluster diagnostics compared to previous versions.

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

### YAML Validation Tip

When working with YAML manifests, always validate your syntax before applying:

```bash
# Validate YAML without applying
kubectl apply --dry-run=client --validate=true -f your-manifest.yaml

# For quick validation with multiple resources
for file in *.yaml; do
  echo "Validating $file..."
  kubectl apply --dry-run=client --validate=true -f $file || echo "Error in $file"
done
```

## Scenario 1: Cluster Configuration and API Server Failure

### Key Learning Objectives
- Diagnose issues with the Kubernetes API server
- Understand the static pod manifests in `/etc/kubernetes/manifests/`
- Troubleshoot authorization mode configuration errors
- Use system logs to identify control plane failures
- Learn the recovery process for API server failures

### Problem Statement
Your organization's Kubernetes API server has stopped responding. Users are unable to interact with the cluster, and applications cannot be deployed. Your task is to diagnose and resolve the issue.

```
Healthy Cluster                          Broken Cluster
┌─────────────────┐                     ┌─────────────────┐
│                 │                     │                 │
│  API Server ✓   │──────────► ✗       │  API Server ✗   │──┐
│                 │                     │                 │  │
└─────────────────┘                     └─────────────────┘  │
        │                                       │            │
        │                                       │            │
        ▼                                       ▼            │
┌─────────────────┐                     ┌─────────────────┐  │
│                 │                     │                 │  │
│  ETCD      ✓    │                     │  ETCD      ✓    │  │
│                 │                     │                 │  │
└─────────────────┘                     └─────────────────┘  │
        │                                       │            │
        │                                       │            │
        ▼                                       ▼            │
┌─────────────────┐                     ┌─────────────────┐  │
│  Other Control  │                     │  Other Control  │  │
│  Plane    ✓     │◄───────────────────│  Plane    ✓     │◄─┘
│                 │                     │                 │
└─────────────────┘                     └─────────────────┘
```

> **Relevant Official Documentation**:
> - [Kubernetes API Server](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)
> - [Troubleshooting Clusters](https://kubernetes.io/docs/tasks/debug/debug-cluster/)

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

5. If necessary, use enhanced debug functionality:
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

4. Verify recovery:
   ```bash
   # Wait for API server to start
   watch -n1 kubectl get nodes
   ```

### Quick Reference for API Server Issues

| Issue | Symptoms | Quick Fix |
|-------|----------|-----------|
| Misconfigured manifest | API server pod not starting, kubelet logs show manifest errors | Fix YAML syntax in `/etc/kubernetes/manifests/kube-apiserver.yaml` |
| Certificate issues | API server logs show TLS errors | Verify certificate paths in manifest, check certs in `/etc/kubernetes/pki/` |
| Authorization issues | 403 errors, Permission denied | Fix `--authorization-mode` flag in API server manifest |
| etcd connectivity | API server logs with etcd connection errors | Verify etcd endpoints in `--etcd-servers` |
| Resource constraints | API server pod evicted or pending | Check node resources, adjust requests/limits |

When the API server is down:
1. ✅ First check kubelet status: `systemctl status kubelet`
2. ✅ Check kubelet logs: `journalctl -u kubelet | grep apiserver`
3. ✅ Examine API server manifest: `/etc/kubernetes/manifests/kube-apiserver.yaml`
4. ✅ Check container runtime for API server container status: `crictl ps | grep apiserver`

For more details, see: [Kubernetes API Server Recovery Guide](https://kubernetes.io/docs/tasks/debug/debug-cluster/troubleshoot-apiserver/)

### Do Not Forget (Key Takeaways)
- ✅ API Server issues often involve manifest errors or certificate problems
- ✅ Check kubelet logs first when the API Server is down (`journalctl -u kubelet`)
- ✅ API Server manifest is at `/etc/kubernetes/manifests/kube-apiserver.yaml`
- ✅ Always check resource usage with `kubectl top nodes` if scheduling seems stuck
- ✅ Use `crictl` commands when `kubectl` doesn't work due to API Server issues

## Scenario 2: Networking, Services, and DNS Troubleshooting

### Key Learning Objectives
- Diagnose CoreDNS configuration issues
- Understand DNS resolution in Kubernetes pods
- Learn how ConfigMaps affect CoreDNS behavior
- Use targeted debugging to isolate network services issues
- Apply configuration changes to restore DNS functionality

### Problem Statement
Applications are reporting DNS resolution failures and connectivity issues between pods. Your task is to diagnose and resolve the network infrastructure problems.

> **Relevant Official Documentation**:
> - [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
> - [Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)

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

### Do Not Forget (Key Takeaways)
- ✅ DNS issues often involve CoreDNS ConfigMap errors or pod failures
- ✅ Always check DNS resolution from inside a test pod (`kubectl exec -it <pod> -- nslookup kubernetes.default`)
- ✅ Verify CoreDNS pods are running (`kubectl get pods -n kube-system -l k8s-app=kube-dns`)
- ✅ Check pod DNS configuration in `/etc/resolv.conf` for correct nameserver
- ✅ For DNS troubleshooting, see [Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)

## Scenario 3: Cluster Node Failure and Recovery

### Key Learning Objectives
- Identify and troubleshoot node failures
- Master the node drainage process for maintenance
- Understand kubelet configuration and its impact on node health
- Learn proper procedures for removing nodes from a cluster
- Verify workload rescheduling after node recovery

### Problem Statement
One of your worker nodes has become unresponsive and is not accepting new pod deployments. Your task is to diagnose the issue, recover the node if possible, or safely evict and reschedule pods if necessary.

> **Relevant Official Documentation**:
> - [Safely Drain a Node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)
> - [Troubleshooting Clusters](https://kubernetes.io/docs/tasks/debug/debug-cluster/)

### Fault Injection
Execute on k8s-worker1:
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
   # The k8s-worker1 node will show as NotReady
   ```

2. Examine pods affected by the node failure:
   ```bash
   kubectl get pods --all-namespaces -o wide | grep k8s-worker1
   ```

3. Check node description for events:
   ```bash
   kubectl describe node k8s-worker1
   ```

4. SSH to the problematic node and check kubelet status:
   ```bash
   ssh k8s-worker1
   sudo systemctl status kubelet
   sudo journalctl -u kubelet | tail -n 50
   ```

5. Check for missing configuration files:
   ```bash
   sudo ls -la /etc/kubernetes/
   ```

### Remediation
1. Restore kubelet configuration on k8s-worker1:
   ```bash
   sudo mv /etc/kubernetes/kubelet.conf.bak /etc/kubernetes/kubelet.conf
   sudo systemctl restart kubelet
   ```

2. If node recovery isn't possible, safely drain the node:
   ```bash
   kubectl drain k8s-worker1 --ignore-daemonsets --delete-emptydir-data
   ```

3. If the node is permanently unavailable, remove it from the cluster:
   ```bash
   kubectl delete node k8s-worker1
   ```

4. Check pod rescheduling to other nodes:
   ```bash
   kubectl get pods --all-namespaces -o wide
   ```

5. Verify node recovery:
   ```bash
   kubectl get nodes
   # k8s-worker1 should show as Ready
   ```

### Do Not Forget (Key Takeaways)
- ✅ Node NotReady state is often caused by kubelet configuration issues
- ✅ Always check kubelet status first (`systemctl status kubelet`)
- ✅ For persistent node failures, safely drain the node (`kubectl drain <node> --ignore-daemonsets`)
- ✅ For nodes that won't recover, remove them from the cluster (`kubectl delete node <node>`)
- ✅ If you see pods stuck on a failed node, see [Force Delete Pods](https://kubernetes.io/docs/tasks/run-application/force-delete-stateful-set-pod/)

## Scenario 4: ETCD Backup and Restore

### Key Learning Objectives
- Create and verify etcd backups
- Restore a cluster from an etcd snapshot
- Understand the etcd data directory structure
- Learn proper backup verification procedures
- Master the restore process for disaster recovery

### Problem Statement
A critical mistake has been made, resulting in the accidental deletion of important deployments and services. You need to restore the etcd backup to recover the cluster state.

> **Relevant Official Documentation**:
> - [Operating etcd clusters for Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
> - [Backing up an etcd cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)

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

### Do Not Forget (Key Takeaways)
- ✅ Always back up etcd before making cluster changes (`etcdctl snapshot save`)
- ✅ Verify backup integrity (`etcdctl snapshot status`)
- ✅ Stop API server before restoring etcd (move manifests out of `/etc/kubernetes/manifests/`)
- ✅ After restore, ensure proper ownership (`chown -R etcd:etcd /var/lib/etcd`)
- ✅ For detailed etcd operations, see [Operating etcd clusters](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

## Scenario 5: Multi-Container Pod Troubleshooting and Resource Management

### Key Learning Objectives
- Diagnose issues with multi-container pods
- Understand resource constraints and their impact
- Troubleshoot container startup dependencies
- Learn to fix container image and command issues
- Master volume sharing between containers

### Problem Statement
A critical application involves a multi-container pod with a sidecar logs collector. The pod is not starting properly due to resource constraints and container dependencies. Your task is to diagnose and fix the issues.

> **Relevant Official Documentation**:
> - [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
> - [Managing Resources for Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

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

### Do Not Forget (Key Takeaways)
- ✅ Check pod status with `kubectl describe pod` to see specific container issues
- ✅ Review container logs for each container (`kubectl logs pod-name -c container-name`)
- ✅ For CrashLoopBackOff, check the previous container logs (`kubectl logs pod-name -c container-name --previous`)
- ✅ Verify resource constraints with `kubectl top pods` and compare to limits/requests
- ✅ For advanced container debugging, see [Scenario 17: Advanced Debugging with Ephemeral Containers](#scenario-17-advanced-debugging-with-ephemeral-containers)

## Scenario 6: RBAC, Authentication, and Authorization

### Key Learning Objectives
- Configure and troubleshoot Role-Based Access Control (RBAC)
- Understand ServiceAccounts and their relationship to RBAC
- Diagnose permission issues in Kubernetes
- Create and manage Roles and RoleBindings
- Generate kubeconfig files for different users

### Problem Statement
A new developer needs access to the cluster but with limited permissions. However, they're reporting that they can't access any resources. Your task is to diagnose and fix the RBAC configuration issue.

> **Relevant Official Documentation**:
> - [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
> - [Configure Service Accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

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

### Do Not Forget (Key Takeaways)
- ✅ Verify permissions with `kubectl auth can-i` before creating resources
- ✅ Understand the difference between Role (namespace) and ClusterRole (cluster-wide)
- ✅ Check Role and RoleBinding are in the same namespace
- ✅ The `roleRef` in RoleBinding must point to a valid Role/ClusterRole
- ✅ For comprehensive RBAC guidance, see [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

## Scenario 7: Cluster Upgrade and Component Failure

### Key Learning Objectives
- Plan and execute Kubernetes version upgrades
- Troubleshoot component version mismatches
- Handle node draining and cordoning during upgrades
- Manage workload availability during cluster maintenance
- Recover from upgrade failures

### Problem Statement
Your organization requires an upgrade of the Kubernetes cluster from v1.31 to v1.32. During the upgrade, you encounter component failures that need to be addressed while maintaining cluster functionality.

> **Note**: This scenario uses the latest Kubernetes version (v1.32), which includes enhanced upgrade mechanisms and streamlined component management.

> **Relevant Official Documentation**:
> - [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
> - [Safely Drain a Node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)

### Fault Injection
Simulate a problematic upgrade scenario:
```bash
# First, create some workloads to maintain during upgrade
kubectl create deployment important-app --image=nginx --replicas=3
kubectl create service clusterip important-app --tcp=80:80

# Simulate component version mismatch on k8s-worker2
ssh k8s-worker2 "sudo sed -i 's/v1.31/v1.30/' /etc/kubernetes/kubelet.conf"
ssh k8s-worker2 "sudo systemctl restart kubelet"

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
   kubectl describe node k8s-worker2
   ```

3. Check kubelet configuration on k8s-worker2:
   ```bash
   ssh k8s-worker2 "sudo systemctl status kubelet"
   ssh k8s-worker2 "sudo journalctl -u kubelet | tail -n 50"
   ssh k8s-worker2 "sudo cat /etc/kubernetes/kubelet.conf"
   ```

4. Check the kubelet service configuration on master:
   ```bash
   ssh k8s-master "sudo ls -la /etc/systemd/system/kubelet.service.d/"
   ssh k8s-master "sudo cat /usr/lib/systemd/system/kubelet.service"
   ```

### Remediation
1. Fix kubelet configuration on k8s-worker2:
   ```bash
   ssh k8s-worker2 "sudo sed -i 's/v1.30/v1.32/' /etc/kubernetes/kubelet.conf"
   ssh k8s-worker2 "sudo systemctl restart kubelet"
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
   # For k8s-worker1
   kubectl drain k8s-worker1 --ignore-daemonsets
   ssh k8s-worker1 "sudo kubeadm upgrade node"
   ssh k8s-worker1 "sudo dnf upgrade -y kubelet"
   ssh k8s-worker1 "sudo systemctl restart kubelet"
   kubectl uncordon k8s-worker1
   
   # For k8s-worker2
   kubectl drain k8s-worker2 --ignore-daemonsets
   ssh k8s-worker2 "sudo kubeadm upgrade node"
   ssh k8s-worker2 "sudo dnf upgrade -y kubelet"
   ssh k8s-worker2 "sudo systemctl restart kubelet"
   kubectl uncordon k8s-worker2
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

### Do Not Forget (Key Takeaways)
- ✅ Always run `kubeadm upgrade plan` before upgrading
- ✅ Upgrade control plane components before worker nodes
- ✅ Drain nodes before upgrading to prevent workload disruption
- ✅ Uncordon nodes after upgrade to resume scheduling
- ✅ Version skew between components should be no more than -1/+1 (for example, if API server is v1.32, kubelet can be v1.31-v1.32)
- ✅ For upgrade guidance, see [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

## Scenario 8: Persistent Volume and Storage Troubleshooting

### Key Learning Objectives
- Configure and troubleshoot Persistent Volumes and Persistent Volume Claims
- Understand storage provisioning and binding modes
- Diagnose access mode mismatches between PVs and PVCs
- Resolve storage permissions issues on host nodes
- Configure storage for stateful applications

### Problem Statement
A stateful application deployment is failing because its PersistentVolumeClaims are stuck in a Pending state. Your task is to diagnose and resolve storage configuration issues preventing the application from starting.

> **Note**: For dynamic provisioning with StorageClasses and CSI drivers, see [Scenario 16: Dynamic Storage Provisioning with CSI Drivers](#scenario-16-dynamic-storage-provisioning-with-csi-drivers).

> **Relevant Official Documentation**:
> - [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
> - [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

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
          - k8s-worker1
EOF

kubectl apply -f /tmp/local-pv.yaml

# Create directory on k8s-worker1 with incorrect permissions
ssh k8s-worker1 "sudo mkdir -p /mnt/data && sudo chmod 700 /mnt/data"

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
   ssh k8s-worker1 sudo getenforce
   # If it returns "Enforcing", it might be blocking volume access
   
   # Temporarily set to permissive for testing
   ssh k8s-worker1 sudo setenforce 0
   
   # For a permanent fix, create the correct SELinux context
   ssh k8s-worker1 sudo chcon -Rt svirt_sandbox_file_t /mnt/data
   
   # For RHEL/AlmaLinux environments, check contexts more thoroughly
   ssh k8s-worker1 "sudo ls -lZ /mnt/data"
   
   # Apply the container file context policy
   ssh k8s-worker1 "sudo semanage fcontext -a -t container_file_t '/mnt/data(/.*)?'"
   ssh k8s-worker1 "sudo restorecon -Rv /mnt/data"
   ```

2. **Directory permissions**:
   ```bash
   # Check directory permissions
   ssh k8s-worker1 ls -la /mnt/data
   
   # Fix permissions if needed - typical container UIDs need read/write
   ssh k8s-worker1 sudo chmod 755 /mnt/data
   ```

3. **AppArmor/SecComp profiles** (if enabled):
   ```bash
   # Check if profiles are loaded
   ssh k8s-worker1 sudo aa-status
   
   # Check Docker/containerd configuration for default profiles
   ssh k8s-worker1 cat /etc/docker/daemon.json
   ```

4. **Disk space issues**:
   ```bash
   # Check available disk space
   ssh k8s-worker1 df -h
   
   # Check inode usage (sometimes overlooked)
   ssh k8s-worker1 df -i
   ```

These OS-level checks are especially important for local storage provisioners.

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
             - k8s-worker1
   EOF
   
   kubectl apply -f /tmp/fixed-local-pv.yaml
   ```

2. Fix directory permissions on the worker node:
   ```bash
   ssh k8s-worker1 "sudo chmod 755 /mnt/data"
   ssh k8s-worker1 "sudo chown 999:999 /mnt/data"  # 999 is the postgres user in the container
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

### Verification
```bash
# Verify the fixed PV
kubectl get pv

# Verify the PVC is bound
kubectl get pvc

# Verify the pod is running
kubectl get pod database-0
```

### Quick Reference for Storage Troubleshooting

| Issue | Symptoms | Quick Fix |
|-------|----------|-----------|
| PV/PVC access mode mismatch | PVC stuck in Pending | Ensure PV and PVC access modes match (RWO, RWX, ROX) |
| Storage class not found | ProvisioningFailed events | Create or correct StorageClass name |
| Directory permissions | FailedMount, Permission denied | Check host directory permissions (`chmod 755`) |
| Wrong path on host | FailedMount, directory not found | Verify hostPath exists on worker node |
| SELinux issues | Permission denied despite correct ownership | Check SELinux context or set to permissive |

When troubleshooting PV/PVC issues:
1. ✅ Check PVC status: `kubectl get pvc`
2. ✅ Check PV status: `kubectl get pv`
3. ✅ Check PVC events: `kubectl describe pvc <name>`
4. ✅ Look for FailedMount events in pod: `kubectl describe pod <name>`
5. ✅ For hostPath volumes, verify directory permissions on node

For SELinux environments, check context:
```bash
# Check SELinux status
ssh k8s-worker1 "getenforce"

# Add correct SELinux context
ssh k8s-worker1 "sudo chcon -Rt svirt_sandbox_file_t /mnt/data"
```

For more details, see: [Kubernetes Volumes documentation](https://kubernetes.io/docs/concepts/storage/volumes/)

### Do Not Forget (Key Takeaways)
- ✅ PVC Pending status usually means PV with matching criteria not found
- ✅ Verify access modes match between PV and PVC (RWO, ROX, RWX)
- ✅ Check storage class exists and is correctly defined
- ✅ For hostPath volumes, verify directory exists and has proper permissions
- ✅ For SELinux environments, check and set proper contexts 
- ✅ For CSI and dynamic provisioning, see [Scenario 16: Dynamic Storage Provisioning with CSI Drivers](#scenario-16-dynamic-storage-provisioning-with-csi-drivers)

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

> **Relevant Official Documentation**:
> - [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
> - [Declare Network Policy](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)

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
1. Check the frontend pod logs for connection errors:
   ```bash
   kubectl get pods -n frontend
   FRONTEND_POD=$(kubectl get pod -n frontend -l app=frontend -o jsonpath='{.items[0].metadata.name}')
   kubectl logs -n frontend $FRONTEND_POD
   # Should show connection refused or timeout errors
   
   # Use the Kubernetes 1.32 network diagnostics tool
   kubectl network diagnose pod/$FRONTEND_POD -n frontend --target-service backend-svc.backend
   
   # Use enhanced debugging with network tools
   kubectl debug -n frontend $FRONTEND_POD --image=nicolaka/netshoot -it --profile=network -- /bin/bash
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

### Do Not Forget (Key Takeaways)
- ✅ NetworkPolicies are namespace-scoped and use labels for selection
- ✅ Default behavior without NetworkPolicies allows all traffic
- ✅ Test connectivity with temporary pods: `kubectl run test --image=busybox -- sleep 3600`
- ✅ Use ephemeral debug containers for network diagnostics (see [Scenario 17](#scenario-17-advanced-debugging-with-ephemeral-containers))
- ✅ For comprehensive NetworkPolicy design, see [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

## Scenario 10: Advanced Scheduling with Taints, Tolerations, and Affinities

### Key Learning Objectives
- Configure and troubleshoot taints and tolerations for node selection
- Understand pod and node affinity/anti-affinity rules
- Use advanced scheduling constraints to distribute workloads
- Diagnose pod scheduling failures
- Implement high-availability deployment patterns using topology constraints

### Problem Statement
A high-priority application needs to be scheduled with specific node placement requirements, but the pods are either not scheduling or landing on the wrong nodes. Your task is to diagnose scheduling constraints and ensure proper workload placement.

> **Relevant Official Documentation**:
> - [Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
> - [Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

### Fault Injection
Set up a cluster with taints and scheduling constraints:
```bash
# Taint k8s-worker1 to repel regular workloads
kubectl taint nodes k8s-worker1 dedicated=high-priority:NoSchedule

# Taint k8s-worker2 with a taint that has no matching toleration
kubectl taint nodes k8s-worker2 environment=production:NoSchedule

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
   kubectl describe nodes k8s-worker1 | grep Taint
   kubectl describe nodes k8s-worker2 | grep Taint
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
                   - k8s-worker1
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
   # Pods should now be scheduled on k8s-worker1
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
   # Pods should be distributed across k8s-worker1 and k8s-worker2 for HA
   ```

### Do Not Forget (Key Takeaways)
- ✅ Taints prevent pods from scheduling unless they have matching tolerations
- ✅ NodeAffinity controls which nodes a pod can schedule to (based on node labels)
- ✅ PodAffinity/Anti-affinity control pod placement relative to other pods
- ✅ Check unschedulable pods with `kubectl get pods` (Pending) and `kubectl describe pod` (events)
- ✅ For scheduling in depth, see [Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

## Scenario 11: Ingress Controller and TLS Configuration Troubleshooting

### Key Learning Objectives
- Configure and troubleshoot Ingress resources
- Set up TLS termination for secure application access
- Diagnose common Ingress connectivity issues
- Understand how backend services connect to Ingress resources
- Test and verify Ingress routing and TLS functionality

### Problem Statement
Your organization has deployed an Ingress controller to expose applications externally. However, users are reporting connectivity issues and TLS errors when accessing applications. Your task is to diagnose and fix the Ingress configuration while ensuring proper TLS termination.

> **Relevant Official Documentation**:
> - [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
> - [TLS for Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls)

### Fault Injection
Set up a misconfigured Ingress controller and resources:
```bash
# First, deploy the NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/baremetal/deploy.yaml

# Create a namespace for the application
kubectl create namespace webapp

# Deploy a simple web application
cat <<EOF > /tmp/webapp.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-svc
  namespace: webapp
spec:
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 80
EOF

kubectl apply -f /tmp/webapp.yaml

# Create a self-signed certificate with incorrect Common Name
mkdir -p /tmp/certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /tmp/certs/tls.key -out /tmp/certs/tls.crt \
  -subj "/CN=wrong-domain.example.com"

# Create TLS secret
kubectl create secret tls -n webapp webapp-tls \
  --key=/tmp/certs/tls.key \
  --cert=/tmp/certs/tls.crt

# Create Ingress with incorrect configuration
cat <<EOF > /tmp/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  namespace: webapp
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - webapp.example.com
    secretName: nonexistent-tls  # Incorrect secret name
  rules:
  - host: "webapp.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-svc
            port:
              number: 8080  # Incorrect port number
EOF

kubectl apply -f /tmp/ingress.yaml
```

### Diagnostic Steps
1. Check the Ingress controller pods:
   ```bash
   kubectl get pods -n ingress-nginx
   
   # Example output:
   # NAME                                        READY   STATUS    RESTARTS   AGE
   # ingress-nginx-controller-5c8d66c76d-zk8mh   1/1     Running   0          45s
   
   kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller
   
   # Look for error messages like:
   # W0303 12:34:56.789012       1 controller.go:995] Service "webapp/webapp-svc" does not have any active Endpoint
   # E0303 12:34:56.789012       1 controller.go:217] SSL certificate "webapp/nonexistent-tls" does not exist
   ```

2. Check the Ingress resource:
   ```bash
   kubectl get ingress -n webapp
   
   # Example output:
   # NAME             CLASS   HOSTS                ADDRESS   PORTS     AGE
   # webapp-ingress   nginx   webapp.example.com             80, 443   2m
   
   kubectl describe ingress webapp-ingress -n webapp
   
   # Example output showing issues:
   # TLS:
   #   webapp.example.com terminates nonexistent-tls
   # Events:
   #   Type     Reason   Age                From     Message
   #   ----     ------   ----               ----     -------
   #   Warning  Rejected 12s (x3 over 20s)  nginx-ingress-controller  Error obtaining X.509 certificate: secret nonexistent-tls not found
   ```

3. Verify the services configuration:
   ```bash
   kubectl get svc -n webapp
   
   # Example output:
   # NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
   # webapp-svc   ClusterIP   10.96.131.45    <none>        80/TCP    10m
   
   kubectl describe svc webapp-svc -n webapp
   
   # Look for port configuration - note the service has port 80, not 8080:
   # Port:              <unset>  80/TCP
   # TargetPort:        80/TCP
   ```

4. Check TLS secret:
   ```bash
   kubectl get secret -n webapp
   
   # Example output:
   # NAME          TYPE                DATA   AGE
   # webapp-tls    kubernetes.io/tls   2      15m
   
   kubectl describe secret webapp-tls -n webapp
   
   # Note that the secret exists but is not being referenced correctly in the ingress
   ```

5. Test connectivity to the service directly:
   ```bash
   # Create a temporary pod to test service connectivity
   kubectl run tmp-shell -n webapp --rm -i --tty --image nicolaka/netshoot -- bash
   
   # Inside the pod:
   curl webapp-svc
   
   # You should see the Nginx welcome page if service is working
   ```

### Remediation
1. Fix the Ingress configuration:
   ```bash
   cat <<EOF > /tmp/fixed-ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: webapp-ingress
     namespace: webapp
     annotations:
       nginx.ingress.kubernetes.io/ssl-redirect: "true"
   spec:
     ingressClassName: nginx
     tls:
     - hosts:
         - webapp.example.com
       secretName: webapp-tls  # Correct secret name
     rules:
     - host: "webapp.example.com"
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: webapp-svc
               port:
                 number: 80  # Correct port number
   EOF

   kubectl apply -f /tmp/fixed-ingress.yaml
   ```

2. Create a new certificate with the correct Common Name:
   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout /tmp/certs/tls.key -out /tmp/certs/tls.crt \
     -subj "/CN=webapp.example.com"

   kubectl delete secret -n webapp webapp-tls
   kubectl create secret tls -n webapp webapp-tls \
     --key=/tmp/certs/tls.key \
     --cert=/tmp/certs/tls.crt
   ```

3. Verify the Ingress is properly configured:
   ```bash
   kubectl get ingress -n webapp
   
   # Example output after fix:
   # NAME             CLASS   HOSTS                ADDRESS        PORTS     AGE
   # webapp-ingress   nginx   webapp.example.com   192.168.49.2   80, 443   30m
   
   kubectl describe ingress webapp-ingress -n webapp
   
   # Events should no longer show errors
   ```

4. Test the connectivity and TLS configuration:
   ```bash
   # Get the Ingress Controller service IP/port
   kubectl get svc -n ingress-nginx
   
   # Verify using curl (if accessible)
   INGRESS_IP=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
   if [ -z "$INGRESS_IP" ]; then
     INGRESS_IP=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.spec.clusterIP}')
   fi
   INGRESS_PORT=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
   
   # Test with curl (add --resolve for hostname resolution)
   echo "Testing with curl: $INGRESS_IP:$INGRESS_PORT"
   curl -k -v --resolve webapp.example.com:$INGRESS_PORT:$INGRESS_IP https://webapp.example.com:$INGRESS_PORT
   
   # You should see TLS handshake details and the nginx welcome page
   ```

5. Troubleshoot any remaining issues:
   ```bash
   # Check Ingress controller logs for any persistent issues
   kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller
   
   # Verify backend endpoint connectivity
   kubectl get endpoints -n webapp
   ```

### Common Issues and Solutions
1. **TLS Handshake Errors**: Often due to certificate/hostname mismatch
   ```
   # Error in logs:
   SSL error: error:14094412:SSL routines:ssl3_read_bytes:sslv3 alert bad certificate
   ```
   Solution: Ensure certificate CN matches the hostname in the Ingress

2. **404 Not Found**: Usually due to incorrect service name or port
   ```
   # Error in browser:
   404 Not Found
   
   # Error in ingress-controller logs:
   [error] 19#19: *123 upstream not found while connecting to upstream, client: 10.244.0.1, server: webapp.example.com
   ```
   Solution: Verify service name and port in Ingress backend definition

3. **502 Bad Gateway**: Backend service unreachable or not running
   ```
   # Error in browser:
   502 Bad Gateway
   
   # Error in ingress-controller logs:
   [error] 19#19: *123 connect() failed (111: Connection refused) while connecting to upstream
   ```
   Solution: Ensure backend pods are running and service selector matches pod labels

4. **Ingress not getting an address**: Check Ingress controller deployment and service
   ```
   # Check status:
   kubectl get ingress -A
   
   # If ADDRESS column is empty, check:
   kubectl get pods -n ingress-nginx
   kubectl get svc -n ingress-nginx
   ```
   Solution: Verify Ingress controller is properly deployed and has valid service

### Additional Tips for the Exam
- Always check both the Ingress resource and the backend service
- Verify TLS certificates match the hostnames in the Ingress
- Use debug pods (nicolaka/netshoot) to test connectivity from inside the cluster
- Remember port numbers must match between Ingress and Service

### Do Not Forget (Key Takeaways)
- ✅ Ingress requires a running Ingress controller (not included in default Kubernetes)
- ✅ Verify TLS certificate matches the hostname in Ingress rules
- ✅ Check backend service port numbers match the Ingress configuration
- ✅ For TLS issues, verify the Secret exists and contains valid certificate/key
- ✅ For detailed Ingress guide, see [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

## Scenario 12: High-Availability ETCD Cluster Troubleshooting

### Key Learning Objectives
- Understand ETCD high-availability architecture and quorum requirements
- Diagnose etcd cluster member health issues
- Know how to safely remove and add etcd members
- Use etcdctl to verify cluster health and handle quorum failures
- Learn proper backup procedures for multi-node etcd

### Problem Statement
Your organization has a high-availability Kubernetes cluster with a multi-node etcd setup. One of the etcd nodes has failed, causing cluster instability. Your task is to diagnose the etcd cluster health, remove the failed member, and add a replacement node.

> **Relevant Official Documentation**:
> - [Operating etcd clusters for Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
> - [Set up a High-Availability etcd cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/)

### Fault Injection
Note: This simulation assumes you have access to the etcd certificates on the master node. We'll simulate the problem using the existing single-node etcd:
```bash
# Backup etcd first
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /tmp/etcd-snapshot.db

# Simulate a failed etcd member by adding a fake member
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  member add etcd-failed https://192.168.100.100:2380
```

### Diagnostic Steps
1. Check the etcd cluster status:
   ```bash
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     member list -w table
   
   # Example output showing the problem:
   # +------------------+---------+--------+----------------------------+----------------------------+------------+
   # |        ID        | STATUS  |  NAME  |         PEER ADDRS         |        CLIENT ADDRS        | IS LEARNER |
   # +------------------+---------+--------+----------------------------+----------------------------+------------+
   # | 8211f1d0f64f3269 | started | master | https://172.16.8.2:2380    | https://172.16.8.2:2379    |      false |
   # | a3dea766690c24f0 | started | k8s-worker1| https://172.16.8.3:2380    | https://172.16.8.3:2379    |      false |
   # | f4ea13541bc89ce2 | unstarted | etcd-failed | https://192.168.100.100:2380 |                             |      false |
   # +------------------+---------+--------+----------------------------+----------------------------+------------+
   ```

2. Check etcd pod logs:
   ```bash
   kubectl -n kube-system logs $(kubectl -n kube-system get pods -l component=etcd -o name)
   
   # Example output:
   # 2023-03-03 12:34:56.789012 I | etcdserver/membership: failed to connect to 192.168.100.100:2380 (retrying in 1s)
   # 2023-03-03 12:34:57.789012 W | etcdserver/membership: failed to connect to peer 192.168.100.100:2380 (failed to dial)
   ```

3. Check etcd endpoint health:
   ```bash
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     endpoint health -w table
     
   # Example output:
   # +------------------------+--------+-------------+-------+
   # |        ENDPOINT        | HEALTH |    TOOK     | ERROR |
   # +------------------------+--------+-------------+-------+
   # | https://127.0.0.1:2379 |   true | 9.059797ms  |       |
   # +------------------------+--------+-------------+-------+
   ```

4. Check cluster health status:
   ```bash
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     endpoint status -w table
     
   # Example output:
   # +------------------------+------------------+---------------+----------+----------------+
   # |        ENDPOINT        |        ID        |   VERSION     |  LEADER  |    RAFT TERM   |
   # +------------------------+------------------+---------------+----------+----------------+
   # | https://127.0.0.1:2379 | 8211f1d0f64f3269 |    3.5.6      |  8211f1d |           143 |
   # +------------------------+------------------+---------------+----------+----------------+
   ```

5. Examine the etcd configuration:
   ```bash
   kubectl -n kube-system get pod $(kubectl -n kube-system get pods -l component=etcd -o name | cut -d'/' -f2) -o yaml
   
   # Look for initial-cluster configuration and initial-cluster-state
   ```

### Remediation
1. Remove the failed etcd member:
   ```bash
   # Get the ID of the failed member
   FAILED_MEMBER_ID=$(ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     member list | grep etcd-failed | cut -d',' -f1)
   
   echo "Removing failed member with ID: $FAILED_MEMBER_ID"
   
   # Remove the failed member
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     member remove $FAILED_MEMBER_ID
     
   # You should see output like:
   # Member f4ea13541bc89ce2 removed from cluster abc123def
   ```

2. Verify the etcd cluster is healthy after removal:
   ```bash
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     endpoint health -w table
     
   # All remaining endpoints should show healthy
   
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/
    etcd/server.crt \
        --key=/etc/kubernetes/pki/etcd/server.key \
        member list -w table
        
   # The failed member should no longer appear in the list
   ```

3. Add a new etcd member (in a real cluster, this would be on a new node):
   ```bash
   # In a real scenario, you would:
   # 1. Set up a new node with the etcd binary
   # 2. Copy the etcd certificates to the new node
   # 3. Add the new member to the cluster
   
   # For demonstration purposes:
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     member add etcd-new --peer-urls=https://192.168.1.10:2380
     
   # Expected output:
   # Member 2be1eb8f84fb7f63 added to cluster abc123def
   #
   # ETCD_NAME="etcd-new"
   # ETCD_INITIAL_CLUSTER="master=https://172.16.8.2:2380,k8s-worker1=https://172.16.8.3:2380,etcd-new=https://192.168.1.10:2380"
   # ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.1.10:2380"
   # ETCD_INITIAL_CLUSTER_STATE="existing"
   ```

4. On a new node, configure etcd with the information provided:
   ```bash
   # On the new etcd node:
   # Create necessary directories
   sudo mkdir -p /etc/kubernetes/pki/etcd
   sudo mkdir -p /var/lib/etcd
   
   # Copy certificates from an existing etcd node
   # scp -r root@<existing-etcd-node>:/etc/kubernetes/pki/etcd/* /etc/kubernetes/pki/etcd/
   
   # Create etcd.service file with correct configuration
   cat <<EOF | sudo tee /etc/systemd/system/etcd.service
   [Unit]
   Description=etcd
   Documentation=https://github.com/coreos/etcd
   
   [Service]
   Type=notify
   ExecStart=/usr/local/bin/etcd \\
     --name="etcd-new" \\
     --initial-cluster="master=https://172.16.8.2:2380,k8s-worker1=https://172.16.8.3:2380,etcd-new=https://192.168.1.10:2380" \\
     --initial-cluster-state="existing" \\
     --initial-advertise-peer-urls="https://192.168.1.10:2380" \\
     --advertise-client-urls="https://192.168.1.10:2379" \\
     --listen-peer-urls="https://192.168.1.10:2380" \\
     --listen-client-urls="https://192.168.1.10:2379,https://127.0.0.1:2379" \\
     --client-cert-auth=true \\
     --peer-client-cert-auth=true \\
     --cert-file=/etc/kubernetes/pki/etcd/server.crt \\
     --key-file=/etc/kubernetes/pki/etcd/server.key \\
     --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt \\
     --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt \\
     --peer-key-file=/etc/kubernetes/pki/etcd/peer.key \\
     --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt \\
     --data-dir=/var/lib/etcd
   Restart=on-failure
   RestartSec=5
   
   [Install]
   WantedBy=multi-user.target
   EOF
   
   # Start and enable etcd
   sudo systemctl daemon-reload
   sudo systemctl enable etcd --now
   ```

5. Verify the new member has joined:
   ```bash
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     member list -w table
     
   # The new member should appear in the list with status "started"
   ```

### Best Practices for HA ETCD Clusters
1. Always back up etcd before making changes:
   ```bash
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     snapshot save /tmp/etcd-backup-$(date +%Y-%m-%d-%H-%M-%S).db
   ```

2. Always verify the backup is valid:
   ```bash
   ETCDCTL_API=3 etcdctl --write-out=table snapshot status \
     /tmp/etcd-backup-$(date +%Y-%m-%d-%H-%M-%S).db
     
   # Example output:
   # +----------+----------+------------+------------+
   # |   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
   # +----------+----------+------------+------------+
   # | fe01cf57 |    10582 |       1329 | 2.1 MB     |
   # +----------+----------+------------+------------+
   ```

3. Use an odd number of etcd members (3, 5, or 7) to maintain quorum
   - With 3 members, you can tolerate 1 member failure
   - With 5 members, you can tolerate 2 member failures
   - With 7 members, you can tolerate 3 member failures

4. Understanding quorum in etcd:
   - Quorum = (n/2)+1 where n is the cluster size
   - For 3-node cluster: quorum = 2 nodes
   - For 5-node cluster: quorum = 3 nodes
   - Always maintain quorum or your cluster will become read-only

5. Run etcd on dedicated nodes for production
   - Separate etcd from other workloads for stability
   - Use nodes with SSD storage for better performance

6. Monitor etcd metrics for health and performance:
   ```bash
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     --write-out=json endpoint status | jq
     
   # Example output shows detailed member stats including:
   # - Leader ID
   # - Raft term
   # - Raft index
   # - DB size
   # - Current leader
   ```

7. Handling split-brain scenarios:
   - If etcd cluster loses quorum, you might need to restore from backup
   - In emergency, you can force a new cluster with one member:
   ```bash
   # This is ONLY for disaster recovery when quorum is permanently lost
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     snapshot restore /tmp/etcd-backup.db \
     --data-dir=/var/lib/etcd-recovery \
     --name=master \
     --initial-cluster=master=https://172.16.8.2:2380 \
     --initial-cluster-token=etcd-cluster-1 \
     --initial-advertise-peer-urls=https://172.16.8.2:2380 \
     --force-new-cluster
   ```

8. When adding new members:
   - Always add members one at a time
   - Wait for the new member to fully sync before adding the next one
   - Check cluster health after each addition

### Do Not Forget (Key Takeaways)
- ✅ HA etcd clusters need an odd number of members (3, 5, 7) for quorum
- ✅ Quorum = (n/2)+1 where n is cluster size (3 members = 2 quorum)
- ✅ To remove a failed member, use `etcdctl member remove <ID>`
- ✅ To add a new member, use `etcdctl member add <name> --peer-urls=<url>`
- ✅ For etcd cluster operations, see [Operating etcd clusters](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

## Scenario 13: Certificate Rotation and TLS Troubleshooting

### Key Learning Objectives
- Understand Kubernetes certificate management
- Troubleshoot TLS certificate issues
- Learn certificate rotation procedures
- Diagnose kubelet client certificate problems
- Verify API server connection with proper certificates

### Problem Statement
The cluster's kubelets have failing TLS connections to the API server. Upon investigation, you discover that the kubelet client certificates have expired. Your task is to troubleshoot the TLS errors and rotate the kubelet client certificates to restore proper node-to-control-plane communication.

> **Relevant Official Documentation**:
> - [Certificate Management with kubeadm](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)
> - [PKI certificates and requirements](https://kubernetes.io/docs/setup/best-practices/certificates/)

### Fault Injection
Set up a scenario with expired certificates:
```bash
# First, backup the current certificate for reference
ssh k8s-worker1 "sudo cp /var/lib/kubelet/pki/kubelet-client-current.pem /tmp/kubelet-client-backup.pem"

# Create a short-lived certificate on k8s-worker1
ssh k8s-worker1 "sudo kubeadm cert renew kubelet-client --config /tmp/kubeadm-config.yaml"

# Artificially expire the certificate by modifying the timestamp
ssh k8s-worker1 "sudo sed -i 's/NotAfter:/NotAfter: 210101000000Z # /' /var/lib/kubelet/pki/kubelet-client-current.pem"

# Restart kubelet to pick up the "expired" certificate
ssh k8s-worker1 "sudo systemctl restart kubelet"
```

### Diagnostic Steps
1. Check the node status and kubelet healthiness:
   ```bash
   kubectl get nodes
   # k8s-worker1 will show as NotReady

   kubectl describe node k8s-worker1
   # Look for conditions and errors related to kubelet communication
   ```

2. Examine kubelet logs on the problematic node:
   ```bash
   ssh k8s-worker1 "sudo journalctl -u kubelet | tail -n 50"
   # Look for TLS certificate errors
   ```

3. Check certificate expiration:
   ```bash
   ssh k8s-worker1 "sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout | grep -A2 Validity"
   # Note the certificate validity period
   ```

4. Check API server connection from worker node:
   ```bash
   ssh k8s-worker1 "curl -v https://k8s-master:6443/healthz --cacert /etc/kubernetes/pki/ca.crt --cert /var/lib/kubelet/pki/kubelet-client-current.pem --key /var/lib/kubelet/pki/kubelet-client-current.pem"
   # Should show TLS/certificate errors
   ```

### Remediation
1. Rotate the kubelet client certificate:
   ```bash
   # In a kubeadm-managed cluster, we can renew cert with kubeadm
   ssh k8s-worker1 "sudo kubeadm cert renew kubelet-client"
   
   # Alternative manual approach (if needed):
   ssh k8s-worker1 "sudo cp /tmp/kubelet-client-backup.pem /var/lib/kubelet/pki/kubelet-client-current.pem"
   ```

2. Restart kubelet to use the new certificate:
   ```bash
   ssh k8s-worker1 "sudo systemctl restart kubelet"
   ```

3. Verify the node has rejoined the cluster:
   ```bash
   kubectl get nodes
   # k8s-worker1 should show as Ready
   
   # Verify certificate validity
   ssh k8s-worker1 "sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout | grep -A2 Validity"
   ```

4. Set up automatic certificate rotation for future renewals:
   ```bash
   # In Kubernetes 1.32, certificate rotation is enabled by default with advanced features
   # Check that the automatic rotation is functioning
   ssh k8s-worker1 "sudo systemctl status kubelet | grep certificate-rotation"
   
   # If needed, verify the rotation settings in the kubelet configuration
   ssh k8s-worker1 "sudo grep -A5 rotation /var/lib/kubelet/config.yaml"
   
   # Certificates should appear in the auto rotation list
   ssh k8s-worker1 "sudo kubeadm certs check-expiration | grep kubelet-client"
   ```

### Do Not Forget (Key Takeaways)
- ✅ Check certificate expiration with `kubeadm certs check-expiration`
- ✅ Renew certificates with `kubeadm certs renew <component>`
- ✅ Verify certificate dates with `openssl x509 -in cert.pem -text -noout | grep Validity`
- ✅ Kubelet requires valid client certificates to communicate with API server
- ✅ For certificate management, see [Certificate Management with kubeadm](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)

## Scenario 14: Admission Controllers and Pod Security Standards

### Key Learning Objectives
- Understand Kubernetes admission controllers
- Configure and troubleshoot Pod Security Standards
- Diagnose admission control errors and policy violations
- Create workloads that comply with security requirements
- Learn namespace-level security configurations

### Problem Statement
Your organization has implemented Pod Security Standards using Kubernetes Admission Controllers. Development teams are reporting that their workloads are being blocked from deployment. Your task is to diagnose the admission controller issues, identify policy violations, and adjust the deployment to comply with security standards.

> **Relevant Official Documentation**:
> - [Using Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
> - [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)

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

### Do Not Forget (Key Takeaways)
- ✅ Pod Security Standards have three levels: Privileged, Baseline, and Restricted
- ✅ Admission controllers validate resources before they're persisted to etcd
- ✅ If pods are rejected, check events with `kubectl get events` for details
- ✅ Use `kubectl describe namespace` to check Pod Security level labels
- ✅ For security compliance, see [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)

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

> **Relevant Official Documentation**:
> - [Troubleshooting Clusters](https://kubernetes.io/docs/tasks/debug/debug-cluster/)
> - [Debugging Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)

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

# Disrupt networking on k8s-worker1
echo "2. Disrupting network on k8s-worker1..."
ssh k8s-worker1 "sudo iptables -I INPUT -p tcp --dport 10250 -j DROP"

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
ssh k8s-worker1 "sudo mkdir -p /mnt/data && sudo chmod 700 /mnt/data"
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
ssh k8s-worker1 "sudo k8s-node-health-check"
ssh k8s-worker2 "sudo k8s-node-health-check"
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
# Test k8s-worker1 connectivity
ssh k8s-worker1 "curl -k https://k8s-master:6443/healthz"
# Check iptables rules
ssh k8s-worker1 "sudo iptables -L -n"
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

#### 2. Fix Network Connectivity on k8s-worker1
```bash
# Remove the blocking iptables rule
ssh k8s-worker1 "sudo iptables -D INPUT -p tcp --dport 10250 -j DROP"
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
ssh k8s-worker1 "sudo chmod 755 /mnt/data"
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

### Solution Walk-Through for Combined Mock Exam

Here is a step-by-step approach to resolving all issues in the combined mock exam:

1. **API Server Configuration Issue**
   - Diagnosis: API server fails to start due to incorrect etcd server entries in its manifest
   - Fix: `sudo cp /tmp/kube-apiserver.yaml.bak /etc/kubernetes/manifests/kube-apiserver.yaml`
   - Validation: `kubectl get nodes` starts working after API server recovers

2. **Network Connectivity on k8s-worker1**
   - Diagnosis: Blocking iptables rule on port 10250 prevents kubelet communication
   - Fix: `ssh k8s-worker1 "sudo iptables -D INPUT -p tcp --dport 10250 -j DROP"`
   - Validation: Worker node becomes Ready in `kubectl get nodes`

3. **Scheduler RBAC Issues**
   - Diagnosis: Incorrect ClusterRoleBinding for kube-scheduler
   - Fix: 
     ```bash
     kubectl delete clusterrolebinding system:kube-scheduler
     kubectl create clusterrolebinding system:kube-scheduler --clusterrole=system:kube-scheduler --user=system:kube-scheduler
     ```
   - Validation: New pods start getting scheduled normally

4. **Resource Quota Constraints**
   - Diagnosis: Pod can't start due to restrictive ResourceQuota in namespace
   - Fix: 
     ```bash
     kubectl edit resourcequota compute-quota -n restricted-space
     # Increase limits or
     kubectl delete resourcequota compute-quota -n restricted-space
     ```
   - Validation: `kubectl get pods -n restricted-space` shows Running status

5. **PV/PVC Issues**
   - Diagnosis: Access mode mismatch between PV (ReadWriteMany) and PVC (ReadWriteOnce)
   - Fix:
     ```bash
     kubectl delete pv broken-pv
     # Create new PV with correct access mode
     kubectl apply -f fixed-pv.yaml
     ```
   - Validation: `kubectl get pvc -n data-services` shows Bound status

6. **Overly Restrictive NetworkPolicy**
   - Diagnosis: NetworkPolicy blocking traffic in kube-system namespace
   - Fix: `kubectl delete networkpolicy -n kube-system default-deny-all`
   - Validation: Components in kube-system can communicate again

7. **Verification of All Fixes**
   - Run: `kubectl get nodes,pods --all-namespaces`
   - Check: All nodes Ready, critical pods Running
   - Validate: Test applications with `kubectl run test-pod --image=busybox -- sleep 3600` and verify connectivity

For detailed commands and specific error messages encountered during the fixes, refer to Scenario 15.

### Key Exam Tips
During the actual CKA exam, when facing multiple issues:
1. Fix control plane issues first (API server, etcd, scheduler)
2. Restore cluster communication pathways (networking, node connectivity)
3. Resolve RBAC and security issues
4. Address application-specific problems (resource conflicts, storage issues)
5. Always verify fixes before moving to the next issue

### Do Not Forget (Key Takeaways)
- ✅ Always prioritize control plane issues before application problems
- ✅ Use a methodical approach: check node status → check pods → check services
- ✅ For complex issues, break them down into smaller, manageable problems
- ✅ Verify each fix before moving to the next issue
- ✅ Time management is critical, focus on high-point questions first in the real exam
- ✅ For real-world complex issues, see [Troubleshooting Clusters](https://kubernetes.io/docs/tasks/debug/debug-cluster/)

## Scenario 16: Dynamic Storage Provisioning with CSI Drivers

### Key Learning Objectives
- Configure and troubleshoot dynamic storage provisioning
- Understand Container Storage Interface (CSI) drivers
- Create and manage StorageClasses for automated PV provisioning
- Diagnose common storage provisioning failures
- Configure volume expansion and storage class parameters

### Problem Statement
Your team needs to implement dynamic storage provisioning for applications. The current static provisioning approach is causing delays in deployment and scaling. You need to configure a CSI driver, set up appropriate StorageClasses, and ensure applications can automatically get the storage they need.

> **Note**: For static provisioning with manually created PVs, see [Scenario 8: Persistent Volume and Storage Troubleshooting](#scenario-8-persistent-volume-and-storage-troubleshooting).
>
> In Kubernetes 1.32, the CSI framework has been enhanced with automatic migration tools, volume health monitoring, and snapshot scheduling capabilities.

> **Relevant Official Documentation**:
> - [Dynamic Volume Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)
> - [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)

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
cat <<EOF > /tmp/broken-storage-class.yaml
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

kubectl apply -f /tmp/broken-storage-class.yaml

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
   ssh k8s-worker1 sudo getenforce
   # If it returns "Enforcing", it might be blocking volume access
   
   # Temporarily set to permissive for testing
   ssh k8s-worker1 sudo setenforce 0
   
   # For a permanent fix, create the correct SELinux context
   ssh k8s-worker1 sudo chcon -Rt svirt_sandbox_file_t /mnt/data
   
   # For RHEL/AlmaLinux environments, check contexts more thoroughly
   ssh k8s-worker1 "sudo ls -lZ /mnt/data"
   
   # Apply the container file context policy
   ssh k8s-worker1 "sudo semanage fcontext -a -t container_file_t '/mnt/data(/.*)?'"
   ssh k8s-worker1 "sudo restorecon -Rv /mnt/data"
   ```

2. **Directory permissions**:
   ```bash
   # Check directory permissions
   ssh k8s-worker1 ls -la /mnt/data
   
   # Fix permissions if needed - typical container UIDs need read/write
   ssh k8s-worker1 sudo chmod 755 /mnt/data
   ```

3. **AppArmor/SecComp profiles** (if enabled):
   ```bash
   # Check if profiles are loaded
   ssh k8s-worker1 sudo aa-status
   
   # Check Docker/containerd configuration for default profiles
   ssh k8s-worker1 cat /etc/docker/daemon.json
   ```

4. **Disk space issues**:
   ```bash
   # Check available disk space
   ssh k8s-worker1 df -h
   
   # Check inode usage (sometimes overlooked)
   ssh k8s-worker1 df -i
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

### Generalizing CSI Driver Patterns

The concepts and troubleshooting techniques in this scenario apply to all CSI drivers in Kubernetes 1.32, which introduces streamlined driver management and enhanced observability. When adapting to other CSI implementations:

1. **Provider-Specific Parameters**: Each CSI driver has unique parameters in its StorageClass definition.
   ```yaml
   # AWS EBS example with 1.32 optimizations:
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: aws-ebs-csi
   provisioner: ebs.csi.aws.com
   parameters:
     type: gp3
     encrypted: "true"
     iops: "6000"
     throughput: "250"
   reclaimPolicy: Delete
   allowVolumeExpansion: true
   volumeBindingMode: WaitForFirstConsumer
   mountOptions:
     - discard
   healthMonitor:
     enabled: true
     interval: 60s
   ```

2. **Simplified Driver Installation**: Kubernetes 1.32 includes the CSI driver operator for streamlined deployment.
   ```bash
   # Example of using the built-in CSI operator in 1.32
   kubectl csi install aws-ebs \
     --namespace=kube-system \
     --set credentials=secret:aws-creds \
     --set autoResize=true \
     --set snapshotClass=true
   ```

3. **Enhanced Volume Health Monitoring**: Kubernetes 1.32 provides automated volume health checks.
   ```yaml
   # Example VolumeHealthMonitor configuration
   apiVersion: storage.k8s.io/v1
   kind: VolumeHealthMonitor
   metadata:
     name: default-monitor
   spec:
     storageClasses:
     - "*"
     probeInterval: 5m
     actions:
       onUnmounted: Remount
       onIOError: Rebuild
   ```

4. **Topology Awareness with Advanced Scheduling**: Kubernetes 1.32 includes improved zone balancing.
   ```yaml
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: topology-aware-storage
   provisioner: ebs.csi.aws.com
   parameters:
     type: gp3
   volumeBindingMode: WaitForFirstConsumer
   allowedTopologies:
   - matchLabelExpressions:
     - key: topology.kubernetes.io/zone
       values:
       - us-east-1a
       - us-east-1b
   topologyBalancing: true # New in 1.32
   ```

When you encounter a new CSI driver in your Kubernetes 1.32 environment:
1. Use the built-in CSI driver operator for simplified installation
2. Enable the volume health monitoring for automated recovery
3. Configure snapshot schedules for data protection
4. Utilize the enhanced topology features for better zone distribution

### Cross-References with Other Storage Scenarios

If you need to troubleshoot more basic storage issues related to static provisioning, refer to:
- **Scenario 8: Persistent Volume and Storage Troubleshooting** - Covers static PV/PVC binding, access mode mismatches, and volume permissions

When working with stateful applications that need storage:
- **Scenario 4: ETCD Backup and Restore** - Provides insights into managing storage for critical cluster components
- **Scenario 12: High-Availability ETCD Cluster** - Shows advanced multi-node storage configurations

### Additional StorageClass Configurations
For a more comprehensive understanding, here are examples of other common StorageClass configurations:

```yaml
# AWS EBS Storage Class example
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-ebs-sc
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
  fsType: ext4
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

```yaml
# GCE PD Storage Class example
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gce-pd-sc
provisioner: pd.csi.storage.gke.io
parameters:
  type: pd-ssd
  replication-type: none
  disk-encryption-kms-key: projects/my-project/locations/us-central1/keyRings/my-kr/cryptoKeys/my-key
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

```yaml
# Azure Disk Storage Class example
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azure-disk
provisioner: disk.csi.azure.com
parameters:
  skuName: Premium_LRS
  kind: Managed
  cachingMode: ReadOnly
allowVolumeExpansion: true
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

```yaml
# NFS Storage Class example (using NFS subdir external provisioner)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
parameters:
  server: nfs-server.example.com
  path: /exported/path
  mountOptions: "nfsvers=4.1,hard"
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

### Summary of CSI Drivers
Here's a reference for common CSI drivers you might encounter:

| Cloud Provider | CSI Driver | StorageClass Provisioner | Common Parameters |
|----------------|------------|--------------------------|-------------------|
| AWS | aws-ebs-csi-driver | ebs.csi.aws.com | type (gp2, gp3, io1), iopsPerGB, encrypted |
| GCP | gcp-compute-persistent-disk-csi-driver | pd.csi.storage.gke.io | type (pd-standard, pd-ssd), replication-type |
| Azure | azuredisk-csi-driver | disk.csi.azure.com | skuName (Standard_LRS, Premium_LRS), cachingMode |
| Local | local-path-provisioner | rancher.io/local-path | (varies based on implementation) |
| NFS | nfs-subdir-external-provisioner | nfs.csi.k8s.io | server, share, mountOptions |

### Do Not Forget (Key Takeaways)
- ✅ For dynamic provisioning, ensure the StorageClass references a valid provisioner
- ✅ Check provisioner pod logs for errors (`kubectl logs -n <namespace> <provisioner-pod>`)
- ✅ For volume expansion, StorageClass must have `allowVolumeExpansion: true`
- ✅ If using cloud providers, verify proper authentication for the CSI driver
- ✅ If provisioning fails, see [Troubleshooting PersistentVolumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#troubleshooting)
- ✅ For advanced storage topics, see [Scenario 8: Persistent Volume Troubleshooting](#scenario-8-persistent-volume-and-storage-troubleshooting)

## Scenario 17: Advanced Debugging with Ephemeral Containers

### Key Learning Objectives
- Master advanced debugging techniques using ephemeral containers
- Troubleshoot running pods without modifying their original definition
- Diagnose complex networking and application issues
- Understand when and how to use different debugging tools
- Learn how to debug containers with minimal base images

### Problem Statement
Your team has deployed several microservices with minimal container images (distroless or scratch-based) that lack debugging tools. Users are reporting intermittent connectivity issues and performance problems. You need to diagnose these issues without disrupting the running applications or modifying their deployment definitions.

```
┌─────────────────────────────────────────────────────────────┐
│ Pod                                                         │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────────────────┐    │
│  │                 │    │  Ephemeral Debug Container  │    │
│  │  Application    │◄───│                             │    │
│  │  Container      │    │  - Network tools            │    │
│  │                 │    │  - Process monitoring       │    │
│  │ (minimal image) │    │  - Filesystem inspection    │    │
│  └─────────────────┘    └─────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

> **Relevant Official Documentation**:
> - [Debugging with Ephemeral Containers](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/#ephemeral-container)
> - [Using kubectl debug](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#debug)

### Fault Injection
Set up an environment with problematic pods:

```bash
# Create a namespace for our debugging scenarios
kubectl create namespace debug-practice

# Deploy an application using a minimal distroless image
cat <<EOF > /tmp/minimal-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minimal-app
  namespace: debug-practice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: minimal-app
  template:
    metadata:
      labels:
        app: minimal-app
    spec:
      containers:
      - name: app
        image: gcr.io/distroless/static-debian11
        command: ["/busybox/sleep", "infinity"]
        resources:
          limits:
            memory: "50Mi"
            cpu: "100m"
          requests:
            memory: "25Mi"
            cpu: "50m"
---
apiVersion: v1
kind: Service
metadata:
  name: minimal-service
  namespace: debug-practice
spec:
  selector:
    app: minimal-app
  ports:
  - port: 80
    targetPort: 8080
EOF

kubectl apply -f /tmp/minimal-app.yaml

# Deploy a second app that will communicate with the first
cat <<EOF > /tmp/client-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-app
  namespace: debug-practice
spec:
  replicas: 1
  selector:
    matchLabels:
      app: client
  template:
    metadata:
      labels:
        app: client
    spec:
      containers:
      - name: client
        image: busybox:1.28
        command: ["/bin/sh", "-c", "while true; do wget -q -O- minimal-service || echo 'Failed'; sleep 5; done"]
EOF

kubectl apply -f /tmp/client-app.yaml

# Create a NetworkPolicy that's too restrictive
cat <<EOF > /tmp/restrictive-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-traffic
  namespace: debug-practice
spec:
  podSelector:
    matchLabels:
      app: minimal-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: monitoring
EOF

kubectl apply -f /tmp/restrictive-policy.yaml

# Inject high CPU usage in one pod
POD=$(kubectl get pod -n debug-practice -l app=minimal-app -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n debug-practice $POD -- /busybox/sh -c "while true; do :; done" &
```

### Diagnostic Steps

#### 1. Check the application status

```bash
# Check client pod logs to confirm connectivity issues
CLIENT_POD=$(kubectl get pod -n debug-practice -l app=client -o jsonpath='{.items[0].metadata.name}')
kubectl logs -n debug-practice $CLIENT_POD

# Example output:
# wget: can't connect to remote host minimal-service: Connection refused
# Failed
# wget: can't connect to remote host minimal-service: Connection refused
# Failed
```

#### 2. Initial investigation of application pods

```bash
# Check pod status
kubectl get pods -n debug-practice -o wide

# Try to check logs of the minimal app
APP_POD=$(kubectl get pod -n debug-practice -l app=minimal-app -o jsonpath='{.items[0].metadata.name}')
kubectl logs -n debug-practice $APP_POD

# Try to exec into the container (this will fail with distroless)
kubectl exec -it -n debug-practice $APP_POD -- sh
# Example output:
# OCI runtime exec failed: exec failed: container_linux.go:380: starting container process caused: exec: "sh": executable file not found in $PATH: unknown
```

#### 3. Using ephemeral containers for network debugging

```bash
# Add a debug container to investigate network configuration
kubectl debug -n debug-practice $APP_POD -it --image=nicolaka/netshoot --target=app

# Inside the debug container:
# Check if the application is listening on port 8080
netstat -tulpn | grep 8080
# Check basic connectivity
ping minimal-service
# Analyze NetworkPolicy impact
iptables-save | grep DROP
```

#### 4. Using ephemeral containers for process debugging

```bash
# Add a debug container to check process information
kubectl debug -n debug-practice $APP_POD -it --image=busybox --target=app

# Inside the debug container:
# Check running processes
ps aux
# Check CPU and memory usage
top
# Check for file descriptors
ls -la /proc/1/fd/
```

#### 5. Using ephemeral containers for filesystem inspection

```bash
# Add a debug container to inspect the filesystem
kubectl debug -n debug-practice $APP_POD -it --image=ubuntu --target=app

# Inside the debug container:
# Check filesystem usage
df -h
# Inspect the container filesystem
find / -type f -name "*.conf" 2>/dev/null
# Check mounted volumes
mount | grep volume
```

#### 6. Analyzing network policies

```bash
# Check network policies in the namespace
kubectl get networkpolicies -n debug-practice
kubectl describe networkpolicy restrict-traffic -n debug-practice

# Test connectivity from a pod with correct labels
kubectl run test-pod -n debug-practice --labels="role=monitoring" --image=busybox -- sleep 3600
kubectl exec -it test-pod -n debug-practice -- wget -q -O- --timeout=2 minimal-service

# Test connectivity from a pod without correct labels
kubectl run another-test -n debug-practice --image=busybox -- sleep 3600
kubectl exec -it another-test -n debug-practice -- wget -q -O- --timeout=2 minimal-service
# This should fail due to the NetworkPolicy
```

### Remediation

1. Fix the NetworkPolicy to allow proper communication:

```bash
cat <<EOF > /tmp/fixed-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-traffic
  namespace: debug-practice
spec:
  podSelector:
    matchLabels:
      app: minimal-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels: {}
EOF

kubectl apply -f /tmp/fixed-policy.yaml
```

2. Identify and terminate the CPU-intensive process:

```bash
# Add a debug container to identify and stop the process
kubectl debug -n debug-practice $APP_POD -it --image=busybox --target=app

# Inside the debug container:
# Find the busy loop process
ps aux | grep sh
# Terminate the process
kill -9 <PID>
```

3. Update the Service to point to the correct port:

```bash
kubectl edit service minimal-service -n debug-practice
# Change targetPort from 8080 to 80
```

4. Verify fixes:

```bash
# Check client pod logs to confirm connectivity is restored
kubectl logs -n debug-practice $CLIENT_POD -f

# Check CPU usage
kubectl top pod -n debug-practice
```

### Advanced Debugging Techniques

#### 1. Copying a Pod with Debug Options

```bash
# Create a copy of the pod with debugging tools
kubectl debug -n debug-practice $APP_POD --image=ubuntu --share-processes --copy-to=debug-$APP_POD

# This creates a new pod with the same configuration plus debugging tools
```

#### 2. Node-Level Debugging

```bash
# Debug at the node level
NODE=$(kubectl get pod -n debug-practice $APP_POD -o jsonpath='{.spec.nodeName}')
kubectl debug node/$NODE -it --image=nicolaka/netshoot

# Inside the debug container:
chroot /host
# Now you can access the node's filesystem and processes
```

#### 3. Container Runtime Debugging

```bash
# Get container ID
CONTAINER_ID=$(kubectl get pod -n debug-practice $APP_POD -o jsonpath='{.status.containerStatuses[0].containerID}' | sed 's/containerd:\/\///')

# SSH to the node and inspect with crictl
ssh $NODE
sudo crictl inspect $CONTAINER_ID
sudo crictl stats $CONTAINER_ID
```

### Quick Reference for Ephemeral Container Debugging

| Issue | Debug Container | Key Commands |
|-------|----------------|-------------|
| Network | `nicolaka/netshoot` | `netstat -tulpn`, `tcpdump`, `ping`, `curl`, `iptables-save` |
| Process | `busybox` or `ubuntu` | `ps aux`, `top`, `kill`, `strace` |
| Filesystem | `ubuntu` | `ls -la`, `find`, `cat`, `df -h` |
| Memory | `nicolaka/netshoot` | `free -m`, `cat /proc/meminfo`, `smem` |
| Performance | `nicolaka/netshoot` | `top`, `mpstat`, `iostat`, `vmstat` |

For more details, see: [Kubernetes Debugging Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/) and [Ephemeral Containers docs](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)

### Do Not Forget (Key Takeaways)
- ✅ Use `kubectl debug` to add ephemeral containers to running pods
- ✅ For network debugging, use `nicolaka/netshoot` image 
- ✅ For process debugging, use `busybox` or `ubuntu` image
- ✅ Specify target container with `--target=container-name`
- ✅ For pods based on minimal images, see [Debug Running Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)
- ✅ For node-level debugging, try `kubectl debug node/node-name -it --image=ubuntu`

## Appendix: Emergency Commands Reference

This quick reference provides essential commands for common emergency scenarios in the CKA exam:

### Control Plane Recovery

```bash
# When API server is down, check kubelet
systemctl status kubelet
journalctl -u kubelet | tail -50

# Check static pod manifests
ls -la /etc/kubernetes/manifests/

# Check running containers when kubectl fails
crictl ps
crictl logs <container-id>

# Manually restart kubelet
systemctl restart kubelet
```

### Certificate Management

```bash
# Check certificate expiration
kubeadm certs check-expiration

# Renew all certificates
kubeadm certs renew all

# Renew specific certificate
kubeadm certs renew apiserver

# Check certificate details
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep Validity -A2
```

### ETCD Operations

```bash
# Backup etcd (API server working)
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /tmp/etcd-backup.db

# Check backup status
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /tmp/etcd-backup.db

# Restore etcd
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db --data-dir=/var/lib/etcd-restore
```

### Debugging Commands

```bash
# Debug a pod with ephemeral container
kubectl debug <pod-name> -it --image=busybox --target=<container-name>

# Debug a node directly
kubectl debug node/<node-name> -it --image=ubuntu

# Check events (sorted by time)
kubectl get events --sort-by='.lastTimestamp'

# Check pod logs with timestamps
kubectl logs <pod-name> --timestamps
```

### NetworkPolicy Testing

```bash
# Create test pod for connectivity checks
kubectl run test-pod --image=busybox --rm -it -- wget -qO- <service-name>

# Check if NetworkPolicy is blocking traffic
kubectl describe networkpolicy <policy-name> -n <namespace>

# Check effective iptables rules (on node)
iptables-save | grep DROP | grep <pod-ip>
```

### Storage Debugging

```bash
# Check PV/PVC status and details
kubectl get pv,pvc
kubectl describe pv <pv-name>
kubectl describe pvc <pvc-name> -n <namespace>

# Check StorageClass details
kubectl get sc
kubectl describe sc <storage-class-name>

# Check SELinux context for volume paths (on node)
ls -Z /path/to/volume
chcon -Rt svirt_sandbox_file_t /path/to/volume

# For RHEL/AlmaLinux/CentOS systems with SELinux
# Apply the container file context permanently 
semanage fcontext -a -t container_file_t '/path/to/volume(/.*)?'
restorecon -Rv /path/to/volume

# Check storage provisioner logs
kubectl logs -n <namespace> <provisioner-pod-name>
```

### Node Management

```bash
# Drain a node safely
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Mark node as unschedulable (without eviction)
kubectl cordon <node-name>

# Mark node as schedulable again
kubectl uncordon <node-name>

# Check resource usage
kubectl top nodes
kubectl top pods -A
```

### RBAC Verification

```bash
# Check permissions for your current user
kubectl auth can-i <verb> <resource>

# Check permissions for another user
kubectl auth can-i <verb> <resource> --as=<username>

# Check permissions in specific namespace
kubectl auth can-i <verb> <resource> --namespace=<namespace>
```

### Last Resort Recovery

```bash
# If a pod is totally stuck, force delete
kubectl delete pod <pod-name> --grace-period=0 --force

# If API server is broken beyond repair
sudo rm /etc/kubernetes/manifests/kube-apiserver.yaml
sudo cp /path/to/backup/kube-apiserver.yaml /etc/kubernetes/manifests/
```

For more detailed instructions, refer to the specific scenario that matches your issue.

## Common Diagnostic Steps

To avoid repetition, refer to these standard diagnostic procedures throughout the scenarios:

#### Pod Issues

```bash
# 1. Check pod status
kubectl get pods [-n namespace]

# 2. Check pod details - events section is critical
kubectl describe pod <pod-name> [-n namespace]

# 3. Check container logs
kubectl logs <pod-name> [-c container-name] [-n namespace]
# Check previous container (if crashed)
kubectl logs <pod-name> [-c container-name] --previous [-n namespace]
```

#### Network Issues

```bash
# 1. Verify Service exists and has endpoints
kubectl get endpoints <service-name> [-n namespace]

# 2. Check if pods match Service selector
kubectl get pods -l <key>=<value> [-n namespace]

# 3. Test connectivity with temporary pod
kubectl run test --image=busybox [-n namespace] -- sleep 3600
kubectl exec -it test [-n namespace] -- wget -qO- <service-name>

# 4. Check NetworkPolicies
kubectl get networkpolicies [-n namespace]
```

#### Node Issues

```bash
# 1. Check node status
kubectl get nodes
kubectl describe node <node-name>

# 2. Verify kubelet is running on the node
ssh <node-name> "systemctl status kubelet"

# 3. Check kubelet logs
ssh <node-name> "journalctl -u kubelet | tail -50"
```

For more details on specific issues, refer to the relevant scenario.

## CKA Exam Quick Start

1. Set up useful aliases:
   ```bash
   alias k=kubectl
   export do="--dry-run=client -o yaml"
   export now="--force --grace-period 0"
   ```

2. Enable tab completion:
   ```bash
   source <(kubectl completion bash)
   complete -F __start_kubectl k
   ```

3. Bookmark key documentation pages:
   - https://kubernetes.io/docs/reference/kubectl/cheatsheet/
   - https://kubernetes.io/docs/tasks/debug/
   - https://kubernetes.io/docs/tasks/administer-cluster/

