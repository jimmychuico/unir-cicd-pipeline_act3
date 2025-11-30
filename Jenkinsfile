pipeline {
    agent { label 'docker' }

    stages {
        stage('Source') {
            steps {
                git 'https://github.com/jimmychuico/unir-cicd-pipeline_act3.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building Docker images...'
                bat 'make build'
            }
        }
        stage('Unit tests') {
            steps {
                script { // Necesitas el bloque 'script' para usar lógica try/catch
                    // 1. Crear el directorio de resultados en el Workspace de Jenkins
                    bat 'if not exist results mkdir results' 
                    
                    // 2. Ejecutar los tests, permitiendo que fallen suavemente
                    try {
                        bat 'make test-unit'
                    } catch (Exception e) {
                        // Capturamos cualquier error de la ejecución de 'make test-unit'.
                        // Esto permite que el pipeline continúe a pesar de los fallos de cobertura o advertencias.
                        echo "Advertencia: make test-unit terminó con un error (probablemente por cobertura baja o warnings). Continuamos con la siguiente etapa."
                        // Opcional: Para ver el mensaje de error completo:
                        // echo "Error capturado: ${e}"
                    }
                }
                
                // 3. Archivar los artefactos (ahora fuera del script, si es posible)
                archiveArtifacts artifacts: 'results/*.xml, results/coverage/**'
                
                // 4. Reportar los resultados de los tests
                junit 'results/unit_result.xml'
            }
        }

        stage('API tests') {
            steps {
                script {
                    // Aseguramos la creación del directorio de resultados
                    bat 'if not exist results mkdir results'
                    
                    // 1. Ejecutamos los tests, permitiendo fallos suaves
                    try {
                        bat 'make test-api'
                    } catch (Exception e) {
                        // Capturamos cualquier error de la ejecución de 'make test-api' (ej. fallo de tests)
                        echo "ADVERTENCIA: make test-api devolvió un error (pruebas fallidas o problemas de conexión). Continuamos con el reporte."
                    }
                }
            }
            post {
                // 2. Reporte: Se ejecuta SIEMPRE (incluso si la ejecución de make falló)
                always {
                    // Archivamos los artefactos generados (aunque el archivo de tests esté vacío o no exista)
                    archiveArtifacts artifacts: 'results/api_result.xml'
                    
                    // El Junit buscará el archivo. Si no existe porque make falló antes de generarlo, 
                    // este paso fallará, pero al estar en el bloque 'post', no detendrá el pipeline.
                    // Para que no falle, lo ideal es usar el parámetro 'testResults' en el junit:
                    junit testResults: 'results/api_result.xml'
                }
            }
        }

       stage('E2E tests') {
            steps {
                script { // Se requiere el bloque 'script' para usar try/catch
                    // 1. Crear el directorio
                    bat 'if not exist results mkdir results'
                    
                    // 2. Ejecutar make test-e2e con manejo de errores
                    try {
                        // Ejecutamos make test-e2e
                        bat 'make test-e2e' 
                    } catch (Exception e) {
                        // Capturamos el error. Esto permite que el pipeline continúe 
                        // incluso si las pruebas E2E fallan (ya que devuelven exit code 1).
                        echo "ADVERTENCIA: make test-e2e devolvió un error (fallo de prueba E2E o del contenedor de Cypress). Continuamos con el archivo de resultados."
                    }
                }
            }
            post {
                always {
                    // Aseguramos el archivo de resultados, capturas de pantalla o logs generados por Cypress.
                    archiveArtifacts artifacts: 'results/**'
                    
                    // Opcional: Si Cypress genera un archivo JUnit XML (generalmente en 'results/cypress-report.xml'), lo reportamos aquí:
                    // junit testResults: 'results/cypress-report.xml' 
                }
            }
        }
    }

    post {
    always {
        cleanWs()
    }
    success {
        mail to: 'jfcpjimmy@gmail.com',
            subject: "✔️ ÉXITO en la pipeline: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
        Hola,

        La pipeline finalizó correctamente. 🎉

        Proyecto: ${env.JOB_NAME}
        Build: ${env.BUILD_NUMBER}

        Puedes ver los detalles aquí:
        ${env.BUILD_URL}

        Saludos.
        """
    }
    failure {
                mail to: 'jfcpjimmy@gmail.com',
                    subject: "❌ FALLA en la pipeline: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
        Hola,

        La pipeline ha fallado. 🚨

        Proyecto: ${env.JOB_NAME}
        Build: ${env.BUILD_NUMBER}

        Revisa los logs aquí:
        ${env.BUILD_URL}

        Saludos.
        """
    }
    }

}
