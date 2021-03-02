pipeline
{
    agent any
    environment
    {
        scannerHome = tool name : 'sonar_scanner_dotnet', type : 'hudson.plugins.sonar.MsBuildSQRunnerInstallation'
        kubeConfigPath = 'C:/Users/meenakshithukral/.kube/config'
        helmConfigPath = 'D:/setups/helm-v3.3.1-windows-amd64/windows-amd64'
    }
    // options
    // {
    //     discardConcurrentBuilds()
        
    // }
    stages
    {
        //stage('checkout')
        //{
          //  steps
            //{
              //  echo 'checkout'
               // scm checkout
            //}
            
        //}

        stage('restore')
        {
            steps
            {
                echo 'restore'
                bat 'dotnet restore'
            }
        }

        stage('Start Sonar')
        {
            steps
            {
                echo 'start sonar analysis'
                withSonarQubeEnv('Test_Sonar'){
                    bat 'dotnet "C:/Program Files (x86)/Jenkins/tools/hudson.plugins.sonar.MsBuildSQRunnerInstallation/sonar_scanner_dotnet/SonarScanner.MSBuild.dll" begin /k:Meenakshi.Nagp.Exam2 /n:Meenakshi.Nagp.Exam2 /v:1.0'
                }
                
            }
        }

        stage('Build')
        {
            steps
            {
                echo 'build'
                bat 'dotnet build'
            }
        }

        stage('End Sonar')
        {
            steps
            {
                echo 'end sonar analysis'
                withSonarQubeEnv('Test_Sonar'){
                    bat 'dotnet "C:/Program Files (x86)/Jenkins/tools/hudson.plugins.sonar.MsBuildSQRunnerInstallation/sonar_scanner_dotnet/SonarScanner.MSBuild.dll" end'
                }
                
            }
        }

        stage('Push Artifacts')
        {
            steps
            {
                echo 'push artifacts'
                bat 'dotnet publish -c release -o ./WebApplication2/meenakshi'
            }
        }

        stage('Docker build')
        {
            steps
            {
                echo 'build image'
                bat 'docker build -t meenakshi_nagp_exam2:feature -f ./WebApplication2/Dockerfile ./WebApplication2'
            }
        }

        stage('Delete Container if Running')
        {
            steps
            {
                echo 'remove container if exists'
                powershell label : '', script : '''
                $containerId = docker container ls -aq --filter=name=meenakshi_dotnet_f
                if($containerId){
                    docker stop $containerId
                    docker rm $containerId
                }
                '''
            }
        }

        stage('Launch Container')
        {
            steps
            {
                echo 'Run Image'
                bat 'docker run --name meenakshi_dotnet_f -d -p 9090:8080 meenakshi_nagp_exam2:feature'
            }
        }

        stage('Helm Deployment')
        {
            steps
            {
                echo 'deployment'
                withEnv(["KUBECONFIG=${kubeConfigPath}","helm=${helmConfigPath}"]){
                    bat 'helm install nagp-exam-feature-deployment nagp-feature-chart --set replicas=1 --set image.repository=meenakshi_nagp_exam2:feature'
                }
            }
        }
    }
}
