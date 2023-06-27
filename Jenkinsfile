pipeline {
    agent any

    environment {
        NAME_IMG = 'react-test-app'
        NAME_CTN = 'react-test-container'
        APP_PORT: '3003'
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    stages {
        stage('Checkout Git') {
            steps {
                checkout scm
            }
        }
        stage('Inicializar') {
            steps {
                sh 'node -v && rm -Rf node_modules/ && rm -Rf build/'
            }
        }
        stage('Obtener dependencias') {
            steps {
                sh 'npm install'
            }
        }
        stage('Compilar') {
            steps {
                sh 'npm run build'
            }
        }
        stage('Dockerize') {
            steps {
                // Construir la imagen de Docker
                sh 'docker build -t $NAME_IMG .'
            }
        }
        stage('Desplegar') {
            steps {
                // Detener y eliminar el contenedor existente (si existe)
                sh 'docker stop $NAME_CTN || true'
                sh 'docker rm $NAME_CTN || true'

                // Iniciar un nuevo contenedor con la imagen Docker
                sh 'docker run -d --name $NAME_CTN -p $APP_PORT:3000 $NAME_IMG'
            }
        }
    }

    // Notificar en caso de errores
    post {
        always {
            // Archivar los archivos de la aplicación React construida
            archiveArtifacts(artifacts: 'build/**', fingerprint: true)

            // Enviar notificación por correo electrónico
            emailext (
                subject: 'Estado de compilación: ${currentBuild.currentResult}',
                body: """<p>Detalles de la compilación:</p>
                         <p>Estado: ${currentBuild.currentResult}</p>
                         <p>URL de la compilación: ${env.BUILD_URL}</p>""",
                recipientProviders: [developers()],
            )
        }
    }
}
