# **Modernisation de l'Infrastructure HFSQL avec Kubernetes**

Ce projet a pour objectif de démontrer la faisabilité et les avantages de conteneuriser les bases de données HFSQL tout en assurant leur haute disponibilité, leur sécurité et leur résilience.

---

## **Objectifs du Projet**

- Trouver une solution pour isoler les instances HFSQL via des conteneurs.
- Assurer la haute disponibilité (HA) en cas de panne d’un serveur.
- Mettre en place une sauvegarde et une réplication des données.
- Sécuriser les serveurs Linux et les conteneurs.

---

## **Technologies Utilisées**

- **Kubernetes RKE2** : Distribution Kubernetes durcie pour la sécurité et la stabilité.
- **Rancher** : Gestion simplifiée et unifiée des clusters Kubernetes.
- **Longhorn** : Stockage persistant avec réplication et snapshots.
- **MetalLB** : Load Balancer open-source pour Kubernetes.
- **Helm** : Déploiement des applications HFSQL.
- **Zabbix, Prometheus & Grafana** : Monitoring de l’infrastructure.

---

## **Architecture de l'Infrastructure**

![Architecture de l'Infrastructure HFSQL](images/arch_base.jpg)

### **Zoom sur le service load balancing**

![LB](images/zoomlb.jpg)

### **Zoom sur le noeud master**

![NM](images/zoomnm.jpg)

### **Zoom sur le stockage longhorn**

![LH](images/zoomlh.jpg)


---

## **Installation de l'Infrastructure**

### **1. Préparation des Nœuds**
```bash
# Mettre à jour et installer les dépendances
apt update && apt upgrade -y
apt install -y curl gnupg lsb-release software-properties-common \
nfs-common open-iscsi iptables libnetfilter-conntrack3 conntrack \
policycoreutils cryptsetup

# Désactiver le swap
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Activer le routage IP
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
sysctl --system
```

### **2. Installation de RKE2 sur le master**
```bash
mkdir -p /etc/rancher/rke2/
# Changer le token
cat << EOF >> /etc/rancher/rke2/config.yaml
token: rke2SecurePassword  
EOF
curl -sfL https://get.rke2.io | INSTALL_RKE2_CHANNEL=v1.29 INSTALL_RKE2_TYPE=server sh -
systemctl enable rke2-server.service && systemctl start rke2-server.service
```
### **3. Installation de Kubectl**
```bash
ln -s /var/lib/rancher/rke2/data/v1*/bin/kubectl /usr/bin/kubectl
sudo ln -s /var/run/k3s/containerd/containerd.sock /var/run/containerd/containerd.sock
cat << EOF >> ~/.bashrc
export PATH=$PATH:/var/lib/rancher/rke2/bin:/usr/local/bin/
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export CRI_CONFIG_FILE=/var/lib/rancher/rke2/agent/etc/crictl.yaml
alias k=kubectl
EOF
source ~/.bashrc
```

### **4. Installation de RKE2 sur les Workers**
```bash
mkdir -p /etc/rancher/rke2/

# Changer l’IP du master et le token saisis auparavant
cat << EOF >> /etc/rancher/rke2/config.yaml
server: https://10.0.0.15:9345  # Adresse IP du serveur de contrôle RKE2
token: rke2SecurePassword     # Jeton d'authentification partagé
EOF

curl -sfL https://get.rke2.io | INSTALL_RKE2_CHANNEL=v1.29 INSTALL_RKE2_TYPE=agent sh –
systemctl enable rke2-agent.service && systemctl start rke2-agent.service
```
#### **Vérification que les noeuds sont bien en fonctionnement**
![LH](images/verifnodes.jpg)




