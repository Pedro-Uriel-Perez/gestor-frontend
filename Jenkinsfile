// Pipeline FRONTEND con JDK 21 y Maven 3.9, priorizando la calidad:
// compila → pruebas (JUnit + JaCoCo) → SonarQube + Quality Gate → escaneo de vulnerabilidades (Trivy) → empaqueta
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
                withSonarQubeEnv('SonarQube') {    // Manage Jenkins > System > SonarQube servers
                    sh 'mvn -B -Pci sonar:sonar'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                // SonarQube avisa el resultado por webhook; si el Quality Gate no pasa, el pipeline se detiene
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Escaneo de vulnerabilidades (Trivy)') {
            steps {
                // Dependencias del pom.xml y secretos; se excluyen las carpetas target y .mvn.
                // --offline-scan usa las dependencias ya descargadas por Maven (~/.m2) en lugar de consultar Maven Central.
                // Los hallazgos revisados y aceptados están justificados en .trivyignore.
                // El pipeline falla si queda alguna vulnerabilidad HIGH o CRITICAL sin atender.
                sh '''
                    trivy fs --offline-scan --scanners vuln,secret,misconfig --skip-dirs target --skip-dirs .mvn \
                         --ignorefile .trivyignore --format table --output trivy-gestor-frontend.txt .
                    cat trivy-gestor-frontend.txt
                    trivy fs --offline-scan --scanners vuln,secret,misconfig --skip-dirs target --skip-dirs .mvn \
                         --ignorefile .trivyignore --severity HIGH,CRITICAL --exit-code 1 --quiet .
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-gestor-frontend.txt', allowEmptyArchive: true
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
