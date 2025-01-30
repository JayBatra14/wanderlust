pipeline {
    agent any
    environment{
        SONAR_HOME= tool "Sonar"
        CACHE_DIR = "${WORKSPACE}/npm-cache"
    }
    stages {
        stage("Clone Code from GitHub") {
            steps {
                git url: "https://github.com/JayBatra14/wanderlust.git", branch: "devops"
            }
        }
        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("Sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust -Dsonar.exclusions=**/node_modules/**,**/tests/**,**/logs/**,**/*.png,**/*.jpg,**/*.mp4,**/*.zip"
                }
            }
        }

        stage("Cache Dependencies") {
            steps {
                sh '''
                    if [ -d "$CACHE_DIR/node_modules" ]; then
                        cp -r $CACHE_DIR/node_modules .
                    fi
                '''
            }
        }

        stage("Install Dependencies") {
            steps {
                sh '''
                    rm -rf $CACHE_DIR/node_modules
                    npm ci
                    cp -r node_modules $CACHE_DIR/
                '''
            }
        }
        stage("Owasp dependency check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }/*
        stage("Sonar Quality Gate Scan"){
            steps{
                timeout(time: 10, unit: "MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }*/
        stage("Trivy File System Scan"){
            steps{
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage("Deploy using docker compose"){
            steps{
                sh "docker-compose up -d"
            }
        }
    }
}
