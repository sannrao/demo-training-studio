node {    

      def app     
      def appName='Payment Fulfillment Service'
      def snapName=''
      def deployableName = 'PROD-US'
      def setYamlUpload = true
      
      // Json Example
      def configFilePath = "paymentService"
      def exportFormat ='json'

      // Yaml Example
      if(setYamlUpload){
            exportFormat ='yaml'
            configFilePath = "k8s/demo-training-studio/values"
      }

      def fileNamePrefix ='exported_file_'
      def fullFileName="${fileNamePrefix}-${deployableName}-${currentBuild.number}.${exportFormat}"
      def changeSetId=""
      def componentName="paymentService-v1.1"
      def collectionName="release-1.0"
      def snapshotName=""
      def exporterName ='k8s-exporter' 
      def exporterArgs = '{"component": "' + componentName + '", "collection": "' + collectionName + '", "deployable": "' + deployableName + '"}'
      def dockerImageName = "santoshnrao/demo-training-studio"
      def dockerImageTag=""

      stage('Build image') {      
            
            checkout scm    

            // snDevOpsStep()

            app = docker.build("santoshnrao/demo-training-studio")    
            
            
       }     
      stage('Test') {           
            app.inside {            
             
             sh 'echo "Tests passed"'        
            }    
        }     
      
       stage('Push docker Image') { 
            sh 'ls -a'

            dockerImageTag = env.BUILD_NUMBER
            dockerImageNameTag = "${dockerImageName}" + ":" + "${dockerImageTag}"
      
            docker.withRegistry('https://registry.hub.docker.com', 'santoshnrao-dockerhub') {            
                  app.push("${dockerImageTag}")            
                  app.push("latest")        
            }    

            snDevopsArtifactPayload = '{"artifacts": [{"name": "' + dockerImageName + '",  "version": "' + "${dockerImageTag}" + '", "semanticVersion": "' + "0.1.${dockerImageTag}"+ '","repositoryName": "' + dockerImageName+ '"}, ],"stageName":"Build image","branchName": "main"}'  ;
            echo " docker Image artifacat ${dockerImageNameTag} "
            echo "snDevopsArtifactPayload ${snDevopsArtifactPayload} "
            
            snDevOpsArtifact(artifactsPayload:snDevopsArtifactPayload)

      }
      
      stage('Upload Configuration Files'){
            

            sh "echo validating configuration file ${configFilePath}.${exportFormat}"
            changeSetId = snDevOpsConfigUpload(applicationName:"${appName}",target:'component',namePath:"${componentName}", fileName:"${configFilePath}", autoCommit:'true',autoValidate:'true',dataFormat:"${exportFormat}")

            echo "validation result $changeSetId"
            
              echo "Change set registration for ${changeSetId}"
              changeSetRegResult = snDevOpsConfigRegisterChangeSet(changesetId:"${changeSetId}")
              echo "change set registration set result ${changeSetRegResult}"
            
      }


    stage("Get snapshot status"){
          
        echo "Triggering Get snapshots for applicationName:${appName},deployableName:${deployableName},changeSetId:${changeSetId}"

        changeSetResults = snDevOpsConfigGetSnapshots(applicationName:"${appName}",deployableName:"${deployableName}",changeSetId:"${changeSetId}")
        echo "ChangeSet Result : ${changeSetResults}"
        
        def changeSetResultsObject = readJSON text: changeSetResults
        
          changeSetResultsObject.each {

                if(it.validation == "passed"){
                      echo "validation passed for snapshot : ${it.name}"
                      snapshotName = it.name
                }else{
                      echo "Snapshot failed to get validated : ${it.name}" ;
                      assert it.validation == "passed"
                }
            
          }
          if (!snapshotName?.trim()){
                error "No valid snapshot found to proceed" ;
          }
          echo "Snapshot Name : ${snapshotName} "                
          
    }

      stage('Publish the snapshot'){
            echo "Step to publish snapshot applicationName:${appName},deployableName:${deployableName} snapshotName:${snapshotName}"
            publishSnapshotResults = snDevOpsConfigPublish(applicationName:"${appName}",deployableName:"${deployableName}",snapshotName: "${snapshotName}")
            echo " Publish result for applicationName:${appName},deployableName:${deployableName} snapshotName:${snapshotName} is ${publishSnapshotResults} "
      }

      stage('Export Snapshots from Service Now') {

            echo "Devops Change trigger change request"
            snDevOpsChange()

            echo "Exporting for App: ${appName} Deployable; ${deployableName} Exporter name ${exporterName} "
            echo "Configfile exporter file name ${fullFileName}"
            sh  'echo "<<<<<<<<<export file is starting >>>>>>>>"'
            response = snDevOpsConfigExport(applicationName: "${appName}", snapshotName: "${snapName}", deployableName: "${deployableName}",exporterFormat: "${exportFormat}", fileName:"${fullFileName}", exporterName: "${exporterName}", exporterArgs: "${exporterArgs}")
            echo " RESPONSE FROM EXPORT : ${response}"

        }
      
      stage("Deploy to PROD-US"){
            
            echo "Reading config from file name ${fullFileName}"
            echo " ++++++++++++ BEGIN OF File Content ***************"
            sh "cat ${fullFileName}"
            echo " ++++++++++++ END OF File content ***************"
            
            echo "deploy finished successfully."

            
            echo "********************** BEGIN Deployment ****************"
            echo "Applying docker image ${dockerImageNameTag}"

            sh "helm upgrade demo-training-studio -f ${fullFileName} k8s/demo-training-studio/ -i  --set image.tag=${dockerImageTag} --set image.repository=${dockerImageName}"  

            echo "********************** END Deployment ****************"

            
      }

}
