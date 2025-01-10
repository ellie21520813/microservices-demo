def images= ""
pipeline  {
    agent any
    tools {
      maven 'mvn'
    } 
    environment {
        //gitlab
        REPO_URL = 'http://10.30.182.20/root/testcase1.git'
        //harbor
        HARBOR_URL = 'harbor.securityzone.vn'
        IMAGE_GROUP = 'testcase1'
        TAG = 'latest'
        SONAR_URL='http://10.30.182.22'
        SCANNER_HOME= tool 'sonar-scanner'
    }   
    stages{
        stage('Check Repo') {
            steps {
                script {
                    def exitCode = sh(script: "git ls-remote $REPO_URL", returnStatus: true) 
                    if (exitCode != 0) {
                        error("Repository does not exist!")
                    }
                }
            }
        }
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }
        stage('Static Code Analysis') {
            steps {
                    sh '''
	                    $SCANNER_HOME/bin/sonar-scanner -X \
            		    -Dsonar.host.url=${SONAR_URL} \
            		    -Dsonar.login=squ_a9ee65fe301624ed17f2ab2ef4c30dc610b370c9 \
            		    -Dsonar.projectName=group20\
            		    -Dsonar.java.binaries=. \
            		    -Dsonar.projectKey=group20 \
                        '''
                
            }
        }
        
        stage('Build and Push Docker Images') {
            steps {
                script {
                    sh 'docker compose -f ./deploy/docker-compose/docker-compose.yml pull'
                    images = sh(script: "docker compose -f ./deploy/docker-compose/docker-compose.yml config | grep 'image:' | awk '{print \$2}'", returnStdout: true).trim()
                    images.split('\n').each {
                        image -> def imageName = image.split(":")[0]
                        def imageTag = "${HARBOR_URL}/${IMAGE_GROUP}/${imageName}:${BUILD_NUMBER}"
                        sh "docker tag ${image} ${imageTag}"
                        sh "docker push ${imageTag}"
                    }
                }
            }
        }
        stage('Trigger ManifestUpdate') {
            steps {
                echo "Update manifestjob"
                build job: 'update_manifesh_repo', 
                    parameters: [
                        string(name: 'DOCKERTAG', value: "${BUILD_NUMBER}"),
                        string(name: 'images', value: "${images}")
                    ]   
            }
        }
    }
}