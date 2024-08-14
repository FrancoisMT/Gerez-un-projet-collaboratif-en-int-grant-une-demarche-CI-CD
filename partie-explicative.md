# Explications relatives au projet "Gérez un projet collaboratif en intégrant une démarche CI/CD"

URL SonarCloud : https://sonarcloud.io/organizations/francoismt/projects

URL Docker Hub front : https://hub.docker.com/repository/docker/francoismarc/bobapp-front/general

URL Docker Hub back : https://hub.docker.com/repository/docker/francoismarc/bobapp-back/general

## 1. Les étapes des GitHub Actions

### Backend CI/CD

#### Fichier: `.github/workflows/backend_cicd.yml`

**Déclencheurs**:

Ce workflow est déclenché par les événements suivants :

- **Push**:
  - **Chemins surveillés**: `back/**`, `.github/workflows/**`
  - **Branches**: `main`
  - **Description**: Le workflow est exécuté lorsqu'il y a un push dans les chemins spécifiés, ce qui inclut les modifications apportées au code du backend ou aux fichiers de workflow GitHub.

- **Pull Request**:
  - **Chemins surveillés**: `back/**`, `.github/workflows/**`
  - **Branches**: `main`
  - **Types**: `opened`, `synchronize`, `reopened`
  - **Description**: Le workflow est exécuté lorsque une pull request est ouverte, synchronisée (mise à jour avec de nouveaux commits), ou rouverte, si les changements affectent les chemins spécifiés.

**Jobs**:

 1. **Tests : `backend_test`**

   - **Objectif**: Tester le backend, vérifier la couverture de code et analyser la qualité du code avec SonarCloud.
   
   - **Étapes**:
     - **Checkout**: Récupére le code source.
       ```yaml
       - name: Checkout
         uses: actions/checkout@v4
       ```
     - **Set Up JDK 17**: Installe JDK 17 nécessaire pour la compilation Java.
       ```yaml
       - name: Set Up JDK 17
         uses: actions/setup-java@v4
         with:
           java-version: '17'
           distribution: 'adopt'
       ```
     - **Cache Maven Packages**: Cache les dépendances Maven pour accélérer les builds futurs.
       ```yaml
       - name: Cache Maven Packages
         uses: actions/cache@v4
         with:
           path: ~/.m2
           key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
           restore-keys: |
             ${{ runner.os }}-m2
       ```
     - **Cache Build Output**: Cache les fichiers de sortie du build pour optimiser les builds futurs.
       ```yaml
       - name: Cache Build Output
         uses: actions/cache@v4
         with:
           path: target
           key: ${{ runner.os }}-build-${{ hashFiles('**/pom.xml') }}
           restore-keys: |
             ${{ runner.os }}-build-
       ```
     - **Build and Test with Maven**: Compile le projet et exécuter les tests avec Maven.
       ```yaml
       - name: Build and Test with Maven
         run: mvn -B clean verify
       ```
     - **Upload Jacoco Report**: Télécharge le rapport de couverture de code Jacoco.
       ```yaml
       - name: Upload Jacoco Report
         uses: actions/upload-artifact@v4
         with:
           name: jacoco-report
           path: back/target/site/jacoco/
           overwrite: true
           if-no-files-found: error
       ```
     - **Check sonar-project.properties**: Vérifie le contenu du fichier de configuration SonarCloud.
       ```yaml
       - name: Check sonar-project.properties
         run: cat sonar-project.properties
       ```
     - **SonarCloud Scan**: Exécute l'analyse SonarCloud pour évaluer la qualité du code.
       ```yaml
       - name: SonarCloud Scan
         env:
           SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
         run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:4.0.0.4121:sonar
       ```
   
2. **Déploiement : `docker`**

   - **Objectif**: Déployer l'image Docker du backend sur Docker Hub.
   
   - **Étapes**:
     - **Checkout**: Clone le dépôt GitHub pour avoir accès au code source nécessaire pour construire l'image Docker.
       ```yaml
       - name: Checkout
         uses: actions/checkout@v4
       ```
     - **Set Up Docker Buildx**: Configure Docker Buildx, un outil permettant de construire des images Docker multiplateformes.
       ```yaml
       - name: Set Up Docker Buildx
         uses: docker/setup-buildx-action@v3.3.0
       ```
     - **Login to Docker Hub**: Se connecte à Docker Hub en utilisant les informations d'identification stockées en tant que secrets GitHub.
       ```yaml
       - name: Login to Docker Hub
         uses: docker/login-action@v3.2.0
         with:
           username: ${{ secrets.DOCKER_USERNAME }}
           password: ${{ secrets.DOCKER_PASSWORD }}
       ```
     - **Build and Push Backend Docker Image**: Compile et déploie l'image du projet sur Docker Hub. 
       ```yaml
       - name: Build and Push Backend Docker Image
         uses: docker/build-push-action@v6.2.0
         with:
           context: ./back
           file: ./back/Dockerfile
           push: true
           tags: ${{ secrets.DOCKER_USERNAME }}/bobapp-back:latest
       ```

