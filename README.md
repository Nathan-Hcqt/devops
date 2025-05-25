## Introduction

DoodleStudent est une application collaborative, pensée pour simplifier l’organisation d’événements et la gestion de sondages au sein de groupes. 
Cette application a été prise comme base intitiale pour notre projet “Software Bots in Software Engineering”, terme désignant un agent logiciel autonome, capable d’automatiser des tâches répétitives, d’interagir avec les utilisateurs et d’améliorer l’efficacité des processus collaboratifs.


## L’architecture technique de DoodleStudent

## Backend
Développé avec Quarkus (Java), il gère la logique métier, l’accès aux données (MySQL), l’authentification, la gestion des sondages, des utilisateurs, des préférences et des commentaires.
## Frontend 
Construit avec Angular, il offre une interface utilisateur moderne, réactive et accessible, permettant à chaque membre du groupe de participer facilement aux sondages et discussions.
Services complémentaires : intégration d’Etherpad pour la coédition de texte, et d’un serveur mail pour les notifications automatiques.

## GitHub Actions 
Permet d’automatiser l’ensemble du cycle de vie du développement logiciel :
À chaque modification du code (push ou pull request), des workflows sont déclenchés instantanément pour builder, tester et valider le projet.
Ces workflows assurent que chaque contribution respecte les standards de qualité, que les tests passent, et que le projet reste stable et fonctionnel.
Les résultats des builds et des tests sont visibles en temps réel dans l’onglet "Actions" du dépôt GitHub, permettant ainsi une meilleure collaboration, la transparence et la réactivité de l’équipe.
L’intégration de GitHub Actions apporte de nombreux bénéfices :

## Automatisation 
Plus besoin de lancer manuellement les tests ou les builds, tout est fait automatiquement à chaque modification.
Qualité : chaque commit est validé par une série de tests, ce qui limite les régressions et garantit la robustesse du projet.
Collaboration : chaque membre de l’équipe est informé en temps réel de l’état du projet, ce qui facilite la gestion des contributions et la résolution rapide des problèmes.






L’objectif de GitHub Actions est donc de réaliser des tests, la construction d’un projet, son déploiement ou tout autre script que l’on veut exécuter lors d’actions précises comme un push ou un pull request.

## Structure du projet

Afin de mettre en place GitHub Actions nous devons créer un dossier .github/workflows/ à la racine du projet. Dans ce dossier nous stockerons tous les fichiers de workflows en .yml ou .yaml. Nous avons utilisé dans le projet deux fichiers : ci-back.yml et ci-front.yml qui sont voués à tester le back Quarkus et le front Angular du projet Doodle fournit.

## Le fichier ci-back.yml 


Dans ce fichier nous avons tout d’abord l’en-tête avec le nom du workflow suivit par le section on qui déclenche le workflow selon les conditions voulue. Ici il est déclenché lorsqu'un push est effectué avec un modification  dans le dossier api du backend ou une modification de ce même fichier. Il est aussi déclenché lors d’une pull request ciblant la branche main. La balise workflow_dispatch: permet de le lancer manuellement depuis GitHub

```yml
 name: Backend CI/CD - Quarkus

 on:
  push:
    branches: [ main ]
    paths:
      - 'api/**'
      - '.github/workflows/ci-back.yml'
  pull_request:
    branches: [ main ]
    paths:
      - 'api/**'
  workflow_dispatch:
```

Ensuite la Balise env: qui configure les variables d’environnements, ici on utilise java, maven et docker.

```yml
 env:
  JAVA_VERSION: '17'
  MAVEN_OPTS: '-Xmx1024m'
  DOCKER: 'docker.io'
  DOCKER_COMPOSE: 'docker-compose'
```


Puis on a la balise jobs: qui va indiquer toutes les tâches à faire lors du déclenchement du workflow. On précise ici le nom et la version de la VM Ubuntu GitHub-hostée qui effectue le script. On précise aussi le dossier ou le workflow doit opérer.
```yml
 jobs:
  test:
    name: Tests Backend
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./doodlestudent-main/api
```

Nous voici maintenant aux étapes du job. Les étapes sont annoncées dans le fichier par une balise name: et uses: qui définissent leurs noms et leurs actions.

```yml
 steps:


    # Checkout code
      - name: Checkout code
        uses: actions/checkout@v4


    # Set up Java environment
      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'
          cache: 'maven'
     
    # Install Maven
      - name: Install Maven
        run: mvn --version
   
      - name: Rendre mvnw exécutable
        run: chmod +x ./mvnw


    # Install Docker
      - name: Install Docker
        uses: docker/setup-buildx-action@v2
        with:
          version: 'latest'
   
    # Running the application in dev mode
      - name : Running the application in dev mode
        run: docker compose up --detach
   
    # Wait the correct start of the application
      - name: Wait for the application to start
        run: |
          echo "Waiting for the application to start..."
          sleep 10 # Adjust the sleep time as necessary
          echo "Application started successfully."
   
 
    # Compile the project
      - name: Compile the project
        run: ./mvnw clean package
   
    # Running unit tests
      - name: Running unit tests
        run: ./mvnw test
   
    # Stop the application
      - name: Stop the application
        run: docker compose down
```

Les étapes ici sont : 
Checkout code : Récupère le code source du projet 
Set up Java environment  
Install Maven
Install Docker 
Running the application in dev mode : Lance l’application backend Quarkus
Wait the correct start of the application : Permet d’attendre que l’application est bien lancée
Compile the project 
Running unit tests 
Stop the application

Avec ce fichier workflow on va donc initialiser l’application backend sur une machine Ubuntu à chaque push ou pull request et effectuer des tests unitaires. Cela permet de vérifier le bon fonctionnement du backend après chaque modification.

## Le fichier ci-front.yml 
```yml
 name: CI Front Angular


 on:
  push:
    branches: [ main ]
    paths:
      - 'front/**'
      - '.github/workflows/ci-front.yml'
  pull_request:
    branches: [ main ]
    paths:
      - 'front/**'
  workflow_dispatch:


 jobs:
  test:
    name: Tests Front Angular
    runs-on: ubuntu-latest


    defaults:
          run:
            working-directory: ./doodlestudent-main/front


    steps:


    # Checkout code
      - name: Checkout code
        uses: actions/checkout@v4


    # Set up Node.js environment
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'


    # Install dependencies
      - name: Install dependencies
        run: npm ci
       
    # Build the project
      - name: Build the project
        run: npx ng build
     
    # Running unit tests
      - name: Running unit tests
        run : npx ng test --watch=false --browsers=ChromeHeadless
```

Le fichier pour le front est sensiblement le même mais adapté au déploiement de l’application front.


## Résultats
Maintenant que nos fichiers workflow sont créés, nous pouvons nous rendre dans GitHub dans la section actions de notre projet pour observer nos résultats.

Dans cette section on peut observer que les fichiers workflow se déclenchent bien lors d’un push. On remarque aussi directement le résultat des tests.

On peut par la suite accéder aux détails des tests. Pour le backend on peut voir que toutes les étapes du workflow ont été réussies. Ce qui indique donc la qualité des dernières modifications.




Pour la partie du front on a donc une erreur lors de la lecture du fichier.


On accède donc aux détails pour vérifier dans quelle section l'erreur a lieu. Cela peut être une erreur dans l’écriture du workflow ou lors d’une installation ou encore un erreur lors des tests.

On remarque donc que l’erreur est due aux tests unitaires. Ici certaines propriétés sont absentes et donc on peut conclure qu’un problème lors de la dernière mise à jour est arrivé.
