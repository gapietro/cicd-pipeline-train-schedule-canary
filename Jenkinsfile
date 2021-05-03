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
                    def name = "TSSTAGE - Build ${env.BUILD_NUMBER}"
                    def layout = 1204
                    def layoutcode = "fe027d2f-71e4-4e54-a3d1-90cf7892a571"
                    def ports = 31444
                    def canary_replicas = 1
                    
                    env.requestBody = "{\"zoneId\":1,\"instance\":{\"name\":\"${name}\",\"cloud\":\"Pietro Local VMWare\",\"site\":{\"id\":1},\"type\":\"ts\",\"instanceType\":{\"code\":\"ts\"},\"instanceContext\":\"dev\",\"layout\":{\"id\":${layout},\"code\":\"${layoutcode}\"},\"plan\":{\"id\":116,\"code\":\"container-256\",\"name\":\"256MB Memory, 3GB Storage\"}},\"config\":{\"resourcePoolId\":15,\"poolProviderType\":\"kubernetes\",\"customOptions\":{\"f_tsver\":\"${env.BUILD_NUMBER}\",\"_tsrep\":\"${canary_replicas}\"},\"createUser\":true},\"volumes\":[{\"id\":-1,\"rootVolume\":true,\"name\":\"root\",\"size\":3,\"sizeId\":null,\"storageType\":null,\"datastoreId\":12}],\"ports\":[{\"name\":\"HTTP\",\"port\":${ports},\"lb\":\"HTTP\"}]}" 
                    def cdresponse = httpRequest(url: 'https://192.168.10.104/api/instances', acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', ignoreSslErrors: true, customHeaders: [[name: 'Authorization', value: 'Bearer e125ccff-6c05-4664-a21a-500f98e693cc']], requestBody: "${env.requestBody}", responseHandle: 'STRING', validResponseCodes: '200')
                    def props = readJSON text: cdresponse.content.toString()
                    env.canaryid = props.instance.id
                    
                    println("ID: "+env.canaryid)
                    println("Content: "+cdresponse.content)
                }
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
                // remove Canary Deployment  
                httpRequest(url: 'https://192.168.10.104/api/instances/${env.canaryid}?removeVolumes=on', acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'DELETE', ignoreSslErrors: true, customHeaders: [[name: 'Authorization', value: 'Bearer e125ccff-6c05-4664-a21a-500f98e693cc']], responseHandle: 'STRING', validResponseCodes: '200')
                 
                // Find existing Deployment
                script {
                    def cdresponse = httpRequest(url: 'https://192.168.10.104/api/instances/?instanceType=ts', acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'GET', ignoreSslErrors: true, customHeaders: [[name: 'Authorization', value: 'Bearer e125ccff-6c05-4664-a21a-500f98e693cc']], responseHandle: 'STRING', validResponseCodes: '200')
                    def props = readJSON text: cdresponse.content.toString()
                    env.oldprod = props.instance.id           
                }
                
                
                
                // Deploy New Build To Production
                script {
                    def name = "TSPROD - Build ${env.BUILD_NUMBER}"
                    def layout = 1202
                    def layoutcode = "9a3260e9-ad22-4a40-99c2-3acca466d5dc"
                    def ports = 31443
                    def canary_replicas = 0
                    
                    env.requestBody = "{\"zoneId\":1,\"instance\":{\"name\":\"${name}\",\"cloud\":\"Pietro Local VMWare\",\"site\":{\"id\":1},\"type\":\"ts\",\"instanceType\":{\"code\":\"ts\"},\"instanceContext\":\"dev\",\"layout\":{\"id\":${layout},\"code\":\"${layoutcode}\"},\"plan\":{\"id\":116,\"code\":\"container-256\",\"name\":\"256MB Memory, 3GB Storage\"}},\"config\":{\"resourcePoolId\":15,\"poolProviderType\":\"kubernetes\",\"customOptions\":{\"f_tsver\":\"${env.BUILD_NUMBER}\",\"_tsrep\":\"${canary_replicas}\"},\"createUser\":true},\"volumes\":[{\"id\":-1,\"rootVolume\":true,\"name\":\"root\",\"size\":3,\"sizeId\":null,\"storageType\":null,\"datastoreId\":12}],\"ports\":[{\"name\":\"HTTP\",\"port\":${ports},\"lb\":\"HTTP\"}]}" 
                    def cdresponse = httpRequest(url: 'https://192.168.10.104/api/instances', acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', ignoreSslErrors: true, customHeaders: [[name: 'Authorization', value: 'Bearer e125ccff-6c05-4664-a21a-500f98e693cc']], requestBody: "${env.requestBody}", responseHandle: 'STRING', validResponseCodes: '200')
                    def props = readJSON text: cdresponse.content.toString()
                    env.prodid = props.instance.id
                    
                    println("ID: "+env.prodid)
                    println("Content: "+cdresponse.content)
                }  
                
                // remove old production
                httpRequest(url: 'https://192.168.10.104/api/instances/${env.oldprod}?removeVolumes=on', acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'DELETE', ignoreSslErrors: true, customHeaders: [[name: 'Authorization', value: 'Bearer e125ccff-6c05-4664-a21a-500f98e693cc']], responseHandle: 'STRING', validResponseCodes: '200')
                
            }
        }
    }
}
