pipeline 
{
	agent any
    
	environment {
		// coorindate for check runs
		checkrun_repo_commit = "${ghprbGhRepository}/${ghprbActualCommit}"
	}
       
	stages { 
	
		stage('Checkrun update') 
		{
		
			steps {
				
				echo("Building check run coordinate: ")
				echo("ghprbGhRepository = ${ghprbGhRepository}")
				echo("ghprbActualCommit = ${ghprbActualCommit}")
				echo("checkrun_repo_commit = ${checkrun_repo_commit}")
			
				build(job: 'github-commit-checkrun',
					parameters:
					[
						string(name: 'checkrun_repo_commit', value: "${checkrun_repo_commit}"), 
						string(name: 'src_Job_id', value: "${env.JOB_NAME}/${env.BUILD_NUMBER}"),
						string(name: 'src_details_url', value: "${env.BUILD_URL}"),
						string(name: 'checkrun_status', value: "in_progress")
					],
					wait: false, propagate: false)
			} // steps
		} // stage('Checkrun update') 
		
		stage('Initialize') 
		{
			
            
			steps {
				timestamps {
					ansiColor('xterm') {
						
						dir('online_distribution') {
							deleteDir()
						}			
						dir('qa_html') {
							deleteDir()
						}			
						dir('report') {
						 	deleteDir()
						}
					
						sh('hostname')
						sh('pwd')
						sh('env')
						sh('ls -lvhc')

												
						script
						{
							currentBuild.displayName = "${env.BUILD_NUMBER} - ${sha1}"
							
							if (params.upstream_build_description)
							{
								echo ("Override build descriiption with ${upstream_build_description}");
    						currentBuild.description = "${upstream_build_description}"
							}
						}
					}
				}
			}
		}
		
		stage('ContainerCheck') 
		{
			
            
			steps {
				timestamps {
					ansiColor('xterm') {
						
						dir('utilities/jenkins/built-test/') {
							
							sh('$singularity_exec_sphenix tcsh -f singularity-check.sh')
						
						}
					}
				}
			}
		}

		stage('Git Checkout')
		{
			
			steps 
			{
				timestamps { 
					ansiColor('xterm') {
						
						dir('online_distribution') {
							// git credentialsId: 'sPHENIX-bot', url: 'https://github.com/sPHENIX-Collaboration/online_distribution.git'
							
							checkout(
							   [
				            $class: 'GitSCM',
				            extensions: [               
				                [$class: 'CleanCheckout'],     
				                
				                [
				                $class: 'PreBuildMerge',
				                options: [
				                    mergeRemote: 'origin',
				                    mergeTarget: 'master'
				                    ]
				                ],
				            ],
				            branches: [
				                [name: "${sha1}"]
				            ], 
				            userRemoteConfigs: 
				            [[
				                credentialsId: 'sPHENIX-bot', 
				                url: 'https://github.com/${ghprbGhRepository}.git', // https://github.com/sPHENIX-Collaboration/online_distribution.git
				                refspec: ('+refs/pull/*:refs/remotes/origin/pr/* +refs/heads/master:refs/remotes/origin/master'), 
				                branch: ('*')
				            ]]
				        ] //checkout
					    )// checkout
	    
	
						}//						dir('online_distribution') {
						

					}//					ansiColor('xterm') {
					
				}//				timestamps { 
				
			}//			steps 
			
		}//stage('SCM Checkout')
		
		// hold this until jenkins supports nested parallel
		stage('Build')
		{
			parallel {
			
				stage('cpp-check')
				{
					when {
    			 	// case insensitive regular expression for truthy values
						expression { return run_cppcheck ==~ /(?i)(Y|YES|T|TRUE|ON|RUN)/ }
					}
					steps 
					{
						echo ("starting cpp-check with run_cppcheck = ${run_cppcheck}")
		
						script
						{
				   		def built = build(job: 'cpp-check-online_distribution',
			    			parameters:
			    			[
							string(name: 'checkrun_repo_commit', value: "${checkrun_repo_commit}"), 
							string(name: 'sha_online_distribution', value: "${sha1}"), 
							string(name: 'git_url_online_distribution', value: "https://github.com/${ghprbGhRepository}.git"), 
				    			string(name: 'upstream_build_description', value: "${currentBuild.description}"),
					    		string(name: 'ghprbPullLink', value: "${ghprbPullLink}")
				    		],
			    			wait: true, propagate: false)
			    			
						copyArtifacts(projectName: 'cpp-check-online_distribution', filter: 'report/*', selector: specific("${built.number}"));
		    		
						if ("${built.result}" != 'SUCCESS')
						{
						   error('Build New FAIL by cppcheck')
    						}
					}
		   		}
				}// Stage - cpp check

			} // parallel {
		}//stage('Build')
		
	}//stages
		
	post {
		always{
			// archiveArtifacts artifacts: 'build/new/rebuild.log'
			
			dir('report')
			{
				sh('ls -lvhc')
						
				script
				{
					
    			echo("start report building ...");
    			sh ('pwd');
				
			def report_content = """
## Build & test report 
Report for [commit ${ghprbActualCommit}](${ghprbPullLink}/commits/${ghprbActualCommit}):"""
				
			if ("${currentBuild.currentResult}" == "FAILURE")
			{
  				report_content = """${report_content}
[![Jenkins on fire](https://raw.githubusercontent.com/sPHENIX-Collaboration/utilities/master/jenkins/material/build_failded.png)](${env.BUILD_URL})"""
			}
			if ("${currentBuild.currentResult}" == "ABORT")
			{
  				report_content = """${report_content}
[![Jenkins aborted](https://raw.githubusercontent.com/sPHENIX-Collaboration/utilities/master/jenkins/material/jenkins_logo_snow-128p.png)](${env.BUILD_URL})"""
			}
			if ("${currentBuild.currentResult}" == "SUCCESS")
			{
  				report_content = """${report_content}
[![Jenkins passed](https://raw.githubusercontent.com/sPHENIX-Collaboration/utilities/master/jenkins/material/jenkins_logo_pass-128p.png)](${env.BUILD_URL})"""
			}

  			report_content = """${report_content}
* [![Build Status](${env.JENKINS_URL}/buildStatus/icon?job=${env.JOB_NAME}&build=${env.BUILD_NUMBER})](${env.BUILD_URL}) [builds and tests overall are ${currentBuild.currentResult}](${env.BUILD_URL})."""
				
    			def files = findFiles(glob: '*.md')
    			echo("all reports: $files");
    			// def testFileNames = files.split('\n')
    			for (def fileEntry : files) 
    			{    			
    				String file = fileEntry.path;    				
    				
    				String fileContent = readFile(file).trim();
    				
    				echo("$file  -> ${fileContent}");
    				
    				// update report summary
    				report_content = "${report_content}\n${fileContent}"		
    				
    				// update build description
    				// currentBuild.description = "${currentBuild.description}\n${fileContent}"		
    			}    			
    			
  				report_content = """${report_content}

--------------------
_Automatically generated by [sPHENIX Jenkins continuous integration](${env.JOB_DISPLAY_URL})_
[![sPHENIX](https://raw.githubusercontent.com/sPHENIX-Collaboration/utilities/master/jenkins/material/sphenix-logo-white-bg-72p.png)](https://www.sphenix.bnl.gov/web/) &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; [![jenkins.io](https://raw.githubusercontent.com/sPHENIX-Collaboration/utilities/master/jenkins/material/jenkins_logo_title-72p.png)](https://jenkins.io/)"""
    			
			  	writeFile file: "summary.md", text: "${report_content}"		
			  	
				build(job: 'github-comment-label',
					  parameters:
					  [
							string(name: 'ghprbPullLink', value: "${ghprbPullLink}"), 
							string(name: 'LabelCategory', value: ""),
							string(name: 'githubComment', value: "${report_content}")
						],
						wait: false, propagate: false)
			  	
				build(job: 'github-commit-checkrun',
					parameters:
					[
						string(name: 'checkrun_repo_commit', value: "${checkrun_repo_commit}"), 
						string(name: 'src_Job_id', value: "${env.JOB_NAME}/${env.BUILD_NUMBER}"),
						string(name: 'src_details_url', value: "${env.BUILD_URL}"),
						string(name: 'checkrun_status', value: "completed"),
						string(name: 'checkrun_conclusion', value: "${currentBuild.currentResult}"),
						string(name: 'output_title', value: "sPHENIX Jenkins Report for ${env.JOB_NAME}"),
						string(name: 'output_summary', value: "[![Build Status](${env.JENKINS_URL}/buildStatus/icon?job=${env.JOB_NAME}&build=${env.BUILD_NUMBER})](${env.BUILD_URL}) [builds and tests overall are ${currentBuild.currentResult}](${env.BUILD_URL})."),
						string(name: 'output_text', value: "${currentBuild.displayName}\n\n${currentBuild.description}")
					],
					wait: false, propagate: false
				) // build(job: 'github-commit-checkrun',
			
				}// script
				
			}
			
			archiveArtifacts artifacts: 'report/*.md'
			
		}
	
	}
	
}//pipeline 
