pipeline {
    agent any

    environment {
        serviceName = 'exam-backend'
    }

    tools {
        maven 'my_mvn'
    }

    stages {
        stage('package') {
            steps {
                sh 'ls'
                echo 'delete target'
                sh 'cd ./exam-admin && rm -rf target'

                echo 'start package'
                sh 'cd ./exam-admin && mvn -B -DskipTests clean package'
                echo 'maven package jar success'
            }
        }

        stage('delete container and images') {
            steps {
                echo 'docker will delete old images'
                sh 'docker rmi -f \$(docker images | grep $serviceName | awk \'{print $3}\') | true'
            }
        }

        stage('build docker image && push local image to docker repository') {
            steps {
                sh 'printenv'
                echo 'docker build image'
                sh 'docker build --no-cache -t $serviceName:${BUILD_NUMBER} -f ./exam-admin/Dockerfile .'
                sh 'docker tag $serviceName:${BUILD_NUMBER} 192.168.56.110:5000/$serviceName:${BUILD_NUMBER}'
                sh 'docker push 192.168.56.110:5000/$serviceName:${BUILD_NUMBER}'
            }
        }

        stage('deploy image to k8s') {
            steps {
                sh "ls"
                sh "sed -i 's#<image-name>#abc.com/$serviceName:${BUILD_NUMBER}#g;' ./k8s.yml"
                sh 'cat k8s.yml'
                sh 'kubectl apply -f ./k8s.yml'
            }
        }
    }
}
