This is a very aggressive timeline, but it is **achievable** if you can dedicate 6-8 hours daily (especially on weekends) and focus exclusively on **hands-on practice**. Since you are a beginner, you must skip all theory fluff and learn by doing.

Here is your **5-Week CKA Crash Course Plan** (with Heavy Hands-On & Error-Centric Learning).

### Pre-Requisites (Day 0 - Do this before starting)[](https://terminal-hrpsernhcboizpeq.labs.kodekloud.com/#p)

1. **Environment Setup (Most Important)**
    - Install **kubectl**.
    - Create a free account on **Killercoda** (browser-based Kubernetes playground) - Use this for quick tests.
    - Sign up for **Killercoda CKA Scenarios** (Free).
    - **Paid but critical:** Buy access to **Killer.sh CKA Simulator** (from the official CKA exam provider). Don't buy the exam yet, just the simulator.
2. **Mindset:** Do not take notes. **Type everything.** Break your cluster often (delete kube-proxy, delete etcd, etc.) and fix it.

---

### Week 1: The Foundation (Cluster Lifecycle & Basics)[](https://terminal-hrpsernhcboizpeq.labs.kodekloud.com/#week-1-the-f)

**Goal:** Break your cluster, fix it, and understand the bones.

|Day|Topic|Action (Heavy Hands-On)|
|---|---|---|
|**1**|**K8s Architecture**|**Don't read docs.** Do this: `kubectl get pods -n kube-system`. Find `etcd`, `kube-apiserver`, `kube-controller-manager`. **Delete them.** Watch the cluster die. Then watch it recover (DaemonSet/Static Pods).|
|**2**|**kubeadm Init**|Spin up 2 VMs (Vagrant or DigitalOcean $5 droplets). Use `kubeadm init` to create a cluster. **Make a mistake:** Forget to install a CNI plugin. Cluster is "NotReady". Debug it, install Calico/Flannel.|
|**3**|**Pods & Namespaces**|Create a pod manually (`kubectl run nginx --image=nginx`). Exec into it (`kubectl exec -it`). **Make it fail:** Give it a wrong image name. Check `kubectl describe pod` and `kubectl logs`.|
|**4**|**Deployments & Replicas**|Create a Deployment. Scale it up/down (`kubectl scale`). Perform a Rolling Update. **Destroy it:** `kubectl delete deployment` → Recreate it from YAML (not from CLI).|
|**5**|**Services**|Expose a Deployment (`ClusterIP`, `NodePort`). **Test access:** Try hitting the NodePort from outside. It fails? Fix it. (Check firewall, kube-proxy, endpoints).|
|**6**|**ConfigMaps & Secrets**|Create a ConfigMap. Mount it into a Pod as a volume. Create a Secret. Base64 encode/decode it manually. **Security error:** Permission denied on secret mount? Fix file permissions.|
|**7**|**Storage (Volumes)**|Create a hostPath volume. Write data to it. Delete the Pod. **Check data persists** (it should). Use `emptyDir`. Delete Pod → Data is gone. Understand the difference.|

---

### Week 2: Deep Dive & Cluster Management (Hardest Week)[](https://terminal-hrpsernhcboizpeq.labs.kodekloud.com/#w)

**Goal:** Master scheduling, networking, and security. **Expect to break things frequently.**

|Day|Topic|Action (Heavy Hands-On)|
|---|---|---|
|**8**|**Labels & Selectors**|Assign labels to nodes. Schedule a pod to a specific node using `nodeSelector`. **Fail:** Use a label that doesn't exist. Pod is Pending. Debug with `kubectl describe pod`.|
|**9**|**Taints & Tolerations**|Taint a node: `kubectl taint nodes worker1 key=value:NoSchedule`. Run a pod without toleration → Pending. Add toleration → Schedules. **Critical:** Practice `NoExecute` taint (evicts existing pods).|
|**10**|**Network Policies**|Create a default deny policy. **Your app breaks.** Create an allow policy for Ingress/Egress. Test connectivity using `netshoot` container (`kubectl run --rm -it test --image=nicolaka/netshoot`).|
|**11**|**etcd Backup & Restore**|This is **GUARANTEED** on the exam. SSH into the control plane node. `ETCDCTL_API=3 etcdctl snapshot save`. **Delete your cluster.** Restore from snapshot. `etcdctl snapshot restore` + move data dir. **You MUST do this twice.**|
|**12**|**RBAC (Roles & Bindings)**|Create a ServiceAccount. Create a Role (only `get pods`). Bind them. Use `auth can-i` to verify. **Access Denied:** Try to `delete` a pod with this SA. It fails. Understand why.|
|**13**|**Static Pods & Manual Scheduling**|Create a Pod manifest in `/etc/kubernetes/manifests/`. Watch it auto-start. **Bypass the scheduler:** Create a pod with `spec.nodeName` set (no scheduler needed).|
|**14**|**Troubleshooting (Week 2 Review)**|**Scenario:** Cluster is not working. Use: `journalctl -u kubelet`, `kubectl get events --all-namespaces`, `crictl logs`. Fix 3 broken scenarios (Coredns crash, Kubelet cert expired, Node not ready).|

