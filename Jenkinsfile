pipeline {
    environment{
        registry = "diegocanas/server"
        registryCredential = 'dockerhub'
        dockerImage = ''
        dockerT = 'docker'
    }
    agent any

    tools {
        nodejs "node"
        dockerTool dockerT
    }

    stages {
        stage ('🏁Checkout'){
            steps{
                script{
                    checkout([$class: 'GitSCM', 
                        branches: [[name: '*/main']], 
                        userRemoteConfigs: [[credentialsId: 'MD-TOKEN', url: 'https://github.com/DiegoCanas/EJ-1']]])
                } 
            }
        }

        stage ('⬇️Instalacion de dependencias'){
            steps{
                script {
                    sh """
                        node -v 
                        npm -v
                        npm install
                        npm i -D lint
                    """
                }                
            }
        }

        stage ('🥽Linteo'){
            steps{
                sh 'npm run lint'
            }
        }

        stage ('🧪Test'){
            steps{
                sh 'npm test'
            }

        }

        stage ('🔨Build'){
            steps{
                script {
                    sh 'npm run build'
                }
                
            }

        }

        stage ('🪄Creación de la imagen docker'){
            steps{
                script {
                    dockerImage = docker.build registry
                }
            }
        }

        stage ('⬆️Subida de la imagen al registry'){
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                    dockerImage.tag("latest")
                    dockerImage.push()
                    }
                }
            }
        }

        stage ('✈️Despliegue'){
            steps{
                script{
                    kubeconfig(credentialsId: 'kubeconfig') {
                    sh """
                        kubectl apply -f service.yaml --namespace=namespace-server
                        kubectl apply -f deployment.yaml --namespace=namespace-server
                        kubectl get pods --namespace=namespace-server
                    """

                    }
                }
            }
        }
    }
    post {
        always{
            cleanWs()
        }
    }
}