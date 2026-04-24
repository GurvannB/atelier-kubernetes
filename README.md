# Atelier Kubernetes

app.js:
```javascript
const fs = require('fs');
const express = require('express');
const app = express();
// Lecture fichier de configuration
let config = {};
try {
  const rawConfig = fs.readFileSync('/app/config/config.json');
  config = JSON.parse(rawConfig);
} catch (err) {
  console.log("No config file found, using defaults");
}
// Variables d'environnement (secrets)
const dbUser = process.env.DB_USER || "default_user";
const dbPassword = process.env.DB_PASSWORD || "default_password";
app.get('/', (req, res) => {
  res.json({
    message: config.message || "Hello World",
    database: {
      user: dbUser,
      password: dbPassword ? "******" : "not set"
    }
  });
});
app.listen(3000, () => {
  console.log("Server started on port 3000");
});
```

## Étape 1 — Packaging Docker

## Objectif
Créer une image Docker fonctionnelle de l’application.

### Travail à réaliser
- Écrire un Dockerfile permettant de :
  - Installer les dépendances nécessaires
  - Copier le code source
  - Démarrer l’application
- Construire l’image Docker
- Tester localement le fonctionnement du conteneur

### Contraintes
- L’application doit être accessible sur le port 3000
- Le conteneur ne doit pas contenir de configuration en dur

## Étape 2 — Déploiement avec un Deployment Kubernetes

### Objectif
Déployer votre conteneur dans Kubernetes.

### Travail à réaliser
- Créer un manifest Kubernetes de type Deployment
- Déployer au moins 2 replicas
- Exposer l’application (Service Kubernetes)

### Vérifications attendues
- Les pods doivent être en état Running
- L’application doit être accessible depuis l’extérieur du cluster

## Étape 3 — Configuration avec ConfigMap

### Objectif
Externaliser la configuration applicative.

### Travail à réaliser
- Créer un ConfigMap contenant un fichier config.json
- Monter ce fichier dans le conteneur à l’emplacement attendu : /app/config/config.json
- Adapter votre Deployment pour utiliser ce ConfigMap

### Vérifications attendues
- Modifier le ConfigMap doit modifier le comportement de l’application
- Aucun rebuild de l’image ne doit être nécessaire

## Étape 4 — Gestion des secrets

### Objectif
Sécuriser les données sensibles.

### Travail à réaliser
- Créer un Secret Kubernetes
- Y stocker les variables suivantes :
  - DB_USER
  - DB_PASSWORD
- Injecter ces variables dans le conteneur via des variables d’environnement

### Vérifications attendues
- L’application doit refléter les valeurs injectées
- Les données ne doivent pas apparaître en clair dans les manifests

## Étape 5 — Déploiement en GitOps avec ArgoCD

### Objectif
Mettre en place un déploiement automatisé basé sur Git.

### Travail à réaliser
- Créer un dépôt Git contenant :
  - Les manifests Kubernetes (Deployment, Service, ConfigMap, Secret)
- Installer et configurer Argo CD
- Déclarer une application ArgoCD pointant vers votre dépôt
- Déployer automatiquement l’application dans le cluster

### Vérifications attendues
- Toute modification du dépôt Git doit être répercutée automatiquement
- L’état du cluster doit être synchronisé avec le dépôt

## Étape 6 — Bonus - Déploiement multienvironnement

### Objectif
Mettre en place un déploiement sur plusieurs environnements, développement et
production.

### Travail à réaliser
A l'aide de l'architecture construite dans les étapes précédentes, proposez une méthode
pour gérer le déploiement dans plusieurs environnement et gérer le cycle de vie de
l'application dans git.

### Vérifications attendues
- Réaliser de manière simple une évolution qui est d'abord déployée en dev, puis
validé en prod en explicitant les étapes git.

## Livrables attendus
- Dépôt Git contenant :
  - Code source
  - Dockerfile
  - Manifests Kubernetes
- Captures d’écran ou preuves de :
  - Déploiement fonctionnel
  - Mise à jour via ConfigMap
  - Injection des secrets
  - Synchronisation ArgoCD
  - 
## Remarques
- Vous devez rechercher par vous-même les syntaxes YAML et commandes nécessaires
- L’utilisation de la documentation officielle est encouragée
- Aucun copier-coller complet de solution n’est attendu, mais une compréhension réelle des concepts