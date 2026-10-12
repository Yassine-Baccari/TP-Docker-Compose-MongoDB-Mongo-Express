# TP Docker Compose – MongoDB + Mongo Express

⏱️ **Durée estimée : 1 heure**
🎯 **Niveau : débutant en Docker Compose** (connaissances de base de `docker run` conseillées)

---

## 🎯 Objectifs pédagogiques

À la fin de ce TP, vous serez capable de :

1. Expliquer ce qu'est **Docker Compose** et pourquoi l'utiliser ;
2. Lire et écrire un fichier `docker-compose.yml` (services, image, ports, environnement, dépendances) ;
3. Lancer, arrêter et superviser une application **multi-conteneurs** ;
4. Comprendre le **réseau commun** créé automatiquement par Compose.

---

## 📚 Rappel de cours

**Docker Compose** est un outil qui permet de **définir et lancer plusieurs conteneurs Docker** à partir d'un seul fichier **YAML**.

| Avec `docker run` | Avec Docker Compose |
|---|---|
| Une longue commande par conteneur | Un seul fichier déclaratif |
| Options à retaper à chaque fois | Configuration versionnable (Git) |
| Réseau à créer manuellement (`--net`) | **Réseau commun créé automatiquement** |
| Difficile à maintenir | Facile à modifier et à partager |

💡 Tous les services d'un même fichier Compose sont placés dans **le même réseau Docker**. Ils peuvent donc se joindre **en utilisant le nom du service comme nom d'hôte** (ex. `mongo`). C'est l'équivalent automatique de l'option `--net` de `docker run`.

---

## ✅ Partie 0 – Prérequis (5 min)

Vérifiez que Docker et Docker Compose sont installés :

```bash
docker --version
docker compose version
```

Créez un dossier de travail :

```bash
mkdir tp-compose
cd tp-compose
```

> ⚠️ Le nom du dossier est important : Compose l'utilise comme **préfixe** pour nommer les conteneurs et le réseau (ici `tp-compose`).

---

## 📝 Partie 1 – Créer le fichier Compose (10 min)

Dans le dossier `tp-compose`, créez un fichier nommé **`docker-compose.yml`** avec le contenu suivant :

```yaml
services:
    mongo:
        image: mongo
        environment:
            - MONGO_INITDB_ROOT_USERNAME=admin
            - MONGO_INITDB_ROOT_PASSWORD=admin
    mongo-express:
        image: mongo-express
        ports:
            - 8081:8081
        environment:
            - ME_CONFIG_BASICAUTH_USERNAME=test
            - ME_CONFIG_BASICAUTH_PASSWORD=test
            - ME_CONFIG_MONGODB_SERVER=mongo
            - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
            - ME_CONFIG_MONGODB_ADMINPASSWORD=admin
        depends_on:
            - mongo
```

> ⚠️ **YAML est sensible à l'indentation** : utilisez des **espaces** (jamais de tabulations) et gardez une indentation cohérente.

### ❓ Questions de compréhension

Répondez dans un fichier `reponses.md` :

1. Combien de **services** sont définis ? Lesquels ?
2. Quelle image Docker utilise chaque service ?
3. Quel service est accessible depuis votre navigateur ? Sur quel port de votre machine ?
4. À quoi sert `depends_on` ?
5. Que représente la valeur `mongo` dans `ME_CONFIG_MONGODB_SERVER=mongo` ?

---

## 🚀 Partie 2 – Lancer l'application (10 min)

### 2.1 Démarrage

```bash
docker compose up -d
```

- `up` : crée et démarre les services ;
- `-d` : mode *detached* (en arrière-plan).

Observez les lignes affichées : Compose télécharge les images, crée **un réseau** puis démarre les conteneurs.

### 2.2 Vérification

```bash
docker compose ps
```

Les deux services doivent être à l'état `running` (ou `Up`).

### 2.3 Consulter les logs

```bash
docker compose logs mongo-express
docker compose logs -f        # suivre tous les logs en direct (Ctrl+C pour quitter)
```

### ✍️ À faire

- Relevez les **noms** des deux conteneurs. Quel est le format utilisé ?
- Au bout de quelques secondes, quel message indique que Mongo Express est connecté à MongoDB ?

> 💡 Si `mongo-express` redémarre au début, c'est normal : MongoDB met quelques secondes à être prêt. `depends_on` garantit seulement **l'ordre de démarrage**, pas que la base soit *prête*.

---

## 🌐 Partie 3 – Utiliser Mongo Express (10 min)

