library identifier: 'EnmotusBuildTools@master', retriever: modernSCM([$class: 'GitSCMSource', credentialsId: 'enmotusdavecohen', gitTool: 'master_git', remote: 'https://github.com/Enmotus-Dave-Cohen/EnmotusBuildTools.git', traits: [[$class: 'jenkins.plugins.git.traits.BranchDiscoveryTrait'], [$class: 'GitToolSCMSourceTrait', gitTool: 'master_git']]])
pipeline {
  agent {
    node {
      label 'windows_builder'
    	 }
	}
  stages {
    stage('Call Library') {
		steps {
			checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'WindowsBuilder']], gitTool: 'default', submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'enmotusdavecohen', url: 'https://github.com/Enmotus-Dave-Cohen/WindowsBuilder.git']]]
			script {
			InstallWindowsBuilderDependencies.call()
			  }
			}
		}
	stage('Update Version'){
		steps {
			script {
                    if(currentBuild.changeSets.size() > 0) {
                        echo "version number needs to be updated"
						bat(script: 'C:\\Jenkins\\workspace\\output\\versionstamp-1.0.0.exe file=%WORKSPACE%\\diskspd_CLRclassLibrary\\diskspd_CLRclassLibrary\\app.rc Increment >> version.txt')
						withCredentials([usernamePassword(credentialsId: 'enmotusdavecohen', passwordVariable: 'PASSWORD', usernameVariable: 'USER')]) {
							bat(script: 'git commit -a -m "Updated Version Stamp to ${VERSION_STAMP}"')
							bat(script: 'git push https://${USER}:${PASSWORD}@github.com/Enmotus-Dave-Cohen/diskspd.git master')
						}
                    }
                    else {
                        echo "there are no changes in this build"
						bat(script: 'C:\\Jenkins\\workspace\\output\\versionstamp-1.0.0.exe file=%WORKSPACE%\\diskspd_CLRclassLibrary\\diskspd_CLRclassLibrary\\app.rc  >> version.txt')
						}

						env.VERSION_STAMP = readFile 'version.txt'	
						echo "${VERSION_STAMP}"     
					}
			}
		}
	  stage('Build DiskSpd Class Library') {
      steps {
			bat(script: 'PATH = %PATH%;"C:\\Program Files (x86)\\Microsoft SDKs\\Windows\\v10.0A\\bin\\NETFX 4.8 Tools"', label: 'Set path for xsd.exe')
			bat(script: 'setx /m Path "%Path%;C:\\Program Files (x86)\\Microsoft SDKs\\Windows\\v10.0A\\bin\\NETFX 4.8 Tools"', label: 'Setting Path')
			bat '"C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\BuildTools\\MSBuild\\Current\\Bin\\msbuild" /t:Build /p:Configuration=Release /p:Platform="x64" %WORKSPACE%\\diskspd_CLRclassLibrary\\diskspd_CLRclassLibrary.sln'
		}
	  }
	  stage('Tag Build') {
      steps {
		withCredentials([usernamePassword(credentialsId: 'enmotusdavecohen', passwordVariable: 'PASSWORD', usernameVariable: 'USER')]) {
			bat(script: 'git tag -a v${VERSION_STAMP} -m "Build ${VERSION_STAMP}"')
			bat(script: 'git push origin v${VERSION_STAMP}' )
		}
      }
    }
    stage('Deploy to Nexus') {
      steps {
		  script {
		   if(currentBuild.changeSets.size() > 0) {
				bat(script: 'mvn deploy:deploy-file -DgroupId=com.enmotus -DartifactId=diskspd_CLRclassLibrary -Dversion=%VERSION_STAMP% -DgeneratePom=true -Dpackaging=dll -DrepositoryId=enmotus-nexus -Durl=http://23.99.9.34:8081/repository/maven-releases -Dfile=%WORKSPACE%\\diskspd_CLRclassLibrary\\x64\\Release\\diskspd_CLRclassLibrary.dll', returnStatus: true, label: 'mvn deploy:deploy-file')
			}
			else
			{
				echo "Not a new version, No need to push to Nexus."
			}
		  }
	  }
    }
  }
}
