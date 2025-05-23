pipeline {
    agent any

    tools {
        maven 'Maven 3.8.1'  // Ensure this tool name matches your Jenkins tool config
    }

    environment {
        STAGING_SERVER = 'ec2-user@staging-server-ip'
        PROD_SERVER = 'ec2-user@production-server-ip'
        ARTIFACT = 'target/myapp.jar'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building the project...'
                sh 'mvn clean package'
            }
        }

        stage('Unit and Integration Tests') {
            steps {
                echo 'Running unit and integration tests...'
                sh 'mvn test'
                sh 'mvn verify'
            }
        }

        stage('Code Analysis') {
            steps {
                echo 'Running static code analysis with SonarQube...'
                withSonarQubeEnv('SonarQube Server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Security Scan') {
            steps {
                echo 'Running OWASP Dependency-Check...'
                sh 'mvn org.owasp:dependency-check-maven:check'
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo 'Deploying to staging server...'
                sh """
                    scp ${ARTIFACT} ${STAGING_SERVER}:/home/ec2-user/
                    ssh ${STAGING_SERVER} 'java -jar /home/ec2-user/myapp.jar &'
                """
            }
        }

        stage('Integration Tests on Staging') {
            steps {
                echo 'Running integration tests on staging...'
                // Replace with your test framework or scripts
                sh 'newman run postman_collection.json -e staging_environment.json'
            }
        }

        stage('Deploy to Production') {
            steps {
                input message: 'Approve deployment to production?'
                echo 'Deploying to production server...'
                sh """
                    scp ${ARTIFACT} ${PROD_SERVER}:/home/ec2-user/
                    ssh ${PROD_SERVER} 'pkill -f myapp.jar || true'
                    ssh ${PROD_SERVER} 'java -jar /home/ec2-user/myapp.jar &'
                """
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
