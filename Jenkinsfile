pipeline {
    agent { label 'build' }
    parameters {
        string(name: 'APP_NAME', defaultValue: 'myapp')
    }
    environment {
        IMAGE = "${params.APP_NAME}:${BUILD_NUMBER}"
    }
    tools {
        maven 'mvn'
    }
    stages {
        stage('stage I: Git Checkout') {
            steps {
                echo "Git checkout"
                git url: 'https://github.com/iam-dharani/spring-petclinic.git', branch: 'main'
            }
        }
        stage('stage II: build') {
            steps {
                echo "Buiding the package"
                sh 'mvn clean package -Dcheckstyle.skip=true'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', followSymlinks: false
                }
            }
        }
        stage('stage III: code coverage') {
            steps {
                sh 'mvn jacoco:report'
            }
        }
        stage('stage IV: SCA and SAST') {
            parallel {
                stage('OWASP') {
                    steps {
                        echo "OWASP vulnerability check"
                        withCredentials([string(credentialsId: 'nvd-key', variable: 'NVD_KEY')]) {
                        dependencyCheck additionalArguments: """
                        --scan .
                        --data /var/owasp-data
                        --nvdApiKey $NVD_KEY
                        --format ALL""", 
                        odcInstallation: 'owasp-dc'
                        }
                    }
                }
                stage('SAST') {
                    steps {
                        echo "SAST using sonar scanner"
                        withSonarQubeEnv('sonarqube') {
                            script {
                                def sonarHome = tool 'sonar-scanner'
                                sh """
                                ${sonarHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=${params.APP_NAME} \
                                -Dsonar.projectName=${params.APP_NAME} \
                                -Dsonar.java.binaries=target """
                            }
                        }
                    }
                }
            }
        }
        stage('stage V: QualityGate') {
            steps {
                echo "Quality Gate Check"
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Quality Gate check failed hence aborting the pipeline ${qg.status}"
                    }
                    else {
                        echo "Quality Gate check success ${qg.status}"
                    }
                }
            }
        }
        stage('stage VI: Image build and push') {
            steps {
                echo "create docker image and push to docker hub"
                script {
                    def image = docker.build("${env.IMAGE}")
                    docker.withRegistry('https://registry.hub.docker.com','docker-cred'){
                    image.push()
                    }
                }
            }
        }
        stage('stage VII: trivy image scan') {
            steps {
                sh """trivy image \
                --severity HIGH,CRITICAL \
                --exit-code 1 \
                 ${env.IMAGE}"""
            }
        }
        stage('stage VIII: smokerun') {
            steps {
                sh "docker run -d -p 8081:8080 --name smokerun ${env.IMAGE}"
                sh "sleep 20; docker rm --force smokerun"
            }
        }
    }
}
