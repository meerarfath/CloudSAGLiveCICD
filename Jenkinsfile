def ISSolName
def UMSolName
pipeline{
    agent{
        node{
            label 'DesignerWin'
            customWorkspace 'C:/CloudTransformation/SAGLiveWorkspace'
            }
        }
    environment {
        userHome="C: + %homepath%"
    }
    stages{
            stage('checkout GITAssets'){
                steps{
                    script{

                        echo 'checkout GITAssets'
                        if (!fileExists('Assets')) {
                            bat 'mkdir Assets'
                            dir('C:/CloudTransformation/SAGLiveWorkspace/Assets'){ 
                                echo 'cloning the project'                              
                                bat 'git clone --recurse-submodules https://github.com/AbhishekGupta1506/CloudSAGLiveAssets.git'
                        }
                        } else {
                            echo 'Assets directory exist'
                            dir('C:/CloudTransformation/SAGLiveWorkspace/Assets/CloudSAGLiveAssets'){                            
                                echo 'pulling the updates'                              
                                bat 'git pull'
                                bat 'git submodule update'
                            }
                        }                    
                    }                                                        
                }
            }
        stage('Install ABE'){
            steps{
                script{
                    dir('C:/CloudTransformation/SAGLiveWorkspace/script'){
                        bat 'sagabeinstall.bat'
                    }
                    
                }
            }
        }
        stage('Build'){
            steps{
                script{
                    bat 'cp C:/CloudTransformation/SAGLiveWorkspace/script/build.properties C:/SoftwareAG103ABE/common/AssetBuildEnvironment/master_build'
                    dir('C:/SoftwareAG103ABE/common/AssetBuildEnvironment/bin'){
                        bat 'build.bat'
                    }
                }
            }

        }  
        stage('Deploy'){
            //update this step to deploy to Cloud LAR/GIT once it is stable
            steps{
               echo 'deploy assets cloud LAR'
               script{
                   if (fileExists('CloudGIT')) {
                       bat 'rd /s /q CloudGIT'
                       //bat 'rd /s /q CloudGIT@tmp'
                   }
                   bat 'mkdir CloudGIT'
                   dir('C:/CloudTransformation/SAGLiveWorkspace/CloudGIT'){
                       for (int i = 1; i <= 3; i++) {
                            bat "echo print stage :: $i-$i"  

                            if(i <= 2){
                                ISSolName = "stage00-Sol${i}-Sol${i}IS"
                                if( i != 1){
                                    UMSolName = "stage00-Sol${i}-Sol${i}UM"
                                }
                            }
                            else{
                                ISSolName = "stage00-Sol${i}-Sol${i}IS1"
                                UMSolName = "stage00-Sol${i}-Sol${i}UM"
                            }                           
                            if(i != 1){
                                echo 'pushing the UM configuration'
                                bat "echo ${UMSolName}"
                                bat "mkdir ${UMSolName}"
                                dir("C:/CloudTransformation/SAGLiveWorkspace/CloudGIT/${UMSolName}"){
                                    bat 'git config --global http.sslVerify false'
                                    bat 'git config --global credential.helper cache'
                                    bat 'git config --global push.default simple' 
                                    checkout([ $class: 'GitSCM', branches: [[name: '*/master']], extensions: [ [$class: 'CloneOption', noTags: true, reference: '', shallow: true] ], 
                                    submoduleCfg: [], userRemoteConfigs: [[ credentialsId: 'cloudUsernamePassword', 
                                    url: "https://siqa1.saglive.com/integration/rest/internal/wmic-git/${UMSolName}"]]])
                                    if (!fileExists('CC')) {
                                        bat 'mkdir CC'
                                    }
                                    echo 'copy the UM build configuration'
                                    dir("C:/CloudTransformation/SAGLiveWorkspace/CloudGIT/${UMSolName}/CC"){
                                        bat 'cp C:/CloudTransformation/SAGLiveWorkspace/CloudAssetsBuild/CC/localhost-Universal-Messaging-umserver* .'	
                                    }                                       
                                    bat 'git status'
                                    bat 'git remote show origin'
                                    bat 'git show-ref'
                                    bat 'git add .'
                                    bat 'git commit -am "pushing the latest UM build"' 
                                    echo "pushing assets/config to ${UMSolName}" 
                                    bat 'git push origin HEAD:master'  
                                }
                            }

                            dir("C:/CloudTransformation/SAGLiveWorkspace/CloudGIT/${ISSolName}"){
                                bat "echo ${ISSolName}"                            
                                bat "mkdir ${ISSolName}"
                                echo 'pushing the IS Assets/configuration'
                                bat 'git config --global http.sslVerify false'
                                bat 'git config --global credential.helper cache'
                                bat 'git config --global push.default simple' 
                                checkout([ $class: 'GitSCM', branches: [[name: '*/master']], extensions: [ [$class: 'CloneOption', noTags: true, reference: '', shallow: true] ], 
                                submoduleCfg: [], userRemoteConfigs: [[ credentialsId: 'cloudUsernamePassword', 
                                url: "https://siqa1.saglive.com/integration/rest/internal/wmic-git/${ISSolName}"]]])
                                if (!fileExists('IS')) {
                                    bat 'mkdir IS'
                                }
                                echo 'copy the IS build assets'
                                dir("C:/CloudTransformation/SAGLiveWorkspace/CloudGIT/${ISSolName}/IS"){
                                    bat 'cp -r C:/CloudTransformation/SAGLiveWorkspace/CloudAssetsBuild/IS/. .'
                                }
                                if (!fileExists('CC')) {
                                    bat 'mkdir CC'
                                }
                                echo 'copy the IS build configuration'
                                dir("C:/CloudTransformation/SAGLiveWorkspace/CloudGIT/${ISSolName}/CC"){
                                    bat 'cp C:/CloudTransformation/SAGLiveWorkspace/CloudAssetsBuild/CC/localhost-OSGI-IS_default* .'	
                                }                                       
                                bat 'git status'
                                bat 'git remote show origin'
                                bat 'git show-ref'
                                bat 'git add .'
                                bat 'git commit -am "pushing the latest IS build"' 
                                echo "pushing assets/config to ${ISSolName}" 
                                bat 'git push origin HEAD:master'  
                            }
                        }
                   }
               }               
            }
        }   
        stage('Test'){
            steps{
                script{
                    for(int i=0;i<10;i++){

                        try{
                            def response = httpRequest authentication: 'cloudUsernamePassword', url: "https://siqa1.saglive.com/integration/clouddeployment/service/development/Sol2/Sol2IS/invoke/umassets.jmsMessaging.UMQueue.mixedQueue.services.publisher:publishservice"
                            echo "Status: ${response.status}"
                        }
                        catch(exception e){
                            echo "Inside Catch: HTTP request failed"
                        }
                        
                        def responseStatus = ${response.status}
                        if(responseStatus == 200){
                            echo "Status: passed with status ${response.status}"
                            break
                        } 
                        else if(responseStatus == 502){
                            echo "Status: failed with status ${response.status}. Serer not available, its restarting"
                            echo "will retry after 10 sec"
                            sleep 10
                        }
                        else{
                            echo "Status: failed with status ${response.status}. Serer not working"
                            break
                        }
                    }
                    

                }                                                        
            }
        }  
    }
}