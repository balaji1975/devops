pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        
        stage('acceptence') {
            input
               {
            message 'Please confirm to proceed to build stage'
            ok 'Yes'
               }
        }
       
        stage('Build step') {
            steps {
                echo 'Starting Build'
            }
        }
    }
}