---

### Week 3: Exam Simulation Phase 1 (Trial by Fire)[](https://terminal-hrpsernhcboizpeq.labs.kodekloud.com/#week)

**Action:** Do not touch books. Only do **Killer.sh** simulation. You get 2 free sessions with the registration.

- **Day 15-16:** Sit down for **2 hours** (no pauses). Try to do the Killer.sh CKA scenario. **You will fail 50% of tasks.** That's the point.
- **Day 17-18:** Watch the solution videos for Killer.sh. **Type** every command they type. Do not watch passively. Break your cluster and fix it using their solutions.
- **Day 19-21:** Re-do the entire Killer.sh simulation **from scratch**. Aim to finish 90% in under 2 hours.

**Key Error to Practice Here:** Multi-node issues. Make a misconfiguration in kubelet config file (`/var/lib/kubelet/config.yaml`) and watch the node go `NotReady`. Fix the config, restart kubelet → Ready.

---

### Week 4: Exam Simulation Phase 2 & Weak Points[](https://terminal-hrpsernhcboizpeq.labs.kodekloud.com/#week-4-exam)

- **Day 22-23:** Find your weakest area (usually **etcd backup** or **RBAC** or **NetworkPolicy**).
    - **Weak on NetworkPolicy?** Create a namespace "blue" and "green". Allow pods in "blue" to talk to database only. Block "green". Test with `curl`.
    - **Weak on etcd?** Script it:
        
        `#!/bin/bash # Script to backup etcd with endpoints and cacert ETCDCTL_API=3 etcdctl snapshot save /backup/snapshot.db \   --endpoints=https://127.0.0.1:2379 \   --cacert=/etc/kubernetes/pki/etcd/ca.crt \   --cert=/etc/kubernetes/pki/etcd/server.crt \   --key=/etc/kubernetes/pki/etcd/server.key`
        
- **Day 24-26:** **Repeat Killer.sh a third time.** This time, do it without looking at solutions. If stuck, use `kubectl explain` (not Google).
- **Day 27-28:** Find and solve **5 random CKA practice tests online** (e.g., from "KodeKloud CKA Mock Exams" or "Mumshad Mannambeth" – they are excellent). **Deliberately make 1 syntax error per test** (e.g., `--replices=3` instead of `replicas`) and learn to spot it.

---

### Week 5: Certification Week (Final Polish)[](https://terminal-hrpsernhcboizpeq.labs.kodekloud.com/#wee)

- **Day 29:** **Pre-exam Environment Check.**
    - Test your internet connection.
    - **Critical:** Practice switching between the PSI browser window and your terminal _smoothly_.
    - **Alias Check:** Have you set `alias k=kubectl` and `export ns=--all-namespaces`? Do it.
- **Day 30-31:** **One final full mock exam.** Time yourself: 2 hours exactly.
    - **Fail fast strategy:** If a question is hard (e.g., NetworkPolicy), skip it (bookmark), do the easy questions first (etcd backup, RBAC, kubectl run).
- **Day 32:** **Relax.** Review only your `imperative commands` cheat sheet (e.g., `kubectl create deploy`, `kubectl expose`, `kubectl set image`). **Do not touch the cluster.**
- **Day 33:** **Exam Day.** Book the exam slot.
    - **Strategy:** Read all questions first (5 mins). Do the ones you know instantly. Spend remaining time on hard ones.
    - **Don't panic** if you don't finish a question. Partial credit is given.
    - **Cheat Sheet allowed?** No. But you can use `kubectl explain --recursive` and `kubectl api-resources`. Master these.

---

### Critical "Error-Based Learning" Cheat Sheet[](https://terminal-hrpsernhcboizpeq.labs.kodekloud.com/#cri)

To learn fast, do these bugs on purpose:

|Mistake to Make|How to Create it|How to Spot it|
|---|---|---|
|**Wrong image tag**|`kubectl run test --image=nginx:1.99`|`ImagePullBackOff`|
|**Mismatched selector**|Service selector: `app: myapp`, Pod label: `app: mysql`|No endpoints (`kubectl describe svc`)|
|**Missing CNI**|`kubeadm init` without `--pod-network-cidr`|All pods stuck at `ContainerCreating`|
|**Wrong Secret data**|`kubectl create secret generic --from-literal=password='mypass'` then mount it|`base64` error or empty env var|
|**Taint/NodeSelector conflict**|Taint on node + NodeSelector for different node|Pod stays `Pending`|
|**Kubelet dead**|`systemctl stop kubelet` on worker|Node becomes `NotReady`|

### Final Advice for the Exam[](https://terminal-hrpsernhcboizpeq.labs.kodekloud.com/#fi)

