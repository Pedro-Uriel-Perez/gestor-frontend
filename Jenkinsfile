// Pipeline del frontend: compila, prueba y empaqueta con JDK 21 y Maven 3.9 configurados en Jenkins
pipeline {
    agent any

    tools {
        jdk 'jdk-21'          // Manage Jenkins > Tools > JDK
        maven 'maven-3.9'     // Manage Jenkins > Tools > Maven
    }

    options {
        timestamps()
    }

    stages {
        stage('Versiones') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
            }
        }
        stage('Compilar') {
            steps {
                sh 'mvn -B clean compile'
            }
        }
        stage('Pruebas') {
            steps {
                // -Pci omite las pruebas de navegador (Playwright), que se ejecutan en la máquina local
                sh 'mvn -B -Pci test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec', classPattern: 'target/classes', sourcePattern: 'src/main/java'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                // 'SonarQube' es el nombre del servidor en Manage Jenkins > System > SonarQube servers
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn -B -Pci sonar:sonar'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                // SonarQube responde por webhook (http://jenkins-gestor:8080/sonarqube-webhook/)
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Empaquetar') {
            steps {
                sh 'mvn -B -Pci package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
}
