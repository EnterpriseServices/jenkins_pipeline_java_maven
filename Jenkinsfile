#!groovy

               
                def releaseJob = env.JOB_NAME.split('/', 2)[0].contains('RELEASE')
                def buildType = releaseJob ? 'release' : 'snapshot'
                echo "buildType set to $buildType based on job name $env.JOB_NAME"
               	if (buildType == 'release'){ 
               	
               	}
                
                properties([
                       disableConcurrentBuilds(),
                        pipelineTriggers(releaseJob ? [] : [[$class: "GitHubPushTrigger"]]),
                        /*[$class: 'ParametersDefinitionProperty',
                         parameterDefinitions:
                                 [[$class: 'BooleanParameterDefinition',
                                   name: 'CONFIRM_RELEASE',
                                   defaultValue: false,
                                   description: "Check this box to confirm the release."]]] */
              ])
                
               def git_creds = 'GITHUB_CRED_ID'
                
                def buildVersion
                def gitPrevTag
                def gitCreds = [[$class: 'UsernamePasswordMultiBinding',
                                 //credentialsId: '70af3371-6245-4e69-86b6-5808471ca514',
				 credentialsId: git_creds,
                                 usernameVariable: 'GIT_USERNAME',
                                 passwordVariable: 'GIT_PASSWORD']]
                
		def git_user
		def git_password
		node {
		  withCredentials([
		      [$class: 'UsernamePasswordMultiBinding', credentialsId: git_creds, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD'],
		  ]){
		    stage ('SET GIT credentials') {
			echo "GIT_USERNAME::  " + env.GIT_USERNAME
			    git_user = env.GIT_USERNAME
			    git_password = env.GIT_PASSWORD
			    
			    echo "git_user::" + git_user
		      sh """(
			echo "User: ${GIT_USERNAME}"
			echo "Pass: ${GIT_PASSWORD}"
		      )"""
		    }
		  }
		}

                def runMaven = { tasks ->
                    def mvnHome = tool name: 'MAVEN3', type: 'maven'
                    def javaHome = tool name: 'JDK8', type: 'jdk'
                    withEnv(["JAVA_HOME=$javaHome"]) {
                        sh "${mvnHome}/bin/mvn --debug --batch-mode $tasks "
                    }
                }
		
                def projectRepo
                def repoWithCreds = { url ->
                    url.replace(
                            'https://',
                            'https://' + git_user + ':' + git_password + '@')
                }
		def repoName = ""
                //def repoName = { url ->
                  //  url.substring(url.lastIndexOf('/') + 1).replace('.git', '')
                //}
                
                node {
                    stage("Checkout") {
                        //checkout scm
                	echo "GIT_USERNAME  " + env.GIT_USERNAME
                        // Refresh tags, in case something was aborted in a previous run
                        //sh "git tag -l | xargs git tag -d"
                        //sh "git fetch --all"
                
                        projectRepo = sh script: "git ls-remote --get-url",
                                returnStdout: true
                        projectRepo = projectRepo.trim().replace('.git', '')
                        echo "Project URL is $projectRepo"
                    }
                
                    
                
                    
                    stage("Publish") {
                        if (buildType == 'release') {
                            echo "Finding previous tag"
                            try {
                                gitPrevTag = sh script: "git describe --abbrev=0",
                                        returnStdout: true
                                gitPrevTag = gitPrevTag.trim()
                            } catch (Exception e) {
                                gitPrevTag == ''
                            }
				
				
				
                            echo "Tagging the release"
                            def tagMessage = 'Tagging release from Jenkins'
				echo "git config --get remote.origin.url"
				sh "git config --get remote.origin.url"
				//echo "ssh -T git@github.hpe.com"
				//sh "ssh -T git@github.hpe.com"
                            sh "git tag -a -m '$tagMessage' $buildVersion"
				echo "before try block"
                            try {
				    echo "inside try block"
                                withCredentials(gitCreds) {
					echo "inside withCredentials"
					echo "git push --tags $repoWithCreds"+".git"
					echo "git push ${repoWithCreds projectRepo}.git --tags "
				    
                                    	sh "git push ${repoWithCreds projectRepo}.git --tags "
					//sh "git push --tags https://github.hpe.com/satyanarayana-raju/GlobalTaxEngine.git"
					//sh "git push --tags"
					//sh "git push --tags https://satyanarayana.raju@hpe.com:9a099e28697fa364dbe59dbee024c1e8af8fb4ab@github.hpe.com/satyanarayana-raju/GlobalTaxEngine.git"
				
					
					//echo "git push --tags $repoWithCreds"+".git"
                                }
                            }
                            catch (Exception e) {
                                sh "git tag --delete $buildVersion"
                                throw e
                            }
                        }
                    }
                
                   if (releaseJob) {
                        stage("Release notes") {
                            if (gitPrevTag == '') {
                                echo "No previous tags found, cannot do release notes"
                                return
                            }
                            def releaseNotesScript = load 'release-notes.groovy'
                            def templateStr = readFile 'release-notes-template.md'
                            def gitLogFormat = releaseNotesScript.getGitLogFormat()
                            def gitLogStr = sh script:
                                    "git log $gitPrevTag..$buildVersion " +
                                            "$gitLogFormat",
                                    returnStdout: true
                            
                            if (gitLogStr.trim() == '') {
                                echo "Skipping release notes, git log returned nothing"
                                return
                            }
                            gitLogStr = releaseNotesScript.sanitizeGitLogStr(gitLogStr)
                
                            def releaseNotesStr = releaseNotesScript
                                    .generateReleaseNotes(projectRepo,
                                    buildVersion, templateStr, gitLogStr)
                            echo "$releaseNotesStr"
                
                            writeFile text: """{"tag_name": "$buildVersion", "body": 
                "$releaseNotesStr"}""",
                                    file: "release-notes-${buildVersion}.json"
                
                            def projectName = repoName projectRepo
                            sleep 5
                            withCredentials(gitCreds) {
                                sh """curl --user "$env.GIT_USERNAME:$env.GIT_PASSWORD" \
                --data @release-notes-${buildVersion}.json \
                https://github.hpe.com/api/v3/repos/prp/$projectName/releases"""
                            }
                        }
                   }
                } 
