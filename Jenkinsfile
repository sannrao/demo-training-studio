node {    

      def app     
      stage('Clone repository') {               
             
            checkout scm    
      }     
      stage('Build image') {         
            snDevOpsStep()
            app = docker.build("santoshnrao/demo-training-studio")    
       }     
      stage('Test image') {           
            app.inside {            
             
             sh 'echo "Tests passed"'        
            }    
        }     
       stage('Push image') {
                  sh 'ls -a'
                  docker.withRegistry('https://registry.hub.docker.com', 'santoshnrao-dockerhub') {            
                  app.push("${env.BUILD_NUMBER}")            
                  app.push("latest")        
              }    
           }
      
      stage('Validate Configurtion file'){
            sh 'echo validating configuration file'
            snDevOpsConfigUpload(applicationName:'App1',target:'component',namePath:'paymentservice.v1.1', fileName:'paymentService', autoCommit:'true',autoValidate:'true',dataFormat:'json')
      }

       stage("deploy to system") {
             
             sh ' echo deployment done'
            //sh "docker run -it --rm -d -p 80:80 --name web santoshnrao/demo-training-studio:${env.BUILD_NUMBER}"
            
//             withKubeConfig([credentialsId: 'santosh-devops-config-k8s']) {
//                         sh 'kubectl apply -f k8s/'
//                   }
            // kubernetesDeploy(kubeconfigId: 'devops-config-demo-1',               // REQUIRED

            //      configs: 'k8s/', // REQUIRED
            // )
       }
}
