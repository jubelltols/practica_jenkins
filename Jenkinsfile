pipeline {
    agent any 
    parameters {
        string(name: 'ejecutor', defaultValue: 'jubelltols', description: 'Nombre de la persona que ejecuta la pipeline')
        string(name: 'motivo', defaultValue: 'Prueba praactica jenkins', description: 'Motivo por el cual estamos ejecutando la pipeline')
        string(name: 'correo_notificación', defaultValue: 'jubelltols@gmail.com', description: 'Correo al que notificaremos el resultado de cada stage ejecutado')
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
                    env.CYPRESS = sh(script: "npm run cypress", returnStatus:true)
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
        stage('update_readme') {
            steps {
                script {
                    echo "${env.CYPRESS}"
                    sh "node jenkinsScripts/update_readme.js $CYPRESS"
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
                    sh "git config user.name jubelltols"
                    sh "git config user.email jubelltols@gmail.com"
                    sh "git add ."
                    sh "git commit -m 'Pipeline ejecutada por ${params.ejecutor}. Motivo: ${params.motivo}'"
                    withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'USER', passwordVariable: 'PASSWORD')]) {
                        sh 'git remote set-url origin https://"$USER":"$PASSWORD"@github.com/jubelltols/practica_jenkins'
                    }
                    sh "git push origin HEAD:master"
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
                    echo "${env.UPDATE}"
                    echo "${env.DEPLOY}"
                    env.CORREO = ${params.correo_notificación}
                    sh "node jenkinsScripts/notificacion.js"
                }
            }
        } 
    }
}
