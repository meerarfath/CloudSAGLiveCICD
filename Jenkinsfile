pipeline{
    agent{
        node{
            label 'DesignerWin'
            customWorkspace 'C:/CloudTransformation/SAGLiveWorkspace'
            }
        }
    stages{
        stage('CheckOut'){
            parallel{
                    stage('CheckOut Assets'){
                        steps{
                            if (!fileExists('Assets')) {
                                sh 'mkdir Assets'
                                echo 'Assets directory created under ${customWorkspace}'
                            } else {
                                echo 'Assets directory exist'
                            }
                            dir('C:/CloudTransformation/SAGLiveWorkspace/Assets'){
                                
                                sh 'git clone --recursive https://github.com/AbhishekGupta1506/CloudSAGLiveAssets.git'
                            }
                            
                            }
                    }
                    stage('CheckOut Config'){
                        steps{
                            if (!fileExists('Config')) {
                                sh 'mkdir Config'
                                echo 'Config directory created under ${customWorkspace}'
                            } else {
                                echo 'Config directory exist'
                            }
                            dir('C:/CloudTransformation/SAGLiveWorkspace/Config'){
                             sh 'git clone --recursive https://github.com/AbhishekGupta1506/CloudSAGLiveConfig.git'
                            }
                        }
                    }
                }
        }        
    }
}