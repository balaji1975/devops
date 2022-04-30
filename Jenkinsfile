def sendEmail (String addresses, String subject, String body)
{
    echo "Sending Email to ${addresses}, Subject: ${subject} Body: ${body}"
    emailext(mimeType: 'text/html', subject: "${subject}", to: "${addresses}", body: "${body}")
}

def sendEmail (String addresses, String subject, String body, String attachedFile)
{
    echo "Sending Email to ${addresses}, Subject: ${subject} Body: ${body}"
    emailext(mimeType: 'text/html', subject: "${subject}", to: "${addresses}", body: "${body}", attachmentsPattern: "${attachedFile}")
}

pipeline {
    agent any
    stages {
		stage ('Code Pull') {
			steps {
				dir ("${appName}") {
					git credentialsId: 'git.econ.census.gov', branch: "${branchName}",
					url: "${giturl}";
				}
            }
            post {
                success { echo "Successfully completed Code Pull"}
                failure { echo "Code Pull Failed"}
            }
		}
        stage ('Compile Stage') {
            steps {
                
                    sh '/data/jenkins/tools/apache-maven-3.5.2/bin/mvn clean compile'
                
            }
            post {
                success { echo "Successfully completed Compile"}
                failure { echo "Compile Stage Failed"}
            }
        }
        stage ("SonarQube analysis") {
            steps {
                withSonarQubeEnv('Sonarqube6.7.2') {
                    
                        sh '/data/jenkins/tools/apache-maven-3.5.2/bin/mvn sonar:sonar ' +
	                    '-Dsonar.projectName=${JOB_NAME} ' +
                        '-Dsonar.projectKey=${JOB_NAME} ' +
	                    '-Dsonar.projectVersion=${BUILD_NUMBER}'
                     }
                }
	    }
	    stage ("SonarQube Quality Gate") {
            steps {
           	    timeout(time: 1, unit: 'HOURS') {   
                    script {
                        
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            sh """
                            curl -o ${JOB_NAME}_${BUILD_NUMBER}.html https://sso.econ.census.gov/controller/Sonarqube/projectViolationsReportMainHTML_get/maven_pipeline_gitlab
                            curl -o ${JOB_NAME}.html https://sso.econ.census.gov/controller/Sonarqube/projectViolationsReportHTML_get/maven_pipeline_gitlab
                            ls -ltr *.html
                            """
                            sendEmail ("balaji.r.regeti@census.gov","Attachment Testing","Attaching FIle","${JOB_NAME}_${BUILD_NUMBER}.html")
                            emailext attachmentsPattern: '*.html', body: '''Hi ALL,
                            Please find attached SonarQube Analysis Report for the Build -- ${JOB_NAME} and Build #${BUILD_NUMBER}
                            ${FILE,path="${JOB_NAME}_${BUILD_NUMBER}.html"}
                            <br>
                            PLEASE DO NOT REPLY TO THIS EMAIL!!!
                            </br>
                            <br>
                            Thanks</br><br>
                            EIS MASSDB TEAM </br>''', mimeType: 'text/html', subject: 'SonarQube Analysis report for the Build -- ${JOB_NAME} and Build#${BUILD_NUMBER}', to: 'balaji.r.regeti@census.gov'
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                        else {
                            sh """
                            curl -o ${JOB_NAME}_${BUILD_NUMBER}.html https://sso.econ.census.gov/controller/Sonarqube/projectViolationsReportMainHTML_get/maven_pipeline_gitlab
                            curl -o ${JOB_NAME}.html https://sso.econ.census.gov/controller/Sonarqube/projectViolationsReportHTML_get/maven_pipeline_gitlab
                            ls -ltr *.html
                            """
                            sendEmail ("balaji.r.regeti@census.gov","Attachment Testing","Attaching FIle","${JOB_NAME}_${BUILD_NUMBER}.html")

                            emailext attachmentsPattern: '${JOB_NAME}_${BUILD_NUMBER}.html, ${JOB_NAME}.html', body: '''Hi ALL,
                            Jenkins Build -- ${JOB_NAME} and Build #${BUILD_NUMBER} is success
                            PLEASE DO NOT REPLY TO THIS EMAIL!!!
                            Thanks
                            EIS MASSDB TEAM''', subject: 'Jenkins Build -- ${JOB_NAME} and Build#${BUILD_NUMBER}', to: 'balaji.r.regeti@census.gov'
                        }
                    }
                }
            }
        }
        stage ('Testing Stage') {
            steps {
                
                    sh """
                        /data/jenkins/tools/apache-maven-3.5.2/bin/mvn test | tee balaji_test_results.txt
                        ls -l balaji_test_results.txt
                        echo "BALAJI-DEBUG ========================================================"
                        sed -n -e '/Results:/,\$p' balaji_test_results.txt  | head -3 |  sed s/INFO//g > results.txt
                        echo "BALAJI-DEBUG ========================================================"
                    """
                    sendEmail("balaji.r.regeti@census.gov","Test Results","<pre>"+readFile("results.txt")+"</pre>", "balaji_test_results.txt")
                
            }
        }
        stage ('Deploy Stage') {
            steps {
                
                    sh '/data/jenkins/tools/apache-maven-3.5.2/bin/mvn install'
                
            }
        }
    }
}
