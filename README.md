# TP5 - Docker Engine, Jenkins, CI/CD

> **Master II BDCC – Big Data et Cloud Computing**  
> Virtualisation, Cloud Computing, DevOps | 2025/2026

---

## 📋 Description

Pipeline **CI/CD complet** avec Jenkins et Docker : build automatique d'une image Docker et déploiement d'un conteneur nginx affichant une page web.

---

## ✅ Prérequis

- Docker installé et démarré
- Jenkins installé et accessible sur `http://localhost:8080`
- Un compte **GitHub**
- Un compte **Docker Hub**

---

## 🔧 Partie 1 – Intégration Continue (CI)

### Étape 1 – Créer les fichiers du projet

Créer un fichier **`index.html`** :
```html
<!DOCTYPE html>
<html>
<body>
 Choisir votre html file 
</body>
</html>
```

Créer un **`Dockerfile`** :
```dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html
EXPOSE 80
```

Créer un **`docker-compose.yml`** :
```yaml
version: '3'
services:
  jenkins:
    image: jenkins/jenkins
    ports:
      - "8080:8080"
  mon-site:
    build: .
    ports:
      - "80:80"
```

---

### Étape 2 – Lancer les conteneurs

```bash
docker-compose up -d
```

Vérifier que les conteneurs tournent :
```bash
docker ps
```

---

### Étape 3 – Pousser le code vers GitHub

```bash
git init
git add *
git config --global user.email "votre@email.com"
git commit -m "tp5 v1"
git remote add origin https://github.com/VOTRE_USERNAME/VOTRE_REPO.git
git push origin main
```

---

### Étape 4 – Créer le Job Jenkins Freestyle

1. Aller sur Jenkins → **Nouveau Item**
2. Nom : `tp5jobproject` | Type : **Freestyle**
3. **Gestion de code source** → Git → URL de votre repo GitHub
4. **Ce qui déclenche le build** → Scrutation SCM → `* * * * *`
5. **Build** → Ajouter **Docker Build and Publish**
   - Repository Name : `VOTRE_USERNAME/tp5proj`
   - Registry credentials : vos identifiants Docker Hub
6. Cliquer **Sauver** → **Build Now**

---

## 🚀 Partie 2 – CI/CD (Intégration + Déploiement Continus)

### Étape 1 – Job Freestyle avec déploiement

1. Nouveau job : `tp5job2project` | Type : **Freestyle**
2. Même configuration GitHub que précédemment
3. **Build** → **Docker Build and Publish**
   - Repository Name : `VOTRE_USERNAME/tp5projv2`
4. **Build** → **Execute shell** :

```bash
docker stop tp5_container || true
docker rm tp5_container || true
docker run -d --name tp5_container -p 80:80 VOTRE_USERNAME/tp5projv2
```

5. Cliquer **Sauver** → **Build Now**
6. Tester sur : `http://localhost:80`

---

### Étape 2 – Job Pipeline inline (3 stages)

1. Nouveau job : `tp5job3pipeline` | Type : **Pipeline**
2. Coller ce script dans la section **Pipeline** :

```groovy
pipeline {
    environment {
        registry = "VOTRE_USERNAME/tp5projv2"
        registryCredential = 'dockerhub'
        dockerImage = ''
    }
    agent any
    stages {
        stage('Cloning Git') {
            steps {
                git branch: 'main',
                url: 'https://github.com/VOTRE_USERNAME/VOTRE_REPO'
            }
        }
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Publish Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
    }
}
```

3. Cliquer **Sauver** → **Build Now**

---

### Étape 3 – Job Pipeline avec Jenkinsfile (4 stages)

Créer un fichier **`Jenkinsfile`** dans votre projet :

```groovy
pipeline {
    environment {
        registry = "VOTRE_USERNAME/tp5projv2"
        registryCredential = 'dockerhub'
        dockerImage = ''
    }
    agent any
    stages {
        stage('Cloning Git') {
            steps {
                git branch: 'main',
                url: 'https://github.com/VOTRE_USERNAME/VOTRE_REPO'
            }
        }
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Test image') {
            steps {
                script {
                    echo "Tests passed"
                }
            }
        }
        stage('Publish Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
    }
}
```

Pousser vers GitHub :
```bash
git add Jenkinsfile
git commit -m "add Jenkinsfile"
git push origin main
```

Configurer le job Jenkins :
1. Nouveau job : `tp5job4pipeline` | Type : **Pipeline**
2. **Pipeline** → Definition : **Pipeline script from SCM**
3. SCM : **Git** → URL de votre repo
4. Branch : `*/main`
5. Script Path : `Jenkinsfile`
6. Cliquer **Sauver** → **Build Now**

---

### Étape 4 – Ajouter le stage Deploy (5 stages)

Ajouter ce stage dans votre **`Jenkinsfile`** :

```groovy
stage('Deploy image') {
    steps {
        sh "docker stop tp5_container || true"
        sh "docker rm tp5_container || true"
        sh "docker run -d --name tp5_container -p 80:80 VOTRE_USERNAME/tp5projv2:$BUILD_NUMBER"
    }
}
```

Pousser les changements :
```bash
git add Jenkinsfile
git commit -m "add deploy stage"
git push origin main
```

Jenkins déclenche automatiquement le build → vérifier les **5 stages** dans la Stage View.

---

## 🔁 Flux CI/CD

```
git push
    │
    ▼
Jenkins détecte le changement (polling)
    │
    ▼
Cloning Git → Building image → Test image → Publish Image → Deploy image
                                                  │                │
                                                  ▼                ▼
                                             Docker Hub     http://localhost:80
```

---

## 🔧 Plugins Jenkins à installer

Aller dans **Manage Jenkins** → **Plugins** → **Available** et installer :

| Plugin | Utilité |
|--------|---------|
| `Docker Build and Publish` | Build + Push image |
| `Docker Commons` | Fonctions partagées Docker |
| `Docker Pipeline` | Docker dans pipeline Groovy |
| `Pipeline Stage View` | Visualiser les stages |

---

## 📌 Récapitulatif des Jobs

| Job | Type | Stages | Rôle |
|-----|------|--------|------|
| `tp5jobproject` | Freestyle | - | Build + Push Docker Hub |
| `tp5job2project` | Freestyle | - | Build + Push + Deploy |
| `tp5job3pipeline` | Pipeline | 3 | Clone → Build → Publish |
| `tp5job4pipeline` | Pipeline + Jenkinsfile | 5 | Clone → Build → Test → Publish → Deploy |