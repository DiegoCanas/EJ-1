String lastTag() {
    sh('git fetch --tags')
    return sh(script: 'git describe --tags --abbrev=0', returnStdout: true).trim()
}

String calculateNextTag() {
    def lastTagValue = lastTag()

    if (lastTagValue == null) {
        error("No se encontraron etiquetas en el repositorio.")
        return null
    }

    def tagParts = lastTagValue.tokenize('.')
    def x = tagParts[0] as int
    def y = tagParts[1] as int
    def z = tagParts[2] as int

    if (isFeature()) {
        y++
    } else if (isBreak()) {
        x++
        y = 0
        z = 0
    } else if (isFix()) {
        z++
    } else if (isMaster()) {
        x++
        y = 0
        z = 0
    } else {
        echo "No se reconoce la rama actual."
    }

    echo "Versión calculada ${x}.${y}.${z}"
    return "${x}.${y}.${z}"
}

void createTag(String nextTag) {
    echo "Siguiente versión calculada: ${nextTag}"
    sh "git tag ${nextTag}"
    sh "git push origin ${nextTag}"
}

boolean isFeature() {
    return env.BRANCH_NAME.startsWith('feature/')
}

boolean isBreak() {
    return env.BRANCH_NAME == 'break'
}

boolean isFix() {
    return env.BRANCH_NAME.startsWith('fix/')
}

boolean isMaster() {
    return env.BRANCH_NAME == 'master'
}

pipeline {
    environment {
        registry = "diegocanas/server"
        registryCredential = 'dockerhub'
        dockerImage = ''
        dockerT = 'docker'
        TAG_TO_CHECK = ''
    }
    agent any

    tools {
        nodejs "node"
        dockerTool dockerT
    }

    stages {
        stage('🏁Checkout') {
            steps {
                script {
                    checkout([$class: 'GitSCM', 
                        branches: [[name: '*/main']], 
                        userRemoteConfigs: [[credentialsId: 'MD-TOKEN', url: 'https://github.com/DiegoCanas/EJ-1']]])
                        // AÑADIR EL TOKEN
                }
            }
        }

        stage('⬇️Instalacion de dependencias') {
            steps {
                script {
                    sh 'node -v'
                    sh 'npm -v'
                    sh 'npm install'
                }                
            }
        }

        stage('🥽Linteo') {
            steps {
                echo("Linting...")
            }
        }

        stage('🧪Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('🔨Build') {
            steps {
                script {
                    echo 'hols'
                }
            }
        }

        stage('🪄Creación de la imagen docker') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
            }
        }

        stage('⬆️Subida de la imagen al registry') {
            steps {
                script {
                    TAG_TO_CHECK = calculateNextTag()
                    createTag(TAG_TO_CHECK)
                    docker.withRegistry('', registryCredential) {
                        dockerImage.tag(TAG_TO_CHECK)
                        dockerImage.push()
                    }
                }
            }
        }

        stage('✈️Despliegue') {
            steps {
                script {
                    kubeconfig(credentialsId: 'kubeconfig') {
                        sh "sed -i 's|image: diegocanas/server:.*|image: diegocanas/server:${TAG_TO_CHECK}|' deployment.yaml"
                        sh 'kubectl apply -f service.yaml --namespace=namespace-server'
                        sh 'kubectl apply -f deployment.yaml --namespace=namespace-server'
                        sh 'kubectl get pods --namespace=namespace-server'
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

