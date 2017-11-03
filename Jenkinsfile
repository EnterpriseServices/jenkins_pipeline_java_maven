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
                
               def git_creds = 'GIT_CRED_ID2'
                
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
                        checkout scm
                	echo "GIT_USERNAME  " + env.GIT_USERNAME
                        // Refresh tags, in case something was aborted in a previous run
                        sh "git tag -l | xargs git tag -d"
                        sh "git fetch --all"
                
                        projectRepo = sh script: "git ls-remote --get-url",
                                returnStdout: true
                        projectRepo = projectRepo.trim().replace('.git', '')
                        echo "Project URL is $projectRepo"
                    }
                
                    stage("Versioning") {
                        def mavenPom = readMavenPom file: 'pom.xml'
                        echo "POM version is ${mavenPom.version}"
                
                        buildVersion = mavenPom.version.trim().replace('-SNAPSHOT', '')
			    echo "buildVersion::::" + buildVersion
			    buildVersion = ""
			    
                
                        if (env.BRANCH_NAME != 'master') {
				echo "env.BRANCH_NAME::" + env.BRANCH_NAME
                            buildVersion += '-' + env.BRANCH_NAME
				echo "buildVersion::" + buildVersion
				buildVersion = ""
                        }
                
                        if (buildType == 'snapshot') {
                            buildVersion += '-SNAPSHOT'
                        } else if (buildType == 'release') {
                            echo "Determining existing tag versions of $buildVersion"
                            //def tags = sh script: "git tag --list $buildVersion*",
				def tags = sh script: "git tag --list $buildVersion",
                                    returnStdout: true
                            echo tags
                            existingTagVersions = [0]
                            for (String tag in tags.split('\\s')) {
                                tag = tag.trim()
				echo "Tag name: $tag"
                                if (tag == '') continue
                                existingTagVersions <<
                                        (int) ((tag.tokenize('-')[-1]).toInteger())
                            }
                            existingVersionsCnt = existingTagVersions.size()
			    echo "existingVersionsCnt $existingVersionsCnt"
                            maxExistingVersion = String.valueOf(existingTagVersions.max())
			    //maxExistingVersion = String.valueOf(1)
			    echo "maxExistingVersion $maxExistingVersion"
                            thisVersion = String.valueOf(existingTagVersions.max() + 1)
                            if (existingVersionsCnt > 1) {
                                echo "Found ${existingVersionsCnt - 1} existing tag versions"
                                echo "Highest tag version found is $maxExistingVersion"
                            } else {
                                echo "No existing tags versions found"
                            }
                            //buildVersion += '-' + thisVersion
				buildVersion = thisVersion
                        }
                        echo "buildNumber version $buildVersion"
                    }
                
                    /*stage("Build") {
                        echo "Setting POM version numbers to $buildVersion"
                        try {
                            runMaven "versions:set -DnewVersion=$buildVersion"
				echo "after runMaven"
                            if (releaseJob) {
				    echo "inside releaseJob"
                            	echo "releaseJob=$releaseJob; --update-snapshots clean deploy"
                               runMaven "--update-snapshots clean deploy"
				    echo "after runMaven clean deploy"
                            } else {
				    echo "inside else releaseJob"
                                echo "releaseJob=$releaseJob; only deploy"
                               runMaven "clean deploy"
				    echo "after runMaven only deploy"
                               //runMaven "deploy"
                            }
                        } 
                         catch(e)
                         {
                          echo "Build failure:maven build failed $e"
                          throw e
                         }
                        finally {
                            echo "Reverting POM version numbers"
                            runMaven "versions:revert"
                        }
                    }*/
					
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
                            //sh "git tag -a -m '$tagMessage' $buildVersion"
				echo "before try block"
                            try {
				    echo "inside try block"
				    
				    withCredentials([usernamePassword(credentialsId: git_creds, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
    					sh "git tag -a -m '$tagMessage' $buildVersion"
    					sh('git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.hpe.com/satyanarayana-raju/GlobalTaxEngine.git --tags')
				    }
				echo "after with Creds"
				    
                                withCredentials(gitCreds) {
					echo "inside withCredentials"
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
