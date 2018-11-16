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
        stage('CheckOut Assets'){
            steps{
                script{
                    if (!fileExists('Assets')) {
                        bat 'mkdir Assets'
                        dir('C:/CloudTransformation/SAGLiveWorkspace/Assets'){ 
                            echo 'cloning the project'                              
                            bat 'git clone --recurse-submodules https://github.com/AbhishekGupta1506/CloudSAGLiveAssets.git'
                    }
                    } else {
                        echo 'Assets directory exist'
                        dir('C:/CloudTransformation/SAGLiveWorkspace/Assets/CloudSAGLiveAssets'){
                            echo 'pulling the update'                              
                            bat 'git pull'
                            bat 'git submodule update'
                        }
                    }                    
                }                                                        
            }
        }
        stage('Build'){
            steps{
                script{
                    dir('C:/SoftwareAG103/common/AssetBuildEnvironment/bin'){
                        bat 'build.bat'
                    }
                }
            }

        }       
    }
}