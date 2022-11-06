# 

```pipline
def timestr() { 
    script {  
        return sh(script: &#39;date &#43;%Y%m%d%H%M%S&#39;, returnStdout: true).trim()
    }
}

def dockerImage

pipeline{
    agent any
    
    environment {
        time = timestr()
        registry = &#34;xxx.com/payapp-test&#34;
        registryhub = &#34;txhub.xxx.com&#34;
        appName = &#34;api&#34;
    }
    
    options {
        timeout(time: 1, unit: &#39;HOURS&#39;)
        buildDiscarder(logRotator(numToKeepStr: &#39;15&#39;))
        disableConcurrentBuilds()
    }
    
    stages{
        
        stage(&#34;Pull Code&#34;){
            steps{
                git branch: &#39;testing&#39;, credentialsId: &#39;422fb2c7-4d58-440a-98a4-e242b66f3800&#39;, url: &#39;http://gitlab.fgry45iy.com:90/pay/payGateway.git&#39;
            }
        }
        
        stage(&#34;Maven Package&#34;){
            steps{
                withEnv([&#39;PATH&#43;EXTRA=/usr/local/sbin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/apache-maven-3.6.2/bin:/usr/local/maven/bin:/root/bin&#39;]) {
                    sh &#34;mvn package&#34;
                }
            }
        }
        
//         stage(&#39;Building Image&#39;) {
//             steps{
//                 script {
//                     dockerImage = docker.build( registry &#43; &#34;/&#34; &#43; appName &#43; &#34;:$BUILD_NUMBER&#34;)
//                 }
//             }
//         }
        
//         stage(&#39;Push Images To Registry&#39;) {
//             steps {
//                 script {
//                      dockerImage.push()
//                 }
//             }
//         }
        
//         stage(&#39;update&#39;) {
//             steps {
//                 sh &#34;&#34;&#34;
//               curl -k --cert /root/ca/ca.crt --key /root/ca/ca.key  -X PUT -H  &#39;Content-Type: application/yaml&#39; --data &#34;
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
// &#34; https://47.156.81.22:6443/apis/apps/v1/namespaces/pay/deployments/tyapi-api-deploy
//                 &#34;&#34;&#34;
//             }
//         }
    }
    
//     post {
//         cleanup {
//             echo &#39;I have finished, delete dir&#39;
//             deleteDir()
//         }
//     }
}
```
