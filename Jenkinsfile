/* groovylint-disable NglParseError */
pipeline {
    agent any
    tools{
        maven 'maven'
    }
    environment {
        SCANNER_HOME=tool 'sonarqube'
    }
    stages {
        stage('mvn compile') {
            steps{
                sh 'mvn clean compile'
            }
        }
        stage('Lynis Security Scan') {
            steps {
                script {
                    // Execute the Lynis security scan and convert the output to HTML
                    sh 'lynis audit system | ansi2html > lynis-report.html'

                    // Display the absolute path of the report in the Jenkins console output
                    def reportPath = "${env.WORKSPACE}/lynis-report.html"
                    echo "Chemin du rapport Lynis : ${reportPath}"

                    // Archive the report file for access after the build
                    archiveArtifacts artifacts: 'lynis-report.html'
                }
            }
        }
        //stage('OWASP FS SCAN') {
            //steps {
                //script {
                    // Run OWASP Dependency Check
                   //dependencyCheck additionalArguments: '--scan ./ --format HTML', odcInstallation: 'dpcheck'
                
                    // Publish the Dependency Check report
                  //dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                    //archiveArtifacts artifacts: '**/dependency-check-report.html', allowEmptyArchive: true
                //}
            //}
        //}
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonarqube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=webapplication_ekart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=webapplication_ekart '''
                }
            }
        }
        stage('Build'){
            steps{
                sh "mvn package -DskipTests=true "
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'dockerhub') {
                            sh "docker build -t daggu1997/ekart:latest ."
                   }
               }
            }
        }
        stage('TRIVY') {
            steps {
                script {
                    // Effectue le balayage de sécurité de l'image et écrit la sortie dans un fichier HTML
                    sh 'trivy image --format table --timeout 5m -o trivy-image-report.html daggu1997/ekart:latest'

                    // Affiche le chemin absolu du fichier de rapport dans la console de sortie Jenkins
                    def reportPath = "${WORKSPACE}/trivy-image-report.html"
                    echo "Chemin du rapport Trivy : ${reportPath}"

                    // Archive le fichier de rapport pour qu'il soit accessible après la construction
                    archiveArtifacts artifacts: 'trivy-image-report.html'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
              script {
                  withDockerRegistry(credentialsId: 'dockerhub') {
                            sh "docker push daggu1997/ekart:latest"
                  }
              }
            }
        }
        stage('Deploy To localhost') {
            steps {
    // Check if any container is running and stop it
            script {    
                    
            sh "docker run -d -p 8070:8070 daggu1997/ekart:latest"
    }
}
    
}
        stage('Verify the Deployment') {
            steps {
                sh 'sleep 60' // wait for 5 minutes
                sh 'docker ps'
                sh 'echo "Check the file is running"'
            }
        }
        stage('Nikto Security Scan') {
            steps {
                script {
            // Exécuter Nikto et enregistrer la sortie dans nikto-report.html dans le répertoire de travail
            sh 'nikto -h http://localhost:8070 -o Nikto-report.html'
            def reportPath = "${WORKSPACE}/Nikto-report.html"
            echo "Chemin du rapport Nikto : ${reportPath}"
            // Archive le fichier de rapport pour qu'il soit accessible après la construction
            archiveArtifacts artifacts: 'Nikto-report.html'
                }
            }
    
        }
        stage('Create Pull Request') {
            steps {
                script {
                    // Push changes to the 'dev' branch
                    sh 'git push origin main'
                    
                    // Create a pull request on GitHub
                    githubCreatePullRequest(
                        repoOwner: 'Rajendra0609',
                        repoName: 'Ekart',
                        title: 'Merge dev/docker into main',
                        body: 'This pull request merges the dev branch into the main branch.',
                        base: 'main',
                        head: 'dev/docker'
                    )
                }
            }
        }
    }
}


