pipeline {
    agent any 
    parameters {
        string(name: 'ejecutor', defaultValue: 'defaultValue', description: 'Nombre de la persona que ejecuta la pipeline')
        string(name: 'motivo', defaultValue: 'defaultValue', description: 'Motivo por el cual estamos ejecutando la pipeline')
        string(name: 'correo_notificación', defaultValue: 'defaultValue', description: 'Correo al que notificaremos el resultado de cada stage ejecutado')
    }
    /* environment {
        CORREO="jubelltols@outlook.com"
    } */
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
            post {
                success {
                    script {
                        env.LINTER = "SUCCESS"
                    }
                }
                failure {
                    script {
                        env.LINTER = "FAILURE"
                    }
                }
            }
        }
        stage('test') {
            steps {
                script {
                    sh "npm install"
                    sh "npm run build"
                    sh "npm run start &"
                    sh "npm run cypress http://localhost:3000"
                }
            }
            post {
                success {
                    script {
                        env.TEST = "SUCCESS"
                    }
                }
                failure {
                    script {
                        env.TEST = "FAILURE"
                    }
                }
            }
        }
       /*  stage('update_readme') {
            steps {
                script {
                    sh "node jenkinsScripts/update_readme.js"
                }
            }
            post {
                success {
                    script {
                        env.UPDATE = "SUCCESS"
                    }
                }
                failure {
                    script {
                        env.UPDATE = "FAILURE"
                    }
                }
            }
        }
        stage('push_Changes') {
            steps {
                script {
                    sh ""
                }
            }
            post {
                success {
                    script {
                        env.PUSH = "SUCCESS"
                    }
                }
                failure {
                    script {
                        env.PUSH = "FAILURE"
                    }
                }
            }
        }
        stage('deploy_to_Vercel') {
            steps {
                script {
                    sh ""
                }
            }
            post {
                success {
                    script {
                        env.DEPLOY = "SUCCESS"
                    }
                }
                failure {
                    script {
                        env.DEPLOY = "FAILURE"
                    }
                }
            }
        }*/
        stage('notificacion') {
            steps {
                script {
                    sh "node jenkinsScripts/notificacion.js"
                }
            }
        } 
    }
}
