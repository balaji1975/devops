pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        
        Stage('acceptence') {
            input
               {
            message 'Please confirm to proceed to build stage'
            ok 'Yes'
            submitter 'regeti'
               }
        }
       
        stage('Build step') {
            steps {
                echo 'Starting Build'
            }
        }
    }
}
