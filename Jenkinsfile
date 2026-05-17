pipeline {
    agent any

    tools {
        maven 'Maven 3.9.14'
        jdk 'JDK17'
    }

    stages {

        stage('Compilation') {
            steps {
                bat 'mvnw compile'
            }
            post {
                failure {
                    mail to: 'admin@gameverseacademy.ma',
                         subject: "ECHEC Compilation - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                         body: "La compilation a echoue. Voir : ${env.BUILD_URL}"
                }
            }
        }

        stage('Tests') {
            parallel {

                stage('Tests Unitaires') {
                    steps {
                        bat 'mvnw test -DskipTests=false'
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/**/*.xml'
                        }
                        failure {
                            mail to: 'admin@gameverseacademy.ma',
                                 subject: "ECHEC Tests Unitaires - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                                 body: "Les tests unitaires ont echoue. Voir : ${env.BUILD_URL}"
                        }
                    }
                }

                stage('Couverture de Code') {
                    steps {
                        bat 'mvnw jacoco:report'
                    }
                    post {
                        always {
                            jacoco(
                                execPattern: 'target/jacoco.exec',
                                classPattern: 'target/classes',
                                sourcePattern: 'src/main/java'
                            )
                        }
                        failure {
                            mail to: 'admin@gameverseacademy.ma',
                                 subject: "ECHEC Couverture - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                                 body: "La couverture de code a echoue. Voir : ${env.BUILD_URL}"
                        }
                    }
                }

                stage('Documentation') {
                    steps {
                        bat 'mvnw site -DskipTests'
                    }
                    post {
                        failure {
                            mail to: 'admin@gameverseacademy.ma',
                                 subject: "ECHEC Documentation - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                                 body: "La generation de documentation a echoue. Voir : ${env.BUILD_URL}"
                        }
                    }
                }

            }
        }

        stage('Packaging') {
            steps {
                bat 'mvnw package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
                failure {
                    mail to: 'admin@gameverseacademy.ma',
                         subject: "ECHEC Packaging - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                         body: "Le packaging a echoue. Voir : ${env.BUILD_URL}"
                }
            }
        }

        stage('Deploiement Nexus') {
            steps {
                bat 'mvnw deploy -DskipTests'
            }
            post {
                success {
                    mail to: 'admin@gameverseacademy.ma',
                         subject: "SUCCES Deploiement - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                         body: "Le projet a ete deploye avec succes sur Nexus. Voir : ${env.BUILD_URL}"
                }
                failure {
                    mail to: 'admin@gameverseacademy.ma',
                         subject: "ECHEC Deploiement Nexus - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                         body: "Le deploiement sur Nexus a echoue. Voir : ${env.BUILD_URL}"
                }
            }
        }

    }

    post {
        always {
            cleanWs()
        }
        failure {
            mail to: 'admin@gameverseacademy.ma',
                 subject: "ECHEC PIPELINE - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                 body: """
                 Le pipeline GameVerseAcademy a echoue.
                 Build numero : ${env.BUILD_NUMBER}
                 URL du build : ${env.BUILD_URL}
                 Branche      : ${env.GIT_BRANCH}
                 Commit       : ${env.GIT_COMMIT}
                 """
        }
    }
}
