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
                    sh 'node -v'
                    sh 'npm -v'
                    sh 'npm install'
                }                
            }

        }

        stage ('🥽Linteo'){
            steps{
                echo("Linting...")
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
                    echo ('hola')
                    sh ("docker image build -f ./dockerfile -t service:latest .")
                }
                
            }

        }

        stage ('Creación de la imagen docker'){
            steps{
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }

        stage ('Subida de la imagen al registry'){
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                    dockerImage.push()
                    }
                }
            }
        }
    }
}