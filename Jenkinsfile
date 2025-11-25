pipeline {
    agent { label 'dind-agent' }   // Jenkins agent with Docker + Maven + JDK

    environment {
        // ---- SonarQube ----
        SONARQUBE_SERVER       = 'SonarQube'      // must match Jenkins Global config name
        SONAR_PROJECT_KEY      = 'java-app'
        SONAR_PROJECT_NAME     = 'java-app'
        SONAR_SCANNER_TOOL     = 'SonarScanner'

        // ---- Docker / OCI Registry ----
        REGISTRY_URL           = 'docker.io'      // OR iad.ocir.io/xxxx/namespace
        IMAGE_NAME             = 'youruser/java-app'
        IMAGE_TAG              = "v${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID  = 'docker-registry-creds'

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
                    mvn clean package
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}")_
