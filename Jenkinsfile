pipeline {
    agent any
    environment {
        IMAGE_BASE = "trialfwrmrd.jfrog.io/sample-nodejs-app/sample-nodejs-app2"
        TAGS = "v1,v2,v3,v4,v5"
    }

    stages {

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jfrog-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo "$PASSWORD" | docker login trialfwrmrd.jfrog.io -u "$USERNAME" --password-stdin'
                }
            }
        }

        stage('Build and Push 5 Docker Images') {
            steps {
                script {
                    def tagList = TAGS.split(',')

                    tagList.each { tag ->
                        sh """
                                echo "// build version ${tag}" > version.txt
                                docker build --build-arg VERSION=${tag} -t ${IMAGE_BASE}:${tag} .
                                docker push ${IMAGE_BASE}:${tag}
                        """
                    }
                }
            }
        }
    	
	stage('Delete Local Images') {
            steps {
                script {
                    def tagList = TAGS.split(',')

                    tagList.each { tag ->
                        sh "docker rmi ${IMAGE_BASE}:${tag} || true"
                    }
                }
            }
        }

        stage('Pull from Artifactory') {
            steps {
                script {
                    def tagList = TAGS.split(',')

                    tagList.each { tag ->
                        sh "docker pull ${IMAGE_BASE}:${tag}"
                    }
                }
            }
        }
    }
}

post {
        success {
            echo "All 5 images were pushed successfully."
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
 }
