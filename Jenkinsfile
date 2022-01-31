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
                    sh "npm install"
                    sh "npm run build"
                    sh "npm run start &"
                    sh "npm run cypress"
                    sh "echo \$?"
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
        } */
        stage('push_Changes') {
            steps {
                script {                    
                        sh "git config user.name jubelltols"
                        sh "git config user.email jubelltols@gmail.com"
                        sh "git add ."
                        sh "git commit -m 'Update README.md'"
                        script.withCredentials([script.usernamePassword(credentialsId: 'github-token', 
                                                usernameVariable: 'USER', 
                                                passwordVariable: 'PASSWORD')]) {
                        withCredentials([usernameColonPassword(credentialsId: 'github-token', variable: 'USERPASS')]) {     
                            sh "git remote set-url origin https://$PASSWORD:$USER@github.com/jubelltols/practica_jenkins"
                        }
                        sh "git push"
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
