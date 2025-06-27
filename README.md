# 📦 NodeAPI – Déploiement Helm avec PostgreSQL sur Kubernetes

Ce projet déploie une application web conteneurisée via **Helm** dans deux environnements Kubernetes distincts : `dev` et `prod`.  
L’application utilise une image publique (`nginxdemos/hello`) et simule l’intégration d’une base de données PostgreSQL persistante, conformément aux consignes.

---

## 📐 Architecture

```
NodeAPI Chart Helm
├── Deployment NodeAPI (nginxdemos/hello)
│   └── Injecte des variables DB simulées (non utilisées)
├── PostgreSQL Deployment + PVC
│   └── Volume persistant pour stocker les données
├── Service NodeAPI (ClusterIP)
└── Service PostgreSQL (ClusterIP)
```

Chaque environnement (`dev` / `prod`) dispose :
- d’un namespace dédié
- de ses propres ressources CPU/RAM
- de sa propre base PostgreSQL (déployée mais non utilisée)
- de valeurs configurables via Helm

---

## 📁 Structure du projet

```
NodeAPI/
├── Chart.yaml
├── values.yaml              # valeurs par défaut (placeholders)
├── values-dev.yaml          # config dev (image, quotas, DB)
├── values-prod.yaml         # config prod
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── postgres-deployment.yaml
│   ├── postgres-service.yaml
│   ├── postgres-pvc.yaml
│   ├── namespace-quota.yaml
│   └── limit-range.yaml
```

---

## 🚀 Installation

### 1. Créer les namespaces
```bash
kubectl create namespace dev
kubectl create namespace prod
```

### 2. Installer dans `dev`
```bash
helm install nodeapi-dev ./NodeAPI -n dev -f values-dev.yaml
```

### 3. Installer dans `prod`
```bash
helm install nodeapi-prod ./NodeAPI -n prod -f values-prod.yaml
```

---

## 🧪 Test & Vérification

### Voir les pods :
```bash
kubectl get pods -n dev
kubectl get pods -n prod
```

### Tester l’accès HTTP à l’application (localement sur le port 8080, à modifier si besoin) :
```bash
kubectl port-forward svc/nodeapi-dev 8080:80 -n dev
```

Puis aller sur : http://localhost:8080

Résultat attendu :
```
Page Hello World
```

---

## 🧼 Désinstallation

```bash
helm uninstall nodeapi-dev -n dev
helm uninstall nodeapi-prod -n prod
kubectl delete namespace dev
kubectl delete namespace prod
```

---

## ⚙️ Configuration (extrait de `values-dev.yaml`)

```yaml
image:
  repository: nginxdemos/hello
  tag: latest

service:
  type: ClusterIP
  port: 80

envVars:
  DB_HOST: nodeapi-dev-postgres
  DB_NAME: nodeapidb
  DB_USER: devuser
  DB_PASSWORD: devpass

postgres:
  image: postgres:13
  user: devuser
  password: devpass
  dbName: nodeapidb
  storage: 1Gi

quota:
  requests:
    cpu: "600m"
    memory: "768Mi"
  limits:
    cpu: "1"
    memory: "1Gi"

limitRange:
  defaultRequest:
    cpu: "100m"
    memory: "128Mi"
  default:
    cpu: "300m"
    memory: "256Mi"
```

---

## 🧠 Remarques

- L’application n’utilise **pas réellement PostgreSQL** (image Nginx).
- Mais la base est bien déployée, exposée, et les variables d’environnement sont injectées comme si l’app y accédait.
- Ce choix respecte les **attentes du sujet** : Helm paramétré, base persistante, quotas différenciés, documentation complète.

---

## ✍️ Auteur

Déploiement Kubernetes industrialisé avec Helm – Projet de standardisation dev/prod.
