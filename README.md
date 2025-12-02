# kubernetes-setup

# Kubernetes Cluster Setup (Master + Worker Nodes)

This guide explains how to set up a Kubernetes cluster manually using **kubeadm**, **containerd**, and **Calico CNI** on Ubuntu servers.

---

## 📌 Architecture Used

* **1 Master Node** (Ubuntu)
* **1 Worker Node** (Ubuntu)
* Container Runtime → **containerd**
* Pod Network → **Calico (CIDR: 10.244.0.0/16)**

---

## 🔧 Required Ports (Open in Security Groups)

| Port      | Purpose               |
| --------- | --------------------- |
| **22**    | SSH                   |
| **6443**  | Kubernetes API Server |
| **10250** | Kubelet communication |

---

## 🛠 Step 1: Set Hostnames

### Master Node

```bash
sudo hostnamectl hostname master-node
```

### Worker Node

```bash
sudo hostnamectl hostname worker-node
```

---

## 🟦 Step 2: Run Common Script (On Both Master & Worker)

This installs:

* ✔ containerd
* ✔ kubeadm, kubelet, kubectl
* ✔ kernel modules
* ✔ disables swap

Save the script:

```bash
sudo vim k8s-common-containerd.sh
```

Run it:

```bash
sudo chmod +x k8s-common-containerd.sh
sudo ./k8s-common-containerd.sh
```

---

## 🟩 Step 3: Initialize Kubernetes Master Node

⚠️ Run only on **Master Node**

Save the script:

```bash
sudo vim k8s-master-init.sh
```

Run the script:

```bash
sudo chmod +x k8s-master-init.sh
sudo ./k8s-master-init.sh
```

This will:

* Initialize the master
* Configure kubeconfig
* Install Calico CNI
* Output a join token

After running, it will show a join command like:

```bash
kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Copy this command.

---

## 🟧 Step 4: Join Worker Node to Cluster

⚠️ Run only on **Worker Node**

Save script:

```bash
sudo vim k8s-worker-join.sh
```

Paste your token inside.

Run:

```bash
sudo chmod +x k8s-worker-join.sh
sudo ./k8s-worker-join.sh
```

---

## 🟦 Step 5: Verify Cluster

On Master Node:

```bash
kubectl get nodes -o wide
```

Expected Output:

```bash
master-node   Ready    Control-Plane
worker-node   Ready
```

---

## 🛠 Fix: kubectl Connection Error on Master

If you get an error like:

```bash
The connection to the server localhost:8080 was refused
```

Run:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

🎉 **Setup Complete!**
You now have a working Kubernetes cluster using **kubeadm + containerd + Calico**.
