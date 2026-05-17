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
                    mail to: 'ftatipy.2022@gmail.com',
                         subject: "ECHEC Compilation - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                         body: "La compilation a echoue. Voir : ${env.BUILD_URL}"
                }
            }
        }

        stage('Tests') {
            parallel {

                stage('Tests Unitaires') {
                    steps {
                        bat 'mvnw test'
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/**/*.xml'
                        }
                        failure {
                            mail to: 'ftatipy.2022@gmail.com',
                                 subject: "ECHEC Tests - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                                 body: "Les tests ont echoue. Voir : ${env.BUILD_URL}"
                        }
                    }
                }

                stage('Couverture de Code') {
                    steps {
                        bat 'mvnw test jacoco:report -DskipTests=false'
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
                            mail to: 'ftatipy.2022@gmail.com',
                                 subject: "ECHEC Couverture - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                                 body: "La couverture a echoue. Voir : ${env.BUILD_URL}"
                        }
                    }
                }

                stage('Documentation') {
                    steps {
                        bat 'mvnw site -DskipTests'
                    }
                    post {
                        failure {
                            mail to: 'ftatipy.2022@gmail.com',
                                 subject: "ECHEC Documentation - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                                 body: "La documentation a echoue. Voir : ${env.BUILD_URL}"
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
                    mail to: 'ftatipy.2022@gmail.com',
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
                    mail to: 'ftatipy.2022@gmail.com',
                         subject: "SUCCES Pipeline - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                         body: "Pipeline termine avec succes. Voir : ${env.BUILD_URL}"
                }
                failure {
                    mail to: 'ton.email@gmail.com',
                         subject: "ECHEC Deploiement - GameVerseAcademy - Build #${env.BUILD_NUMBER}",
                         body: "Le deploiement Nexus a echoue. Voir : ${env.BUILD_URL}"
                }
            }
        }

    }

    post {
        always {
            cleanWs()
        }
    }
}
