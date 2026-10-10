pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-21-amazon-corretto.x86_64'
        PATH = "/usr/lib/jvm/java-21-amazon-corretto.x86_64/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Java and Maven') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -B test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn -B package -DskipTests'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                    set -eu

                    echo "Starting Mandali Consultancy deployment..."

                    test -f "$WORKSPACE/target/mandali-consultancy-1.0.0.jar"

                    sudo /usr/local/sbin/deploy-mandali-app

                    echo "Mandali Consultancy deployment completed successfully!"
                '''
            }
        }
    }

    post {
        success {
            echo 'Mandali Consultancy pipeline and deployment completed successfully!'
        }

        failure {
            echo 'Pipeline or deployment failed. Check the console output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}