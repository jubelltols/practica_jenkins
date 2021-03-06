# Práctica Github Actions

Esta práctica consistirá en aplicar una serie de mejoras sobre un proyecto creado con el framework next.js [link](https://nextjs.org/) y que podéis descargar del siguiente repositorio de github [link](https://github.com/dawEstacio/nextjs-blog-practica). Para ello, partiendo de este código, deberéis crear un repositorio en vuestra cuenta de github e incluir en él un .jenkinsfile para poder ejecutar una pipeline.

## 1.Jenkins
### 1.1.Qué es?
Servidor de código abierto que se puede utilizar para para automatizar todo tipo de tareas relacionadas con la creación, prueba y entrega o implementación de software.

Jenkins usa la arquitectura maestro/esclavo:
- Un nodo maestro se encarga de programar los trabajos, asignar esclavos y enviar compilaciones a los esclavos para ejecutar los trabajos. También realizará un seguimiento del estado del esclavo (fuera de línea o en línea) y recuperará las respuestas de los resultados de la compilación de los esclavos y las mostrará en la salida de la consola.
- Un nodo esclavo recibe peticiones de un nodo master y ejecuta los trabajos de construcción que éste le envíe.

### 1.2.Tipos de tareas
- Proyecto de estilo libre. Permite ejecutar tareas simples de forma rápida
- Pipeline. Permite ejecutar un flujo de trabajo entero
- Multiconfiguración. Permite ejecutar la misma tarea en múltiples entornos
- Carpeta. Permite crear directorios para organizar mejor las tareas creadas
- Organización github. Escanea una cuenta de github y crea pipelines para cada repositorio que contenga
- Multibranch pipeline. Permite implementar distintos pipelines dependiendo de la ramas de un proyecto

### 1.3.Configuracion de una pipeline

Para configurar una pipeline nos dirigiremos al menu de la izquierda en jenkins y haremos click en nueva tarea.
Seguidamente añadiremos el nombre y selecionaremos pipelane.

![jenkins](https://github.com/jubelltols/practica_jenkins/blob/master/img/img2.png)
![jenkins](https://github.com/jubelltols/practica_jenkins/blob/master/img/img1.png)

Ahora debemos ir a la seccion pipeline y añadir el repositorio de github y las credenciales necesaria

![jenkins](https://github.com/jubelltols/practica_jenkins/blob/master/img/img3.png)

Por ultimo especificaremos la rama y el nombre del jenkinsfile y guardar.

![jenkins](https://github.com/jubelltols/practica_jenkins/blob/master/img/img6.png)

## 2.Parametros de la pipeline

Añadiremos tres parámetros:
- Ejecutor: de tipo texto en el que se especificará el nombre de la persona que ejecuta la pipeline
- Motivo: de tipo texto también en que podremos especificar el motivo por el cual estamos ejecutando la pipeline.
- Correo notificación: de tipo texto que almacenará el correo al que notificaremos el resultado de cada stage ejecutado

```
    parameters {
        string(name: 'ejecutor', defaultValue: 'jubelltols', description: 'Nombre de la persona que ejecuta la pipeline')
        string(name: 'motivo', defaultValue: 'Prueba praactica jenkins', description: 'Motivo por el cual estamos ejecutando la pipeline')
        string(name: 'correo_notificación', defaultValue: 'jubelltols@gmail.com', description: 'Correo al que notificaremos el resultado de cada stage ejecutado')
    }
```

## 3.Verificar cada 3 horas cambios

Añadimos un trigger para verificar cada 3 horas si se han producido cambios en el repositorio y, de ser así, se ejecutarà la pipeline

```
    triggers {
        pollSCM('0 */3 * * *')
    }
```

## 4.Linter stage
Se encargará de ejecutar el linter que ya está instalado en el proyecto (existe un script para ello en el package.json) para verificar que la sintaxis utilizada es la correcta en todos los ficheros javascript. En caso de que existan errores corregirlos hasta que el job se ejecute sin problemas.
Este stage esta formado por un step que:
- Descargara las dependencias 
- Ejecutara los test

```
    stage('linter') {
        steps {
            script {
                sh "npm install"
                env.LINTER = sh(script: "npm run lint", returnStatus:true)
            }
        }
    }
```
## 5.Cypress stage
Se encargará de ejecutar los tests de cypress [link](https://www.cypress.io/) que contiene el proyecto.
Este stage esta formado por un step que:
- Descargara las dependencias 
- Construira el proyecto
- Iniciara el proyecto en segundo plano
- Ejecutara el test cypress

```
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
```

## 6.Update_Readme stage
Se encargará de publicar en el readme del proyecto el badge que indicará si se han superado los tests de cypress o no.
Este stage esta formado por un step que:
- Ejecutara un script que sera el encargado de actualizar el badge.

```
    stage('update_readme') {
        steps {
            script {
                echo "${env.TEST}"
                env.UPDATE = sh(script: "node jenkinsScripts/update_readme.js $TEST", returnStatus:true)
            }
        }
    }
```

### 6.1.Crear update_readme.js

```
    const fs = __nccwpck_require__(147);
    const core = __nccwpck_require__(453);
    const process = require('process');

    function create_badge(){

        var result = process.argv[2];
        var badge = "";

        console.log(result);
        
        if(result == 0){
            badge = "\n ![Generic badge](https://img.shields.io/badge/tested%20with-Cypress-04C38E.svg) \n";
        }else{
            badge = "\n ![Generic badge](https://img.shields.io/badge/test-failure-red) \n";
        }
        
        fs.readFile('./README.md', 'utf8', function (err, data) {
            if (err) {
                return console.log(err);
            }

            var result = data.replace(/\<\!\-\-\-badge\-\-\-\>((.|[\n|\r|\r\n])*?)\<\!\-\-\-badge\-\-\-\>[\n|\r|\r\n]?(\s+)?/g,"<!---bagde---> + bagde + <!---bagde--->");

            fs.writeFile('./README.md', result, 'utf8', function (err) {
                if (err) return console.log(err);
            });
        });
        
    }

    create_badge();
```

### 7.Push_Changes stage
Se encargará de ejecutar el add, commit y push de los cambios del readme a nuestro repositorio de código. El comentario del commit
deberá seguir el siguiente formato:

```
Pipeline ejecutada por PARAM_EJECUTOR. Motivo: PARAM_MOTIVO
```

Este stage esta formado por un step que:
- Configuracion necesaria de git
- Git Add, Commit y psuh

```
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
```
## 8.Deploy to Vercel stage
Ejecutará un script encargado de publicar el proyecto en la plataforma vercel [link](https://vercel.com/).

### 8.1.Crear cuenta vercel y obtener token de vercel

![token vercel](https://github.com/jubelltols/practica_jenkins/blob/master/img/img11.png)

### 8.2.Instalar Vercel localmente
```
npm i vercel
```
### 8.3.Linkear el proyecto a vercel
```
npx vercel link o vercel
```
### 8.4.Añadir credential a Jenkins
Para añadir una credential a jenkins nos dirigiremos a la configuracion de la pipeline.

![jenkins credential](https://github.com/jubelltols/practica_jenkins/blob/master/img/img12.png)

Seguidamente nos dirigiremos a la sección pipeline > credentials > add > Jenkins

![jenkins credential](https://github.com/jubelltols/practica_jenkins/blob/master/img/img3.png)

Por ultimo seleccionaremos el tipo, añadiremos el secret y id

![jenkins credential](https://github.com/jubelltols/practica_jenkins/blob/master/img/img7.png)

Para utilizar vercel deberemos crear las siguientes credentiales:
 - Id de la organizzacion
 - Id del projecto de vercel 
 - Token de vercel
### 8.5.Añadir al jenkinsfile
```
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
```
### 8.6.Link al despliege de la aplicación

[https://practica-jenkins.vercel.app/](https://practica-jenkins.vercel.app/)

## 9.Notification stage
Se encargará de enviar un correo con:

* Destinatario: dirección de correo establecida como parámetro
* Asunto: Resultado de la pipeline ejecutado
* Cuerpo del mensaje:
    Se ha realizado un push en la rama main que ha provocado la ejecución de la
    pipeline jenkins-practica con los siguientes resultados:
    - Linter_stage: resultado asociado
    - Test_stage: resultado asociado
    - Update_readme_stage: resultado asociado
    - Deploy_to_Vercel_stage: resultado asociado

### 9.1.Crear notificacion.js

```
    var LINTER = process.env.LINTER == 0 ? 'Success' : 'Failed'
    var TEST = process.env.TEST == 0 ? 'Success' : 'Failed'
    var UPDATE = process.env.UPDATE == 0 ? 'Success' : 'Failed'
    var PUSH = process.env.PUSH == 0 ? 'Success' : 'Failed'
    var DEPLOY = process.env.DEPLOY == 0 ? 'Success' : 'Failed'

    const nodemailer = __nccwpck_require__(832);
    
    var transporter = nodemailer.createTransport({
        host: 'smtp.gmail.com',
        port: 465,
        secure: true,
        auth: {
            user: process.env.CORREO,
            pass: process.env.PASSWORD_GOOGLE
        }
    });
    
    var mailOptions = {
        from: process.env.CORREO, 
        to: process.env.CORREO, 
        subject: 'Resultado de la pipeline ejecutada',
        html: `
            <div>   
                <p>Se ha realizado un push en la rama main que ha provocado la ejecución de  la pipeline de practica_jenkins con los siguientes resultados: </p>
                <ul>
                    <li>Linter_stage: ${LINTER} </li>
                    <li>Test_stage: ${TEST} </li>
                    <li>Update_readme_stage: ${UPDATE} </li>
                    <li>Push_changes_stage: ${PUSH} </li>
                    <li>Deploy_to_Vercel_stage: ${DEPLOY} </li>
                </ul>
            </div>
        ` 
    };
    
    transporter.sendMail(mailOptions, function(error, info){
        if (error) {
            console.log(error);
        } else {
            console.log('Email sent: ' + info.response);
        }
    });
```
### 9.2.Crear una contraseña de aplicaion en gmail para poder permitir a nodemailer enviar el correo.

![contraseña de aplicaion en gmail](https://github.com/jubelltols/practica_jenkins/blob/master/img/img4.png)
![contraseña de aplicaion en gmail](https://github.com/jubelltols/practica_jenkins/blob/master/img/img9.png)

### 9.3.Añadir credential a Jenkins
Para añadir una credential a jenkins nos dirigiremos a la configuracion de la pipeline.

![jenkins credential](https://github.com/jubelltols/practica_jenkins/blob/master/img/img12.png)

Seguidamente nos dirigiremos a la sección pipeline > credentials > add > Jenkins

![jenkins credential](https://github.com/jubelltols/practica_jenkins/blob/master/img/img3.png)

Por ultimo seleccionaremos el tipo, añadiremos el secret y id

![jenkins credential](https://github.com/jubelltols/practica_jenkins/blob/master/img/img7.png)

Para enviar el correo necesitaremos crear una credentia en la que se almacene la contraseña del corroe.

### 9.4.Añadir al jenkinsfile
```
    stage('notificacion') {
        steps {
            script {
                withCredentials([
                    string(credentialsId: 'google-password', variable: 'PASSWORD_GOOGLE'),
                ]){
                    env.CORREO = "${params.correo_notificación}"
                    sh "node jenkinsScripts/notificacion.js"
                }
            }
        }
    } 
```
### 9.5.Correo electronico de los resultados de las actions
![correo rusltados](https://github.com/jubelltols/practica_jenkins/blob/master/img/img8.png)

## 10.Jenkinsfile 

```
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
                        env.DEPLOY = sh(script: 'ERCEL_ORG_ID="$VERCELORGID" VERCEL_PROJECT_ID="$VERCELPROJECTID" vercel --prod --scope jubelltols --token="$VERCELTOKEN"', returnStatus:true)
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
```

## 11.Resultados construccion con parametros de la pipeline

![jenkins result](https://github.com/jubelltols/practica_jenkins/blob/master/img/img10.png)

## 12.Plugin de dashboard view 
Instalar el plugin de dashboard view  y configura un dashboard en jenkins para visualizar de forma customizada el resultado de los jobs ejecutados.

- Nos dirigiremos al menu da la parte izquierda en jenkins y click en administar jenkins

![dashboard view](https://github.com/jubelltols/practica_jenkins/blob/master/img/img20.png)

- Haremos click en Administrar Plugins

![dashboard view](https://github.com/jubelltols/practica_jenkins/blob/master/img/img21.png)

- Buscaremos en todos los plugins "dashboard view", seleccionamos el plugin y Install without restart

![dashboard view](https://github.com/jubelltols/practica_jenkins/blob/master/img/img22.png)

- Ahora iremos a mis vistas y click en + 

![dashboard view](https://github.com/jubelltols/practica_jenkins/blob/master/img/img23.png)

- Añadimos el nombre de la nueva vista y su tipo

![dashboard view](https://github.com/jubelltols/practica_jenkins/blob/master/img/img24.png)

- Añadimos los proyectos que veremos en la dashboard

![dashboard view](https://github.com/jubelltols/practica_jenkins/blob/master/img/img25.png)

- Añadimos los portlets del tablero

![dashboard view](https://github.com/jubelltols/practica_jenkins/blob/master/img/img26.png)

- Vista de la Dashboard

![dashboard view](https://github.com/jubelltols/practica_jenkins/blob/master/img/img27.png)

# RESULTADO DE LOS ÚLTIMOS TESTS

<!---badge--->
 ![Generic badge](https://img.shields.io/badge/tested%20with-Cypress-04C38E.svg) 
<!---badge--->