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
                    script {
                        def scannerHome = tool SONAR_SCANNER_TOOL
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                             -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                             -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                             -Dsonar.sources=src \
                             -Dsonar.java.binaries=target
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDENTIALS_ID,
                    usernameVariable: 'REG_USER',
                    passwordVariable: 'REG_PASS'
                )]) {
                    sh """
                        echo "$REG_PASS" | docker login ${REGISTRY_URL} -u "$REG_USER" --password-stdin
                        docker push ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout ${REGISTRY_URL}
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f || true'
        }
    }
}
