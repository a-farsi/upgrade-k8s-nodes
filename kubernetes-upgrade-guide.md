# Kubernetes Cluster Upgrade Guide

## Objectif

Mettre à jour un cluster Kubernetes d’une version **v1.35.2 vers v1.35.3**

---

## 1. Comprendre les composants

### 1.1 Control Plane Components

Les composants du control plane assurent la gestion du cluster.

| Composant                | Rôle |
|--------------------------|------|
| kube-apiserver          | Point d’entrée de toutes les requêtes |
| kube-controller-manager | Gère les états désirés (replicas, etc.) |
| kube-scheduler          | Décide sur quel nœud placer les pods |
| etcd                    | Base de données du cluster |

Mise à jour via :
```
kubeadm upgrade apply
```

---

### 1.2 Node Components

Les composants de nœud permettent l’exécution des pods.

| Composant | Rôle |
|----------|------|
| kubelet  | Agent qui gère les pods sur le nœud |
| kubectl  | Outil en ligne de commande |

Mise à jour via :
```
apt-get install
```

---

## 2. Principe de l’upgrade

La commande suivante :

```
kubeadm upgrade apply v1.35.3
```

Met à jour automatiquement :

- kube-apiserver  
- kube-controller-manager  
- kube-scheduler  
- etcd  

En revanche, les composants suivants ne sont pas mis à jour automatiquement :

- kubelet  
- kubectl  

Ils doivent être mis à jour manuellement.

L’upgrade se déroule donc en deux phases :

```
kubeadm upgrade apply v1.35.3
apt-get install -y kubelet=1.35.3-00 kubectl=1.35.3-00
```

---

## 3. Upgrade du Control Plane (Master Node)

### 3.1 Mettre à jour kubeadm

```
apt-get update
apt-get install -y kubeadm=1.35.3-00
```

### 3.2 Vérifier le plan d’upgrade

```
kubeadm upgrade plan
```

### 3.3 Appliquer l’upgrade

```
kubeadm upgrade apply v1.35.3
```

### 3.4 Drainer le nœud master

```
kubectl drain controlplane --ignore-daemonsets
```

### 3.5 Mettre à jour kubelet et kubectl

```
apt-get install -y kubelet=1.35.3-00 kubectl=1.35.3-00
systemctl daemon-reexec
systemctl restart kubelet
```

### 3.6 Rendre le nœud de nouveau schedulable

```
kubectl uncordon controlplane
```

---

## 4. Upgrade des Worker Nodes

À effectuer nœud par nœud.

### 4.1 Drainer le nœud

```
kubectl drain <worker-node> --ignore-daemonsets --delete-emptydir-data
```

### 4.2 Mettre à jour kubeadm

```
apt-get update
apt-get install -y kubeadm=1.35.3-00
```

### 4.3 Mettre à jour le nœud

```
kubeadm upgrade node
```

### 4.4 Mettre à jour kubelet et kubectl

```
apt-get install -y kubelet=1.35.3-00 kubectl=1.35.3-00
systemctl daemon-reexec
systemctl restart kubelet
```

### 4.5 Rendre le nœud schedulable

```
kubectl uncordon <worker-node>
```

---

## 5. Vérification

```
kubectl get nodes
```

Résultat attendu :

```
NAME           STATUS   VERSION
controlplane   Ready    v1.35.3
worker1        Ready    v1.35.3
worker2        Ready    v1.35.3
```

---

## 6. Bonnes pratiques

- Mettre à jour le control plane avant les workers  
- Mettre à jour les nœuds un par un  
- Toujours utiliser `kubeadm upgrade plan` avant l’upgrade  
- Respecter l’ordre : kubeadm → kubelet → kubectl  
- Utiliser `drain` avant et `uncordon` après chaque mise à jour  
