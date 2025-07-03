+++
title = "OpenShift Single Node (SNO) Installation Guide"
weight = 1
+++

# 🧰 OpenShift SNO Installation Guide

Single Node OpenShift (SNO) is a deployment topology where the entire OpenShift control plane and compute roles are hosted on a **single node**. It's ideal for **edge computing**, **development**, and **lightweight clusters**.

---

## 📋 Prerequisites

| Requirement                  | Details                                                |
|-----------------------------|--------------------------------------------------------|
| **Hardware**                | 8+ vCPU, 32+ GB RAM, 120+ GB disk (SSD preferred)     |
| **OS**                      | RHEL 8.6/9.x or Red Hat CoreOS (RHCOS)                |
| **Access**                  | Internet or local mirrored registry                    |
| **Tools**                   | `openshift-install`, `oc`, `podman`, `jq`             |
| **Network**                 | DHCP or static IP; FQDN DNS entry for API and Ingress |

---

## 🔧 Tools Setup

### Download OpenShift Installer and CLI
```bash
# Set version
OCP_VERSION=4.14.20

# Download installer
curl -LO "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-install-linux.tar.gz"
tar -xvf openshift-install-linux.tar.gz

# Download CLI
curl -LO "https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-client-linux.tar.gz"
tar -xvf openshift-client-linux.tar.gz
```

---

## 🏗️ Create Installation Configuration

```bash
./openshift-install create install-config --dir sno-cluster
```

### Sample `install-config.yaml` for SNO:
```yaml
apiVersion: v1
baseDomain: example.com
metadata:
  name: sno
compute:
- name: worker
  replicas: 0
controlPlane:
  name: master
  replicas: 1
platform:
  none: {}  # Bare metal
pullSecret: '<your-pull-secret>'
sshKey: '<your-ssh-public-key>'
```

---

## 📦 Generate ISO for Installation

```bash
./openshift-install coreos print-stream-json > stream.json
```

Use tools like `coreos-installer` or PXE boot to deploy the RHCOS image to your node.

---

## 💿 Installing RHCOS (on the SNO node)

### Option 1: USB boot or PXE boot into RHCOS live ISO  
Once booted:

```bash
# Example
coreos-installer install /dev/sda \
  --image-url=<URL-to-live-image> \
  --ignition-url=http://<http-server>/ignition-config.ign
```

---

## 🚀 Bootstrap and Install Cluster

```bash
./openshift-install create manifests --dir sno-cluster
./openshift-install create ignition-configs --dir sno-cluster
```

Copy ignition files to an HTTP server accessible by the node.

Then install OpenShift:
```bash
./openshift-install wait-for install-complete --dir sno-cluster --log-level=debug
```

---

## 🌐 Access the Cluster

After installation completes:

- **Console URL**: `https://console-openshift-console.apps.sno.example.com`
- **API URL**: `https://api.sno.example.com:6443`

Get admin credentials:
```bash
cat sno-cluster/auth/kubeadmin-password
```

Login:
```bash
oc login -u kubeadmin -p <password> https://api.sno.example.com:6443
```

---

## 🧼 Cleanup

To delete and recreate:
```bash
rm -rf sno-cluster
```

---

## 📚 References

- [Official OpenShift SNO Docs](https://docs.openshift.com/container-platform/latest/scalability_and_performance/using-sno.html)
- [Bare Metal UPI Installation Guide](https://docs.openshift.com/container-platform/latest/installing/installing_bare_metal/installing-bare-metal.html)

---

> ✅ Tip: SNO is a great way to run OpenShift at the edge or in constrained environments.
