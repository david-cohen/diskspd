library identifier: 'EnmotusBuildTools@master', retriever: modernSCM([$class: 'GitSCMSource', credentialsId: 'EnmotusGitAgent', gitTool: 'master_git', remote: 'https://github.com/Enmotus/EnmotusBuildTools.git', traits: [[$class: 'jenkins.plugins.git.traits.BranchDiscoveryTrait'], [$class: 'GitToolSCMSourceTrait', gitTool: 'master_git']]])
pipeline {
  agent {
    node {
      label 'windows_builder'
    	 }
	}
  stages {
    stage('Call Library') {
		steps {
			checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'WindowsBuilder']], gitTool: 'default', submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'EnmotusGitAgent', url: 'https://github.com/Enmotus/WindowsBuilder.git']]]
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
				bat(script: 'C:\\Jenkins\\workspace\\output\\versionstamp.exe file=%WORKSPACE%\\diskspd_CLRclassLibrary\\diskspd_CLRclassLibrary\\app.rc >> lastversion.txt')
				bat(script: 'C:\\Jenkins\\workspace\\output\\versionstamp.exe file=%WORKSPACE%\\diskspd_CLRclassLibrary\\diskspd_CLRclassLibrary\\app.rc Increment >> version.txt')
				env.LAST_VERSION_STAMP = readFile 'lastversion.txt'
				env.VERSION_STAMP = readFile 'version.txt'	
				echo "${VERSION_STAMP}"  
				withCredentials([usernamePassword(credentialsId: 'EnmotusGitAgent', passwordVariable: 'PASSWORD', usernameVariable: 'USER')]) {
					bat(script: 'git commit -a -m "Updated Version Stamp to %VERSION_STAMP%"')
					bat(script: 'git pull --tags https://%USER%:%PASSWORD%@github.com/Enmotus/diskspd.git')
					bat(script: 'git push https://%USER%:%PASSWORD%@github.com/Enmotus/diskspd.git master')
				}
			    }
			    else {
				echo "there are no changes in this build"
				bat(script: 'C:\\Jenkins\\workspace\\output\\versionstamp.exe file=%WORKSPACE%\\diskspd_CLRclassLibrary\\diskspd_CLRclassLibrary\\app.rc  >> version.txt')
				}
				env.VERSION_STAMP = readFile 'version.txt'	
				echo "${VERSION_STAMP}" 
				currentBuild.description = "Version ${VERSION_STAMP}"

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
	  		  script {
				 if(currentBuild.changeSets.size() > 0) {
					withCredentials([usernamePassword(credentialsId: 'EnmotusGitAgent', passwordVariable: 'PASSWORD', usernameVariable: 'USER')]) {
						bat(script: 'git tag -a %VERSION_STAMP% -m "Build %VERSION_STAMP%"')
						bat(script: 'git pull https://%USER%:%PASSWORD%@github.com/Enmotus/diskspd.git')
						bat(script: 'git push https://%USER%:%PASSWORD%@github.com/Enmotus/diskspd.git master --tags' )
				}
			}
		}
      }
    }
	stage('Doxygen') {
      steps {
	  		  script {
				if(currentBuild.changeSets.size() > 0) {
				  bat 'C:\\Jenkins\\workspace\\output\\doxygen.exe %WORKSPACE%\\Doxygen.cfg'
			  publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'docs', reportFiles: 'index.html', reportName: 'Generated Architectural Documents', reportTitles: ''])
				}
			}
      	}
    }
	stage('Release Notes') {
      steps {
		  script {
		   if(currentBuild.changeSets.size() > 0) {
			   dir(path: 'C:\\Jenkins\\workspace\\output') {
				 bat 'mkdir %WORKSPACE%\\ReleaseNotes_v2.0.20a-%VERSION_STAMP%'
				 bat 'mkdir %WORKSPACE%\\ReleaseNotes_%LAST_VERSION_STAMP%-%VERSION_STAMP%'
				 bat 'C:\\Jenkins\\workspace\\output\\releasenotes.exe path=%WORKSPACE% name=Diskspd From=v2.0.20a To=%VERSION_STAMP% repo=https://github.com/Enmotus/diskspd id=2388216 pivotal=cf2870f765936d635fa8bbdd20d81fea >> 5WORKSPACE%\\ReleaseNotes_v2.0.20a-%VERSION_STAMP%\\index.html'
				 bat 'C:\\Jenkins\\workspace\\output\\releasenotes.exe path=%WORKSPACE% name=Diskspd From=%LAST_VERSION_STAMP% To=%VERSION_STAMP% repo=https://github.com/Enmotus/diskspd id=2388216 pivotal=cf2870f765936d635fa8bbdd20d81fea >> %WORKSPACE%\\ReleaseNotes_%LAST_VERSION_STAMP%-%VERSION_STAMP%\\index.html'
				}

				publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: "ReleaseNotes_v2.0.20a-"+env.VERSION_STAMP, reportFiles: 'index.html', reportName: 'Release Notes', reportTitles: 'Release Notes'])
				publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: "ReleaseNotes_"+env.LAST_VERSION_STAMP+"-"+env.VERSION_STAMP, reportFiles: 'index.html', reportName: 'Build Release Notes', reportTitles: 'Build Release Notes'])
			}
		 }
      }
    }
    stage('Deploy to Nexus') {
      steps {
		  script {
		   if(currentBuild.changeSets.size() > 0) {
				bat(script: 'mvn deploy:deploy-file -DgroupId=com.enmotus -DartifactId=diskspd_CLRclassLibrary -Dversion=%VERSION_STAMP% -DgeneratePom=true -Dpackaging=dll -DrepositoryId=enmotus-nexus -Durl=http://23.99.9.34:8081/repository/maven-releases -Dfile=%WORKSPACE%\\diskspd_CLRclassLibrary\\x64\\Release\\diskspd_CLRclassLibrary.dll', returnStatus: true, label: 'mvn deploy:deploy-file')
				createSummary icon:'package.png', text: "<a href=\'http://23.99.9.34:8081/repository/maven-releases/com/enmotus/diskspd_CLRclassLibrary/${VERSION_STAMP}/diskspd_CLRclassLibrary-${VERSION_STAMP}.dll\'>Download DiskSpd CLR Class Library</a>"
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
