pipeline {
    agent any

    environment {
        APP_USER   = 'realretar'
        NODE_01    = '192.168.100.21'
        NODE_02    = '192.168.100.22'
        DEPLOY_DIR = '/home/realretar/cinema-app'
    }

    stages {
        stage('1. Environment Setup') {
            steps {
                echo 'Preparing deployment dirs on App Nodes'
                // Create the application directory on both servers if it doesnt exist
                sh "ssh ${APP_USER}@${NODE_01} 'mkdir -p ${DEPLOY_DIR}'"
                sh "ssh ${APP_USER}@${NODE_02} 'mkdir -p ${DEPLOY_DIR}'"
            }
        }

        stage('2. Build & Package') {
            steps {
                echo 'Compiling Java code and building executable JAR file'
                // Maven compiles the code and creates a standalone .jar file in the /target folder
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('3. Deploy') {
            steps {
                echo 'Copying compiled JAR files to App Nodes with SCP'
                // We use * because Maven names the file based on the version in pom.xml
                sh "scp target/*.jar ${APP_USER}@${NODE_01}:${DEPLOY_DIR}/cinema-booking.jar"
                sh "scp target/*.jar ${APP_USER}@${NODE_02}:${DEPLOY_DIR}/cinema-booking.jar"
            }
        }

        stage('4. Launch Application') {
            steps {
                echo 'Restarting Spring Boot services on App Nodes'
                
                sh "ssh -n -o StrictHostKeyChecking=no ${APP_USER}@${NODE_01} bash ${DEPLOY_DIR}/restart-app.sh"
                sh "ssh -n -o StrictHostKeyChecking=no ${APP_USER}@${NODE_02} bash ${DEPLOY_DIR}/restart-app.sh"
            }
        }
    }

    post {
        success {
            echo 'Deployment complete'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