1. Ouvrez votre navigateur sur : **http://localhost:8081**
2. Identifiez-vous avec :
   - Utilisateur : `test`
   - Mot de passe : `test`
3. Dans l'interface :
   - Créez une base de données nommée **`ecole`** ;
   - Dans cette base, créez une collection **`etudiants`** ;
   - Ajoutez un document, par exemple :

```json
{
  "nom": "Dupont",
  "prenom": "Alice",
  "filiere": "Informatique"
}
```

4. Ajoutez 2 autres documents de votre choix.

### 🔎 Vérification en ligne de commande

Connectez-vous directement à MongoDB dans le conteneur :

```bash
docker compose exec mongo mongosh -u admin -p admin --authenticationDatabase admin
```

Puis dans le shell Mongo :

```javascript
show dbs
use ecole
db.etudiants.find()
exit
```

✅ Vous devez retrouver les documents créés depuis l'interface web.

---

## 🔗 Partie 4 – Comprendre le réseau Compose (5 min)

### 4.1 Lister les réseaux

```bash
docker network ls
```

Repérez le réseau nommé **`tp-compose_default`** : il a été créé automatiquement par Compose.

### 4.2 Inspecter le réseau

```bash
docker network inspect tp-compose_default
```

Relevez les **adresses IP** des deux conteneurs.

### 4.3 Résolution de noms

Depuis le conteneur `mongo-express`, testez la communication avec `mongo` **par son nom** :

```bash
docker compose exec mongo-express ping -c 3 mongo
```

### ❓ Questions

1. Pourquoi `mongo-express` peut-il joindre `mongo` sans connaître son adresse IP ?
2. Le service `mongo` a-t-il un port publié (`ports:`) ? Peut-on y accéder depuis votre machine ? Pourquoi est-ce une bonne pratique de sécurité ?

---

## 🧩 Partie 5 – Exercices de consolidation (15 min)

Choisissez **au moins un** exercice :

1. **Changer le port** : faites en sorte que Mongo Express soit accessible sur le port **9090** de votre machine.
2. **Redémarrage automatique** : ajoutez `restart: unless-stopped` aux deux services. Testez avec `docker kill` sur un conteneur.
3. **Variables d'environnement** : déplacez les identifiants dans un fichier `.env` et utilisez la syntaxe `${MONGO_USER}` dans le fichier Compose.
4. **Publier MongoDB** : ajoutez `ports: - 27017:27017` à `mongo` et connectez-vous avec `mongosh` depuis votre machine (si installé). Discutez des risques.

---

## 🧹 Nettoyage

```bash
docker compose down        # supprime conteneurs + réseau
```

---

## 📋 Aide-mémoire des commandes

| Commande | Rôle |
|---|---|
| `docker compose up -d` | Crée et démarre les services en arrière-plan |
| `docker compose ps` | Liste les services du projet |
| `docker compose logs -f [service]` | Affiche les logs en continu |
| `docker compose exec <service> <cmd>` | Exécute une commande dans un conteneur en cours |
| `docker compose stop` / `start` | Arrête / relance sans supprimer |
| `docker compose restart` | Redémarre les services |
| `docker compose down` | Arrête et supprime conteneurs et réseau |
| `docker compose config` | Valide et affiche le fichier Compose final |

---

## 🛠️ Dépannage

| Problème | Piste de solution |
|---|---|
| `yaml: line X: found character that cannot start any token` | Tabulation ou mauvaise indentation dans le YAML |
| `port is already allocated` | Le port 8081 est déjà utilisé : changez le port côté hôte (`9090:8081`) |
| Mongo Express affiche une erreur de connexion | Attendez quelques secondes puis `docker compose restart mongo-express` |
| Page web inaccessible | Vérifiez `docker compose ps` et `docker compose logs mongo-express` |
| Les modifications du fichier ne sont pas prises en compte | Relancez `docker compose up -d` |

---

## ⚠️ Remarque sécurité

Les identifiants `admin/admin` et `test/test` sont acceptables **uniquement pour ce TP**. En production, il faut utiliser des mots de passe forts, stockés dans des **secrets** ou un fichier `.env` **non versionné**, et ne jamais exposer la base de données directement sur Internet.

---

## 📤 Livrable

Rendez :

- votre fichier `docker-compose.yml` final ;
- votre fichier `reponses.md` avec les réponses aux questions ;
- une capture d'écran de Mongo Express affichant la base `ecole` et ses documents.

**Bon travail ! 🐳**
