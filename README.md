# Mini-projet Docker — POZOS Student List

**Nom : Ozbag Yunus**

---

## 🎯 Objectif du projet

L’objectif de ce mini-projet est de dockeriser l’application **POZOS Student List** en mettant en place :

* Une image Docker pour l’API (Flask)
* Un déploiement complet via `docker-compose` (API + Website)
* Un registre Docker privé avec une interface web
* Une livraison documentée avec preuves de fonctionnement

---

## 🧱 Architecture du projet

* **API** : Flask (Python)
* **Website** : PHP (image `php:apache`)
* **Orchestration** : Docker Compose
* **Registry privé** : Docker Registry v2 + UI web

---

## 📁 Structure du dépôt

```
mini-projet-docker/
├── README.md
├── docker-compose.yml
├── simple_api/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── student_age.py
│   └── student_age.json
├── website/
│   └── (fichiers du site POZOS)
└── registry/
    └── docker-compose.registry.yml
```

---

## 1️⃣ Construction de l’image API

### Dockerfile

Fichier : `simple_api/Dockerfile`

Caractéristiques :

* Image de base : `python:3.13-slim`
* Installation des dépendances système (LDAP, SSL, outils de build)
* Installation des dépendances Python
* Exposition du port `5000`
* Volume `/data` pour le fichier `student_age.json`

### Build de l’image

```bash
docker build -t student-list-api ./simple_api
```

### Test de l’API

```bash
docker run -d --name student-api \
  -p 5000:5000 \
  -v "$PWD/simple_api/student_age.json:/data/student_age.json:ro" \
  student-list-api

curl -u toto:python http://localhost:5000/pozos/api/v1.0/get_student_ages
```

📸 **Screenshot 1 — API fonctionnelle (curl + réponse JSON)**
*(à insérer ici)*

---

## 2️⃣ Déploiement via Docker Compose (API + Website)

### Fichier : `docker-compose.yml`

* **API**

  * Image : `student-list-api`
  * Port exposé : `5000`
  * Montage du fichier JSON dans `/data/student_age.json`
* **Website**

  * Image : `php:apache`
  * Variables d’environnement `USERNAME` et `PASSWORD`
  * Montage du site via `./website:/var/www/html`
  * Dépend de l’API
* Réseau Docker dédié

### Lancement

```bash
docker-compose up -d
```

### Test

Accès au site :

* [http://localhost:8080](http://localhost:8080)

Cliquer sur **“List Student”** pour afficher la liste des étudiants.

📸 **Screenshot 2 — Website affichant la liste des étudiants**
*(à insérer ici)*

---

## 3️⃣ Mise en place d’un Docker Registry privé

### Fichier : `registry/docker-compose.registry.yml`

Composants :

* **Registry** : `registry:2`
* **Interface web** : `joxit/docker-registry-ui`
* Persistance des images via un volume Docker
* Activation du CORS pour l’accès navigateur
* Réseau dédié

### Lancement

```bash
cd registry
docker-compose -f docker-compose.registry.yml up -d
```

Accès à l’UI :

* [http://192.168.56.103:8081](http://192.168.56.103:8081)

---

## 4️⃣ Push de l’image API dans le registry privé

### Configuration Docker (registry HTTP)

Ajout du registry en insecure registry :

Fichier `/etc/docker/daemon.json`

```json
{
  "insecure-registries": ["192.168.56.103:5001"]
}
```

Redémarrage de Docker :

```bash
sudo systemctl restart docker
```

### Tag et push

```bash
docker tag student-list-api:latest 192.168.56.103:5001/student-list-api:1.0
docker push 192.168.56.103:5001/student-list-api:1.0
```

### Vérification

```bash
curl http://192.168.56.103:5001/v2/_catalog
curl http://192.168.56.103:5001/v2/student-list-api/tags/list
```

📸 **Screenshot 3 — UI du registry affichant l’image poussée**
*(à insérer ici)*

---

## ✅ Conclusion

L’application POZOS a été entièrement dockerisée avec succès :

* API fonctionnelle dans un conteneur Docker
* Website accessible via Docker Compose
* Images stockées et visibles dans un registre Docker privé

Ce projet permet de reproduire facilement l’environnement complet via Docker.

---

## 📌 Commandes utiles

```bash
docker-compose ps
docker-compose logs -f
docker logs student-api
docker logs student-website
docker logs private-registry
docker logs registry-ui
```
