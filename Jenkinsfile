pipeline {
    agent any

    tools {
        nodejs "node23"
    }

    environment {
        VERCEL_TOKEN = credentials('vercel-token')
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
                // Notificación a Slack con manejo de errores
                try {
                    slackSend(
                        channel: '#alertas', 
                        message: "${currentBuild.result == 'SUCCESS' ? '✅' : '❌'} Build ${currentBuild.result} for ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                    )
                } catch (Exception e) {
                    echo "Error sending Slack notification: ${e.message}"
                }

                // Notificación por correo con manejo de errores
                try {
                    if(currentBuild.result == 'SUCCESS') {
                        emailext(
                            to: 'marcosvalljo666,marcosvalle546@gmail.com,Danielams_6@hotmail.com,Genarohuertav11@gmail.com,Pabloarellano825@gmail.com',
                            subject: "✅ Build exitoso: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            body: "El build fue exitoso. Ver: ${env.BUILD_URL}"
                        )
                    } else {
                        emailext(
                            to: 'marcosvalljo666@gmail.com,marcosvalle546@gmail.com,Danielams_6@hotmail.com,Genarohuertav11@gmail.com,Pabloarellano825@gmail.com',
                            subject: "❌ Build fallido: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            body: "Hubo un fallo en el build. Ver detalles: ${env.BUILD_URL}"
                        )
                    }
                } catch (Exception e) {
                    echo "Error sending email notification: ${e.message}"
                }
            }
        }
    }
}
