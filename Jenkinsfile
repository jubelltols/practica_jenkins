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
                    env.LINTER = sh(script: "npm run lint", returnStatus:true)
                }
            }
        }
        stage('test') {
            steps {
                script {
                    sh "npm install"
                    sh "npm run build"
                    sh "npm run start &"
                    env.TEST = sh(script: "npm run cypress", returnStatus:true)
                }
            }
        }
        stage('update_readme') {
            steps {
                script {
                    echo "${env.TEST}"
                    env.UPDATE = sh(script: "node jenkinsScripts/update_readme.js $TEST", returnStatus:true)
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
                    env.PUSH = sh(script: "git push origin HEAD:master", returnStatus:true)
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
                        env.DEPLOY = sh(script: 'VERCEL_ORG_ID="$VERCELORGID" VERCEL_PROJECT_ID="$VERCELPROJECTID" vercel --prod --scope jubelltols --token="$VERCELTOKEN"', returnStatus:true)
                    }
                    
                }
            }
        }
        stage('notificacion') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'google-password', variable: 'PASSWORD_GOOGLE'),
                    ]){
                        echo "${env.LINTER}"
                        echo "${env.TEST}"
                        echo "${env.UPDATE}"
                        echo "${env.PUSH}"
                        echo "${env.DEPLOY}"
                        env.CORREO = "${params.correo_notificación}"
                        sh "node jenkinsScripts/notificacion.js"
                    }
                }
            }
        } 
    }
}
