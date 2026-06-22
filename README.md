# 📝 Todo App — Kubernetes / Minikube

Application web Todo déployée sur Kubernetes avec Minikube, composée d'un frontend Nginx, d'une API Flask et d'une base de données PostgreSQL.

---

## Architecture

```
Navigateur
    |
    | http://localhost:8080
    v
Frontend (Nginx)
    |
    | Appel HTTP vers /api/
    v
Backend API (Flask)
    |
    | Connexion PostgreSQL
    v
Base de données PostgreSQL
```

---

## Structure du projet

```
todo-k8s-app/
├── frontend/
│   ├── Dockerfile
│   ├── index.html
│   ├── app.js
│   └── nginx.conf
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── main.py
└── k8s/
    ├── namespace.yaml
    ├── configmap.yaml
    ├── secret.yaml
    ├── postgres-pvc.yaml
    ├── postgres-deployment.yaml
    ├── postgres-service.yaml
    ├── backend-deployment.yaml
    ├── backend-service.yaml
    ├── frontend-deployment.yaml
    └── frontend-service.yaml
```

---

## Ressources Kubernetes utilisées

| Ressource | Nom | Rôle |
|---|---|---|
| Namespace | `todo-app` | Isole toutes les ressources de l'application dans un espace dédié, évitant les conflits avec d'autres projets sur le cluster |
| ConfigMap | `todo-config` | Stocke les variables de configuration non sensibles (nom de la base, utilisateur, port, host) et les injecte dans les pods |
| Secret | `todo-secret` | Stocke le mot de passe PostgreSQL de façon sécurisée (encodé en base64), sans l'écrire en clair dans les Deployments |
| PersistentVolumeClaim | `postgres-pvc` | Réserve un espace de stockage persistant pour PostgreSQL, pour que les données survivent au redémarrage des pods |
| Deployment | `postgres` | Gère le pod PostgreSQL et assure qu'il reste toujours en cours d'exécution |
| Deployment | `backend` | Gère le pod de l'API Flask et lui injecte les variables d'environnement pour se connecter à PostgreSQL |
| Deployment | `frontend` | Gère le pod Nginx qui sert l'interface web et redirige les appels `/api/` vers le backend |
| Service | `postgres-service` | Expose PostgreSQL en interne au cluster uniquement, accessible par le backend via son nom DNS |
| Service | `backend-service` | Expose l'API Flask en interne, accessible par le frontend via Nginx |
| Service | `frontend-service` | Expose le frontend pour permettre le port-forward et l'accès depuis le navigateur |

---

## Déploiement

### Prérequis

- [Minikube](https://minikube.sigs.k8s.io/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Docker](https://www.docker.com/)

### Étapes

```bash
# 1. Démarrer Minikube
minikube start

# 2. Construire les images dans Minikube
minikube image build -t todo-frontend:1.0 ./frontend
minikube image build -t todo-backend:1.0 ./backend

# 3. Appliquer les manifests Kubernetes
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/postgres-pvc.yaml
kubectl apply -f k8s/postgres-deployment.yaml
kubectl apply -f k8s/postgres-service.yaml
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/frontend-service.yaml

# 4. Vérifier que tout est Running
kubectl get all -n todo-app
```

---

## Accès à l'application

### Frontend

```bash
kubectl port-forward -n todo-app service/frontend-service 8080:80
```

Ouvrir : [http://localhost:8080](http://localhost:8080)

### Backend (test API)

```bash
kubectl port-forward -n todo-app service/backend-service 5000:5000
```

```bash
# Lister les tâches
curl http://localhost:5000/tasks

# Ajouter une tâche
curl -X POST http://localhost:5000/tasks \
  -H "Content-Type: application/json" \
  -d "{\"title\":\"Apprendre Kubernetes\"}"
```

---

## Commandes de vérification

```bash
kubectl get all -n todo-app
kubectl get pvc -n todo-app
kubectl get configmap -n todo-app
kubectl get secret -n todo-app
```

---

## Nettoyage

```bash
kubectl delete namespace todo-app
minikube stop
```
