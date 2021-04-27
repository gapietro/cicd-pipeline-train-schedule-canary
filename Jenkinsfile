pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "gapietro/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                script {
                    env.requestBody = "{\"zoneId\":1,\"instance\":{\"name\":\"TSDEV - Build ${env.BUILD_NUMBER}\",\"cloud\":\"Pietro Local VMWare\",\"site\":{\"id\":1},\"type\":\"ts\",\"instanceType\":{\"code\":\"ts\"},\"instanceContext\":\"dev\",\"layout\":{\"id\":1202,\"code\":\"760ffff8-d86b-4118-8c3f-8de1c70d90e1\"},\"plan\":{\"id\":116,\"code\":\"container-256\",\"name\":\"256MB Memory, 3GB Storage\"}},\"config\":{\"resourcePoolId\":15,\"poolProviderType\":\"kubernetes\",\"customOptions\":{\"f_tsver\":\"${env.BUILD_NUMBER}\"},\"createUser\":true},\"volumes\":[{\"id\":-1,\"rootVolume\":true,\"name\":\"root\",\"size\":3,\"sizeId\":null,\"storageType\":null,\"datastoreId\":12}],\"ports\":[{\"name\":\"HTTP\",\"port\":31443,\"lb\":\"HTTP\"}]}"

                }  
                def cdresponse = httpRequest(url: 'https://192.168.10.104/api/instances', acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', ignoreSslErrors: true, customHeaders: [[name: 'Authorization', value: 'Bearer e125ccff-6c05-4664-a21a-500f98e693cc']], requestBody: "${env.requestBody}", responseHandle: 'STRING', validResponseCodes: '200')
                def props = readJSON text : env.cdresponse
                println("ID: "+cdresponse.content)
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                script {
                    env.requestBody = "{\"zoneId\":1,\"instance\":{\"name\":\"TSDEV - Build ${env.BUILD_NUMBER}\",\"cloud\":\"Pietro Local VMWare\",\"site\":{\"id\":1},\"type\":\"ts\",\"instanceType\":{\"code\":\"ts\"},\"instanceContext\":\"dev\",\"layout\":{\"id\":1202,\"code\":\"760ffff8-d86b-4118-8c3f-8de1c70d90e1\"},\"plan\":{\"id\":116,\"code\":\"container-256\",\"name\":\"256MB Memory, 3GB Storage\"}},\"config\":{\"resourcePoolId\":15,\"poolProviderType\":\"kubernetes\",\"customOptions\":{\"f_tsver\":\"${env.BUILD_NUMBER}\"},\"createUser\":true},\"volumes\":[{\"id\":-1,\"rootVolume\":true,\"name\":\"root\",\"size\":3,\"sizeId\":null,\"storageType\":null,\"datastoreId\":12}],\"ports\":[{\"name\":\"HTTP\",\"port\":31443,\"lb\":\"HTTP\"}]}"

                }  
                httpRequest(url: 'https://192.168.10.104/api/instances', acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', ignoreSslErrors: true, customHeaders: [[name: 'Authorization', value: 'Bearer e125ccff-6c05-4664-a21a-500f98e693cc']], requestBody: "${env.requestBody}", responseHandle: 'STRING', validResponseCodes: '200')
            
                script {
                    env.requestBody = "{\"zoneId\":1,\"instance\":{\"name\":\"TSDEV - Build ${env.BUILD_NUMBER}\",\"cloud\":\"Pietro Local VMWare\",\"site\":{\"id\":1},\"type\":\"ts\",\"instanceType\":{\"code\":\"ts\"},\"instanceContext\":\"dev\",\"layout\":{\"id\":1202,\"code\":\"760ffff8-d86b-4118-8c3f-8de1c70d90e1\"},\"plan\":{\"id\":116,\"code\":\"container-256\",\"name\":\"256MB Memory, 3GB Storage\"}},\"config\":{\"resourcePoolId\":15,\"poolProviderType\":\"kubernetes\",\"customOptions\":{\"f_tsver\":\"${env.BUILD_NUMBER}\"},\"createUser\":true},\"volumes\":[{\"id\":-1,\"rootVolume\":true,\"name\":\"root\",\"size\":3,\"sizeId\":null,\"storageType\":null,\"datastoreId\":12}],\"ports\":[{\"name\":\"HTTP\",\"port\":31443,\"lb\":\"HTTP\"}]}"

                }  
                httpRequest(url: 'https://192.168.10.104/api/instances', acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', ignoreSslErrors: true, customHeaders: [[name: 'Authorization', value: 'Bearer e125ccff-6c05-4664-a21a-500f98e693cc']], requestBody: "${env.requestBody}", responseHandle: 'STRING', validResponseCodes: '200')
            
            }
        }
    }
}
