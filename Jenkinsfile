
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

                    test -f target/mandali-consultancy-1.0.0.jar

                    sudo /usr/bin/install -o jenkins -g jenkins -m 644 \
                        target/mandali-consultancy-1.0.0.jar \
                        /opt/mandali-app/app.jar

                    sudo /usr/bin/systemctl restart mandali-app

                    for i in $(seq 1 15); do
                        if curl -fsS http://127.0.0.1:8081/ -o /dev/null; then
                            echo "Mandali Consultancy deployed successfully!"
                            exit 0
                        fi
                        sleep 2
                    done

                    echo "Deployment health check failed."
                    sudo /usr/bin/systemctl status mandali-app --no-pager || true
                    exit 1
                

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
