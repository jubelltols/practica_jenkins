pipeline {
    agent any 
    parameters {
        string(name: 'ejecutor', defaultValue: 'defaultValue', description: 'Nombre de la persona que ejecuta la pipeline')
        string(name: 'motivo', defaultValue: 'defaultValue', description: 'Motivo por el cual estamos ejecutando la pipeline')
        string(name: 'correo_notificación', defaultValue: 'defaultValue', description: 'Correo al que notificaremos el resultado de cada stage ejecutado')
    }
    triggers {
        pollSCM('0 */3 * * *')
    }
    stages {
        stage('linter') {
            steps {
                script {
                    sh "npm install"
                    sh "npm run lint"
                }
            }
        }
        stage('test') {
            steps {
                script {
                    sh "npm install"
                    sh "npm run start"
                    sh "npm run test"
                }
            }
        }
       /*  stage('update_readme') {
            steps {
                script {
                    sh "node jenkinsScripts/index.js"
                }
            }
        } */
    }
}