### Frontend CI/CD

#### Fichier: `.github/workflows/frontend_cicd.yml`

**Déclencheurs**:

Ce workflow est déclenché par les événements suivants :

- **Push**:
  - **Chemins surveillés**: `front/**`, `.github/workflows/**`
  - **Branches**: `main`
  - **Description**: Le workflow est exécuté lorsqu'il y a un push dans les chemins spécifiés, ce qui inclut les modifications apportées au code du frontend ou aux fichiers de workflow GitHub.

- **Pull Request**:
  - **Chemins surveillés**: `front/**`, `.github/workflows/**`
  - **Branches**: `main`
  - **Types**: `opened`, `synchronize`, `reopened`
  - **Description**: Le workflow est exécuté lorsque une pull request est ouverte, synchronisée (mise à jour avec de nouveaux commits), ou rouverte, si les changements affectent les chemins spécifiés.

**Jobs**:

1. **Tests : `frontend_test`**

   - **Objectif**: Tester le frontend, vérifier la couverture de code et analyser la qualité du code avec SonarCloud.
   
   - **Étapes**:
     - **Checkout**: Récupère le code source.
       ```yaml
       - name: Checkout
         uses: actions/checkout@v4
       ```
     - **Use Node.js**: Configure la version de Node.js nécessaire pour le projet.
       ```yaml
       - name: Use Node.js ${{ matrix.node-version }}
         uses: actions/setup-node@v4
         with:
           node-version: ${{ matrix.node-version }}
       ```
     - **Cache Node Modules**: Cache les modules Node.js pour accélérer les builds futurs.
       ```yaml
       - name: Cache Node Modules
         uses: actions/cache@v4
         with:
           path: ~/.npm
           key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
           restore-keys: |
             ${{ runner.os }}-node-
       ```
     - **Install Dependencies**: Installe les dépendances du projet.
       ```yaml
       - name: Install Dependencies
         run: npm ci
       ```
     - **Build Angular Project**: Compile le projet Angular.
       ```yaml
       - name: Build Angular Project
         run: npm run build
       ```
     - **Run Tests and Generate Code Coverage Report**: Exécute les tests et génére un rapport de couverture de code.
       ```yaml
       - name: Run Tests and Generate Code Coverage Report
         run: npm run test -- --no-watch --no-progress --browsers=ChromeHeadless --code-coverage
       ```
     - **Upload Code Coverage Report**: Télécharge le rapport de couverture de code.
       ```yaml
       - name: Upload Code Coverage Report
         uses: actions/upload-artifact@v4
         with:
           name: front-coverage-report
           path: front/coverage/
           overwrite: true
           if-no-files-found: error
       ```
     - **SonarCloud Scan**: Exécute l'analyse SonarCloud pour évaluer la qualité du code frontend.
       ```yaml
       - name: SonarCloud Scan
         uses: SonarSource/sonarcloud-github-action@master
         with:
           projectBaseDir: front
         env:
           SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
       ```

2. **Déploiement : `docker`**

   - **Objectif**: Déployer l'image Docker du frontend sur Docker Hub.
   
   - **Étapes**:
     - **Checkout**: Clone le dépôt GitHub pour accéder au code source nécessaire pour construire l'image Docker.
       ```yaml
       - name: Checkout
         uses: actions/checkout@v4
       ```
     - **Set Up Docker Buildx**: Configure Docker Buildx pour permettre la construction d'images Docker multiplateformes.
       ```yaml
       - name: Set Up Docker Buildx
         uses: docker/setup-buildx-action@v3.3.0
       ```
     - **Login to Docker Hub**: Se connecte à Docker Hub en utilisant les informations d'identification stockées en tant que secrets GitHub.
       ```yaml
       - name: Login to Docker Hub
         uses: docker/login-action@v3.2.0
         with:
           username: ${{ secrets.DOCKER_USERNAME }}
           password: ${{ secrets.DOCKER_PASSWORD }}
       ```
     - **Build and Push Frontend Docker Image**: Compile et déploie l'image du projet sur Docker Hub.
       ```yaml
       - name: Build and Push Frontend Docker Image
         uses: docker/build-push-action@v6.2.0
         with:
           context: ./front
           file: ./front/Dockerfile
           push: true
           tags: ${{ secrets.DOCKER_USERNAME }}/bobapp-front:latest
       ```

## 2. Key Performance Indicators (KPIs)

## 3. L’analyse des métriques 

## 4. Retours utilisateurs