- **Use `kubectl run` (imperative) for 80% of tasks.** It's fast. Only use YAML for complex things (Volumes, NetworkPolicy).
- **Master `kubectl create -f /tmp/file.yaml --dry-run=client -o yaml > final.yaml`** . This generates valid YAML from your commands.
- **The exam is in a controlled PSI environment.** Your terminal has no `vim`? Use `vi` or `nano`. Practice using `nano` (easier to exit).
- **Time management:** You have ~3 mins per question. If a question takes longer, **skip it**.

You can do this. The key is `mkdir /tmp/exploit ; kubectl exec -it pod -- /bin/sh` and constantly breaking things to understand them. Good luck

---
# 🎯 CKA Certification in 5 Weeks – Beginner-Friendly Learning Path

**Target:** CKA Certified by Week 5  
**Course:** Mumshad’s CKA Course (Udemy: _CKA Certified Kubernetes Administrator with Practice Tests_)  
**Approach:** Heavy hands-on, learn from mistakes, daily labs, no theory-only days.

---

## 📅 Week 1 – Kubernetes Core Concepts & Cluster Setup (Days 1–7)

|Day|Topics (Mumshad Course Sections)|Hands-on Practice|
|---|---|---|
|**1**|**Introduction + Core Concepts** (Pod, ReplicaSet, Deployment, Namespace, Services basics)|Create a pod manually. Scale a ReplicaSet. Expose with ClusterIP. Break things (wrong image, wrong port) → debug with `kubectl describe`, `logs`.|
|**2**|**Scheduling** (Manual scheduling, Labels & Selectors, Taints & Tolerations, NodeSelector, Affinity)|Schedule a pod to a specific node. Add taint & test toleration. Use `kubectl label` to change node labels.|
|**3**|**Logging & Monitoring** (Cluster monitoring, Application logs, Metrics Server)|Install Metrics Server (or use `kubectl top`). Monitor pod resource usage. Simulate high CPU → see OOMKill.|
|**4**|**Application Lifecycle Management** (Rollout, Rollback, Autoscaling, ConfigMap, Secrets)|Perform rolling update & rollback. Create ConfigMap/Secret and mount as volume or env var. Test wrong secret name → error.|
|**5**|**Cluster Infrastructure** (ETCD, Kube-api, Kube-scheduler, Kube-controller-manager – **understand components, not deep details** )|Use `kubectl cluster-info dump`. Explore `/etc/kubernetes/manifests/` (static pods). Start a pod from static manifest.|
|**6**|**Installation & Configuration** (Kubeadm cluster setup basics – Mumshad covers `kubeadm` init, join, upgrade)|Practice `kubeadm init` on a fresh VM (or use killercoda.com). Then `kubeadm join` a worker. Break a node → repair.|
|**7**|**Week 1 Review – Mini Hackathon**|**No new topics.** Rebuild a broken cluster from scratch. Solve 5 random scenarios from Mumshad’s practice tests. Fix errors like `CrashLoopBackOff`, `ImagePullBackOff`, `NodeNotReady`.|

---

## 📅 Week 2 – Networking, Storage, Security & Troubleshooting (Days 8–14)[](https://terminal-hrpsernhcboizpeq.labs.kodekloud.com/#week)

| Day    | Topics                                                                                                                                | Hands-on Practice                                                                                                                  |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **8**  | **Networking Basics** (Cluster networking, Pod networking, CNI, Service types – NodePort, LoadBalancer, Ingress)                      | Install Calico/Flannel. Create a NodePort service. Access pod from outside. Debug `kubectl port-forward`.                          |
| **9**  | **Service & DNS** (DNS in Kubernetes, Service discovery, Network Policies)                                                            | Create a simple app & test `service-name.namespace.svc.cluster.local`. Write a NetworkPolicy that blocks traffic → test with `nc`. |
| **10** | **Storage** (Volumes, PersistentVolumes, PVC, StorageClass, StatefulSet basics)                                                       | Create PV & PVC. Mount in a pod. Delete PVC → see pending status. Create a StatefulSet with persistent storage.                    |
| **11** | **Security: RBAC & Service Accounts** (Roles, ClusterRoles, RoleBindings, ServiceAccounts)                                            | Create a ServiceAccount. Bind a Role that can only get pods. Test `kubectl auth can-i`. Break by wrong binding → error.            |
| **12** | **Security: Certificates & TLS** (kubeconfig, contexts, certificates, TLS for Ingress)                                                | Generate a new kubeconfig user. Create a self-signed TLS cert for Ingress. Use `kubectl config use-context` to switch.             |
| **13** | **Troubleshooting** (Common scenarios: pod pending, CrashLoop, node not ready, DNS failure, kubelet not running, etcd backup/restore) | **Hands-on marathon:** Simulate 10+ real problems. Use `kubectl describe`, `logs`, `exec`, `crictl`, `journalctl -`                |