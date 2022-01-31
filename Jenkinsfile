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
                    env.TEST = sh {'''npm run build
                        npm run start &
                        npm run cypress'''}
                }
            }
            /* post {
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
            } */
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
        }*/
        stage('deploy_to_Vercel') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'vercel-org-id', variable: 'VERCELORGID'),
                        string(credentialsId: 'vercel-project-id', variable: 'VERCELPROJECTID'),
                        string(credentialsId: 'vercel-token', variable: 'VERCELTOKEN')
                    ]){
                        sh 'VERCEL_ORG_ID="$VERCELORGID" VERCEL_PROJECT_ID="$VERCELPROJECTID" vercel --prod --scope jubelltols --token="$VERCELTOKEN"'
                    }
                    
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
        }
        stage('notificacion') {
            steps {
                script {
                    echo "${env.LINTER}"
                    echo "${env.TEST}"
                    echo "${env.DEPLOY}"
                    /* sh "node jenkinsScripts/notificacion.js" */
                }
            }
        } 
    }
}
