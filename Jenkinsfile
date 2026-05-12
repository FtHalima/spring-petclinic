pipeline {
    agent any

    tools {
        maven 'Maven 3.5.2'
        jdk 'jdk1.8.0_151'
    }

    stages {

        // ── ÉTAPE 1 : COMPILATION ──────────────────────────────────
        stage('Build') {
            steps {
                echo '>>> Compilation du projet'
                bat 'mvn compile'
            }
            post {
                failure {
                    mail to: 'halima.ftati@esi.ac.ma',
                         subject: "ÉCHEC Build – ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                         body: "La compilation a échoué.\nURL : ${env.BUILD_URL}"
                }
            }
        }

        // ── ÉTAPE 2 : TESTS EN PARALLÈLE ──────────────────────────
        stage('Tests') {
            parallel {

                stage('Tests Unitaires') {
                    steps {
                        echo '>>> Tests unitaires'
                        bat 'mvn test'
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/**/*.xml'
                        }
                        failure {
                            mail to: 'halima.ftati@esi.ac.ma',
                                 subject: "ÉCHEC Tests Unitaires – ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                                 body: "URL : ${env.BUILD_URL}"
                        }
                    }
                }

                stage('Couverture de Code') {
                    steps {
                        echo '>>> Couverture (Cobertura)'
                        bat 'mvn cobertura:cobertura'
                    }
                    post {
                        always {
                            cobertura coberturaReportFile: 'target/site/cobertura/coverage.xml',
                                      onlyStable: false,
                                      failNoReports: false
                        }
                        failure {
                            mail to: 'halima.ftati@esi.ac.ma',
                                 subject: "ÉCHEC Couverture – ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                                 body: "URL : ${env.BUILD_URL}"
                        }
                    }
                }

                stage('Documentation') {
                    steps {
                        echo '>>> Génération du site Maven'
                        bat 'mvn site'
                    }
                    post {
                        always {
                            publishHTML(target: [
                                allowMissing         : false,
                                alwaysLinkToLastBuild: true,
                                keepAll              : true,
                                reportDir            : 'target/site',
                                reportFiles          : 'index.html',
                                reportName           : 'Maven Site'
                            ])
                        }
                        failure {
                            mail to: 'halima.ftati@esi.ac.ma',
                                 subject: "ÉCHEC Documentation – ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                                 body: "URL : ${env.BUILD_URL}"
                        }
                    }
                }

            } // fin parallel
        }

        // ── ÉTAPE 3 : PACKAGING ────────────────────────────────────
        stage('Packaging') {
            steps {
                echo '>>> Empaquetage JAR/WAR'
                bat 'mvn package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar,target/*.war',
                                     fingerprint: true
                }
                failure {
                    mail to: 'halima.ftati@esi.ac.ma',
                         subject: "ÉCHEC Packaging – ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                         body: "URL : ${env.BUILD_URL}"
                }
            }
        }

        // ── ÉTAPE 4 : DÉPLOIEMENT NEXUS ───────────────────────────
        stage('Déploiement Nexus') {
            steps {
                echo '>>> Déploiement sur Nexus'
                bat 'mvn deploy -DskipTests'
            }
            post {
                failure {
                    mail to: 'halima.ftati@esi.ac.ma',
                         subject: "ÉCHEC Déploiement – ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                         body: "URL : ${env.BUILD_URL}"
                }
            }
        }

    } // fin stages

    // ── NOTIFICATIONS GLOBALES ─────────────────────────────────────
    post {
        success {
            mail to: 'halima.ftati@esi.ac.ma',
                 subject: "SUCCÈS Pipeline – ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Tout s'est bien passé !\nURL : ${env.BUILD_URL}"
        }
        failure {
            echo 'Une étape a échoué — vérifier les logs Jenkins.'
        }
    }

} // fin pipeline
