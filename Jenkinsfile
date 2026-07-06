pipeline {
    agent any

    options {
        // Garde les 10 derniers builds, et coupe un build bloque apres 30 min.
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {

        stage('Checkout') {
            steps {
                // En Multibranch Pipeline, "checkout scm" recupere AUTOMATIQUEMENT
                // la bonne branche (main, dev, feature/...) detectee par Jenkins.
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // On se déplace dans le sous-dossier pour exécuter le build Maven
                dir('projet03-jenkins-maven-tests') {
                    sh 'mvn -B clean compile'
                }
            }
        }

        stage('Test') {
            steps {
                // On se déplace dans le sous-dossier pour exécuter les tests
                dir('projet03-jenkins-maven-tests') {
                    sh 'mvn -B test'
                }
            }
            post {
                always {
                    // Recherche les rapports JUnit spécifiquement dans le sous-dossier du projet
                    junit 'projet03-jenkins-maven-tests/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                dir('projet03-jenkins-maven-tests') {
                    sh 'mvn -B package -DskipTests'
                    // Archive le JAR depuis le bon dossier cible
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Analyse (facultatif)') {
            steps {
                // Analyse statique Checkstyle. catchError : une alerte de style
                // ne doit pas faire echouer tout le pipeline.
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    dir('projet03-jenkins-maven-tests') {
                        sh 'mvn -B checkstyle:checkstyle'
                        publishHTML(target: [
                            reportDir: 'target/site',
                            reportFiles: 'checkstyle.html',
                            reportName: 'Checkstyle',
                            keepAll: true,
                            allowMissing: true,
                            alwaysLinkToLastBuild: true
                        ])
                    }
                }
            }
        }
    }

    post {
        success { echo 'Pipeline termine avec succes.' }
        failure { echo 'Echec du pipeline : consultez les logs de l etape en rouge.' }
    }
}
