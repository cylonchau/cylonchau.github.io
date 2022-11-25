# 

```pipline
def timestr() { 
    script {  
        return sh(script: 'date +%Y%m%d%H%M%S', returnStdout: true).trim()
    }
}

def dockerImage

pipeline{
    agent any
    
    environment {
        time = timestr()
        registry = "xxx.com/payapp-test"
        registryhub = "txhub.xxx.com"
        appName = "api"
    }
    
    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '15'))
        disableConcurrentBuilds()
    }
    
    stages{
        
        stage("Pull Code"){
            steps{
                git branch: 'testing', credentialsId: '422fb2c7-4d58-440a-98a4-e242b66f3800', url: 'http://gitlab.fgry45iy.com:90/pay/payGateway.git'
            }
        }
        
        stage("Maven Package"){
            steps{
                withEnv(['PATH+EXTRA=/usr/local/sbin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/apache-maven-3.6.2/bin:/usr/local/maven/bin:/root/bin']) {
                    sh "mvn package"
                }
            }
        }
        
//         stage('Building Image') {
//             steps{
//                 script {
//                     dockerImage = docker.build( registry + "/" + appName + ":$BUILD_NUMBER")
//                 }
//             }
//         }
        
//         stage('Push Images To Registry') {
//             steps {
//                 script {
//                      dockerImage.push()
//                 }
//             }
//         }
        
//         stage('update') {
//             steps {
//                 sh """
//               curl -k --cert /root/ca/ca.crt --key /root/ca/ca.key  -X PUT -H  'Content-Type: application/yaml' --data "
// apiVersion: apps/v1
// kind: Deployment
// metadata:
//   name: tyapi-api-deploy
//   namespace: pay
// spec:
//   replicas: 2
//   selector:
//     matchLabels:
//       app: pay-api
//   template:
//     metadata:
//       labels:
//         app: pay-api
//     spec:
//       containers:
//       - name: pay-jv2-api
//         image: txhub.99xyp.com/payapp-test/api:$BUILD_NUMBER
//         ports:
//         - name: payapi
//           containerPort: 8081
// " https://47.156.81.22:6443/apis/apps/v1/namespaces/pay/deployments/tyapi-api-deploy
//                 """
//             }
//         }
    }
    
//     post {
//         cleanup {
//             echo 'I have finished, delete dir'
//             deleteDir()
//         }
//     }
}
```
