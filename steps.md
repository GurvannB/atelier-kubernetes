# Étapes

## Étape 1

Écriture du code dans le fichier `app.js` :

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

Initialisation d'un projet Node.js et installation d'Express :

```bash
npm init -y
npm install express
```

Rédaction du Dockerfile (Fichier Dockerfile)

Build de l'image Docker :

```bash
docker build -t gurvannbrenne/app:1.0.0 .
```

Test local du conteneur :

```bash
docker run --rm --name=node-app-test-container -p 3000:3000 -e DB_USER=test_user -e DB_PASSWORD=test_password gurvannbrenne/app:1.0.0
curl http://localhost:3000
``` 

On attend le résultat suivant : 

```json
{"message":"Hello World","database":{"user":"test_user","password":"******"}}
```

On peut arrêter le container :

```bash
docker stop node-app-test-container
```

## Étape 2

Création du manifest Kubernetes (Fichier: deployment.yaml)

Déploiement dans Kubernetes :

Pour que kubernetes puisse trouver l’image, il faut d’abord la pousser sur un registry accessible

```bash
docker login # Se connecter à Docker Hub ou un autre registry
docker push gurvannbrenne/app:1.0.0 # Push l'image sur le registry
kubectl apply -f deployment.yaml # Appliquer le manifest Kubernetes (donc créer le Deployment)
kubectl get deployments # Vérifie que le Deployment a été créé
kubectl get pods # Vérifie qu'il y a bien 2 pods en état Running
```

On expose ensuite l'application avec un Service Kubernetes (Fichier: service.yaml)

```bash
kubectl apply -f service.yaml # Appliquer le manifest du Service
kubectl get services # Vérifie que le Service a été créé
```

On vérifie ensuite la connexion à l’application depuis un autre pod :

```bash
kubectl run app-test-service --image=nicolaka/netshoot -it --rm -- /bin/bash
curl http://app-service
```

On doit de nouveau retrouver le JSON de tout à l'heure

Accéder à l'app depuis l'extérieur du cluster :

Pour cela, on met en place MetalLB

On change le deployment en LoadBalancer en ajoutant "type: LoadBalancer" dans le manifest du service

```bash
kubectl apply -f service.yaml # Appliquer le manifest du Service modifié
kubectl get services # Vérifie que le Service a été mis à jour et n'a pas encore d'IP externe (<pending>)
```

On applique les fichiers de configuration de MetalLB (Fichier: ip-address-pool.yaml & l2advertissement.yaml)

La plage d'adresses IP doit être adaptée à l'environnement. Ici elle est de 192.168.64.100-192.168.64.150 car mon k3s est sur 192.168.3/24

```bash
kubectl apply -f ip-address-pool.yaml
kubectl apply -f l2advertissement.yaml
kubectl get ipaddresspools.metallb.io -n metallb-system # Vérifie que le pool d'adresses IP a été créé
kubectl get l2advertisements.metallb.io -n metallb-system # Vérifie que la configuration d'annonce a été créée
kubectl get services # Vérifie que le Service a maintenant une IP externe
```

Puis on accède à l'application avec le lien http://<IP_EXTERNE> (dans mon cas: http://192.168.64.100)

