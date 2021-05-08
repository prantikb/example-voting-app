pipeline {
    
    agent none   

    stages {    
        stage("worker-build") {

            when{
                changeset "**/worker/**"
            }

            agent {
                docker {
                    image "maven:3.6.1-jdk-8-slim"
                }
            } 
            
            steps {
                echo "building Worker App"
                dir("worker") {
                    sh "mvn compile"
                }                        
            }
        }
        stage("worker-test") {

            when{
                changeset "**/worker/**"
            }

            agent {
                docker {
                    image "maven:3.6.1-jdk-8-slim"
                }
            } 
            
            steps {
                echo "Running unit tests on worker app"
                dir("worker") {
                    sh "mvn clean test"
                }
            }
        }
        stage("worker-package") {

            when{
                branch "master"
                changeset "**/worker/**"
            }

            agent {
                docker {
                    image "maven:3.6.1-jdk-8-slim"
                }
            } 
            
            steps {
                echo "Packaging Worker App into a jarfile"
                dir("worker") {
                    sh "mvn package -DskipTests"
                    archiveArtifacts artifacts: "**/target/*.jar", fingerprint: true
                }
            }
        }  
        stage("worker-docker-package") {
            agent any
            when {
                changeset "**/worker/**"
                branch "master"
            }
            steps {
                echo "Packaging worker app with Docker"
                script {
                    docker.withRegistry("https://index.docker.io/v1/","dockerlogin") {
                        def workerImage = docker.build("prantik/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("latest")
                    }
                }
            }
        }  
    

    
        stage("vote-build") {
            when {
                changeset "**/vote/**"
            }
            agent {
                docker {
                    image "python:2.7.16-slim"
                    args "--user root"
                }
            }
            steps {
                echo "Compiling vote App"
                dir("vote") {
                    sh "pip install -r requirements.txt"
                }                        
            }
        }
        stage("vote-test") {
            when {
                changeset "**/vote/**"
            }
            agent {
                docker {
                    image "python:2.7.16-slim"
                    args "--user root"
                }
            }
            steps {
                echo "Running unit tests on vote app"
                dir("vote") {
                    sh "pip install -r requirements.txt"
                    sh "nosetests -v"
                }
            }
        }
        stage("vote-docker-package") {
            agent any
            when {
                changeset "**/vote/**"
                branch "master"
            }
            steps {
                echo "Packaging vote app with Docker"
                script {
                    docker.withRegistry("https://index.docker.io/v1/","dockerlogin") {
                        def workerImage = docker.build("prantik/vote:v${env.BUILD_ID}", "./vote")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }        


   
        stage("result-build") {
            when {
                changeset "**/result/**"
            }
            agent {
                docker {
                    image "node:8.16.0-alpine"
                }
            }
            steps {
                echo "Compiling result App"
                dir("result") {
                    sh "npm install"
                }                        
            }
        }
        stage("result-test") {
            when {
                changeset "**/result/**"
            }
            agent {
                docker {
                    image "node:8.16.0-alpine"
                }
            }
            steps {
                echo "Running unit tests on result app"
                dir("result") {
                    sh "npm install"
                    sh "npm test"
                }
            }
        }  
        stage("result-docker-package") {
            agent any
            when {
                changeset "**/result/**"
                branch "master"
            }
            steps {
                echo "Packaging result app with Docker"
                script {
                    docker.withRegistry("https://index.docker.io/v1/","dockerlogin") {
                        def workerImage = docker.build("prantik/result:v${env.BUILD_ID}", "./result")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }           
    }

    post {
        always {            
            echo "the job is complete."
        }
    }
}