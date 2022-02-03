pipeline {
    agent {
        label 'windows-shared'
    }

    environment { 
        PROJECT_NAME 				= 'test-project'
        PROJECT_S3_BUCKET_REGION 		= 'us-east-1'
        PROJECT_S3_BUCKET_NAME 			= 'grammable-travis-watson'
        PROJECT_BUILD_OUTPUT_FILE_NAME 		= 'test.zip'              
        PROJECT_SOLUTION_NAME 			= 'HelloWorld.sln'
	GITHUB_REPO_URL				= 'https://github.com/travis-watson1/jenkinstest2.git'
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
				url: $($ENV:GITHUB_REPO_URL)"
                            ]
                        ]
                    ]
                )
            }
        }
	    
	stage('Restore packages'){
		steps {
			script{
				bat 'C:\\ProgramData\\chocolatey\\lib\\NuGet.CommandLine\\tools\\nuget.exe restore ${PROJECT_SOLUTION_NAME}'
			}
		}
	}

        stage('Building the project') {
            steps {
		    script{
			def msbuild = tool name: 'MSBuild', type: 'hudson.plugins.msbuild.MsBuildInstallation'
         		bat "\"${msbuild}\" ${PROJECT_SOLUTION_NAME}  /p:DeployOnBuild=true /p:DeployDefaultTarget=WebPublish /p:WebPublishMethod=FileSystem /p:SkipInvalidConfigurations=true /t:build /p:Configuration=Release /p:Platform=\"Any CPU\" /p:DeleteExistingFiles=True /p:publishUrl=c:\\inetpub\\wwwroot" 
		    }
	    }
       }

	    
        stage ('zip artifact') {
            steps {
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
