pipeline {
    agent { label 'dind-agent' }   // your Jenkins agent with Docker + Maven

    environment {
        // ---- SonarQube ----
        SONARQUBE_SERVER     = 'SonarQube'      // matches Jenkins configuration
        SONAR_PROJECT_KEY    = 'java-app'
        SONAR_PROJECT_NAME   = 'java-app'
        SONAR_SCANNER_TOOL   = 'SonarScanner'

        // ---- Docker / OCI ----
        REGISTRY_URL         = 'docker.io'      // or iad.ocir.io
        IMAGE_NAME           = 'youruser/java-app'
        IMAGE_TAG            = "v${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'docker-registry-creds'

        // ---- Maven ----
        MAVEN_OPTS = "-Dmaven.test.failure.ignore=true"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test with Maven') {
            steps {
                sh """
                    mvn clean package -DskipTests=false
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
