pipeline {
    agent any

    tools {
        nodejs "node23"
    }

    environment {
        VERCEL_TOKEN = credentials('vercel-token')
        // Configuración de email (puedes mover esto a variables globales en Jenkins)
        EMAIL_DESTINATARIOS = 'marcosvalljo666@gmail.com,marcosvalle546@gmail.com,Danielams_6@hotmail.com,Genarohuertav11@gmail.com,Pabloarellano825@gmail.com'
        EMAIL_REMITENTE = 'tucuenta@gmail.com' // Debe coincidir con tu usuario SMTP
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Conditional Deploy') {
            steps {
                script {
                    def branch = env.GIT_BRANCH ?: sh(
                        script: "git rev-parse --abbrev-ref HEAD",
                        returnStdout: true
                    ).trim()

                    echo "Current branch: ${branch}"

                    if (branch == 'master' || branch == 'origin/master') {
                        echo "Deploying to Vercel..."
                        def output = sh(
                            script: 'npx vercel --prod --token=$VERCEL_TOKEN --yes --name=my-vite-app',
                            returnStdout: true
                        ).trim()
                        def deploymentUrl = output.split('\n').find { it.contains("https") }
                        echo "Deployment URL: ${deploymentUrl}"
                        
                        // Actualiza el environment con la URL de despliegue
                        env.DEPLOYMENT_URL = deploymentUrl
                    } else {
                        echo "No deploy. Just built branch '${branch}'"
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Notificación a Slack
                try {
                    slackSend(
                        channel: '#alertas', 
                        message: "${currentBuild.result == 'SUCCESS' ? '✅' : '❌'} Build ${currentBuild.result} for ${env.JOB_NAME} #${env.BUILD_NUMBER}" +
                                (env.DEPLOYMENT_URL ? "\nDeployment: ${env.DEPLOYMENT_URL}" : "")
                    )
                } catch (Exception e) {
                    echo "Error sending Slack notification: ${e.message}"
                }

                // Notificación por correo mejorada
                try {
                    def emailSubject = "${currentBuild.result == 'SUCCESS' ? '✅ ÉXITO' : '❌ FALLO'}: Build ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                    def emailBody = """
                        <html>
                        <body>
                            <h2>${env.JOB_NAME} - Build #${env.BUILD_NUMBER}</h2>
                            <p><strong>Estado:</strong> ${currentBuild.result}</p>
                            <p><strong>Duración:</strong> ${currentBuild.durationString}</p>
                            <p><strong>URL del Build:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                            ${env.DEPLOYMENT_URL ? "<p><strong>URL de Despliegue:</strong> <a href=\"${env.DEPLOYMENT_URL}\">${env.DEPLOYMENT_URL}</a></p>" : ""}
                            <p><strong>Rama:</strong> ${env.GIT_BRANCH ?: sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()}</p>
                        </body>
                        </html>
                    """
                    
                    emailext(
                        to: env.EMAIL_DESTINATARIOS,
                        subject: emailSubject,
                        body: emailBody,
                        mimeType: 'text/html',
                        replyTo: env.EMAIL_REMITENTE,
                        from: env.EMAIL_REMITENTE
                    )
                } catch (Exception e) {
                    echo "Error sending email notification: ${e.message}"
                    // Intento alternativo con mail básico si falla emailext
                    try {
                        mail(
                            to: env.EMAIL_DESTINATARIOS,
                            subject: "Build ${currentBuild.result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            body: "Estado: ${currentBuild.result}\nVer detalles: ${env.BUILD_URL}",
                            replyTo: env.EMAIL_REMITENTE
                        )
                    } catch (Exception ex) {
                        echo "Fallback email also failed: ${ex.message}"
                    }
                }
            }
        }
    }
}
