pipeline {
    agent {
        label 'windows-shared'
    }

    environment { 
        PROJECT_NAME 					= 'test-project'
        PROJECT_S3_BUCKET_REGION 		= 'us-east-1'
        PROJECT_S3_BUCKET_NAME 			= 'grammable-travis-watson'
        PROJECT_BUILD_OUTPUT_FILE_NAME 	= 'test.zip'              
        PROJECT_SOLUTION_NAME 			= 'HelloWorld.sln'
		AUTO_SCALING_GROUP_NAME			= 'terraform-asg-test-app'    
		ASG_MIN_SIZE					= 1
		ASG_MAX_SIZE					= 2
		ASG_DESIRED_SIZE				= 2
    }

    stages {
        stage('Cloning the project repository from Github') {
            steps {
                checkout(
                    [
                        $class: 'GitSCM', 
                        branches: [[name: '*/master']], 
                        doGenerateSubmoduleConfigurations: false, 
                        extensions: [[$class: 'CleanBeforeCheckout']], 
                        submoduleCfg: [], 
                        userRemoteConfigs: [
                            [
                                credentialsId: 'git', 
                                url: 'https://github.com/travis-watson1/jenkinstest2.git'
                            ]
                        ]
                    ]
                )
            }
        }

        stage('Building the project') {
            steps {
		    script{
			def msbuild = tool name: 'MSBuild', type: 'hudson.plugins.msbuild.MsBuildInstallation'
			//bat "\"${msbuild}\" ${PROJECT_SOLUTION_NAME}  /t:Restore /p:DeployOnBuild=true /p:PublishProfile=FolderProfile /p:Configuration=Release /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}" 
         		bat "\"${msbuild}\" ${PROJECT_SOLUTION_NAME}  /t:Restore" 
		    }

                
		    
		    
		    // bat 'nuget restore ${PROJECT_SOLUTION_NAME}'
		        // bat "\"${tool 'MSBuild_VS2019community'}\\msbuild.exe\" ${PROJECT_SOLUTION_NAME} /p:DeployOnBuild=true /p:DeployDefaultTarget=WebPublish /p:WebPublishMethod=FileSystem /p:SkipInvalidConfigurations=true /t:build /p:Configuration=Release /p:PublishProfile=FolderProfile /p:Configuration=Release /p:DeployDefaultTarget=WebPublish /p:WebPublishMethod=FileSystem /p:SkipInvalidConfigurations=true /t:build /p:Configuration=Release /p:Platform=\"Any CPU\" /p:DeleteExistingFiles=True /p:publishUrl=c:\\inetpub\\wwwroot" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
           }
       }
				stage('Build') {
    					steps {
    					    bat "\"${tool 'MSBuild'}\" HelloWorld.sln /p:DeployOnBuild=true /p:DeployDefaultTarget=WebPublish /p:WebPublishMethod=FileSystem /p:SkipInvalidConfigurations=true /t:build /p:Configuration=Release /p:Platform=\"Any CPU\" /p:DeleteExistingFiles=True /p:publishUrl=c:\\inetpub\\wwwroot"
    					}
				}

	    
        stage ('zip artifact') {
            steps {
                //bat 'mkdir archive'
                //bat 'echo test > archive/test.txt'
                script{ zip zipFile: 'test.zip', archive: false, dir: 'helloworld' }
		archiveArtifacts artifacts: 'test.zip', fingerprint: true
            }
        }

        stage('Uploading application package to s3') {
            steps {
                withCredentials([[
                $class: 'AmazonWebServicesCredentialsBinding',
                accessKeyVariable: 'AWS_ACCESS_KEY_ID', // dev credentials
                credentialsId: 'AWSCRED',
                secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]){
                    powershell '''
                        Import-Module AWSPowerShell
                        Set-AWSCredentials -AccessKey "$($ENV:AWS_ACCESS_KEY_ID)" -SecretKey "$($ENV:AWS_SECRET_ACCESS_KEY)" -StoreAs "$($ENV:PROJECT_NAME)"
                        Write-S3Object -BucketName "$($ENV:PROJECT_S3_BUCKET_NAME)" -File "$($ENV:PROJECT_BUILD_OUTPUT_FILE_NAME)" -Region "$($ENV:PROJECT_S3_BUCKET_REGION)" -ProfileName "$($ENV:PROJECT_NAME)"
                    '''
                }
            }
        }
    }
}
