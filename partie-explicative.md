# Explications relatives au projet "Gérez un projet collaboratif en intégrant une démarche CI/CD"

URL SonarCloud : https://sonarcloud.io/organizations/francoismt/projects

URL Docker Hub front : https://hub.docker.com/repository/docker/francoismarc/bobapp-front/general

URL Docker Hub back : https://hub.docker.com/repository/docker/francoismarc/bobapp-back/general

## 1. Les étapes des GitHub Actions

### Backend CI/CD

#### Fichier: `.github/workflows/backend_cicd.yml`

**Déclencheurs**:

**Jobs**:

 1. Tests :  **backend_test**

- **Objectif**: Tester le backend, vérifier la couverture de code et analyser la qualité du code avec SonarCloud.

   **Étapes**:
   
 2. Déploiement : **docker**
- **Objectif**: Déployer l'image backend sur Docker Hub.

### Frontend CI/CD

#### Fichier: `.github/workflows/frontend_cicd.yml`

**Déclencheurs**:

**Jobs**:

 1. Tests :  **frontend_test**
- **Objectif**: Tester le backend, vérifier la couverture de code et analyser la qualité du code avec SonarCloud.

  **Étapes**:
   
 2. Déploiement : **docker**
- **Objectif**: Déployer l'image front sur Docker Hub.

## 2. Key Performance Indicators (KPIs)

## 3. L’analyse des métriques 

## 4. Retours utilisateurs
