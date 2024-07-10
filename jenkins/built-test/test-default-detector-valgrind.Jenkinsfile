pipeline 
{
	agent any
    
//    environment { 
//        JenkinsBase = 'jenkins/test/'
//    }
       
    options {
        timeout(time: 12, unit: 'HOURS') 
    }
    
	stages { 
	
		stage('Checkrun update') 
		{
		
			steps {
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
		
		stage('Prebuild-Cleanup') 
		{
			
            
			steps {
				timestamps {
					ansiColor('xterm') {
					
						// build(job: 'github-comment-label',
						//   parameters:
						//   [
						// 		string(name: 'ghprbPullLink', value: "${ghprbPullLink}"), 
						// 		string(name: 'LabelCategory', value: "valgrind"),
						// 		string(name: 'LabelStatus', value: "PENDING")
						// 	],
						// 	wait: false, propagate: false)
							
						script {
							currentBuild.displayName = "${env.BUILD_NUMBER} - ${system_config} - ${build_type}"						
							currentBuild.description = "${upstream_build_description} - ${system_config} - ${build_type}" 
						
							if (fileExists('./install'))
							{
								sh "rm -fv ./install"
							}
							if (fileExists('./calibrations'))
							{
								sh "rm -fv ./calibrations"
							}						
							if (fileExists('./build'))
							{
								sh "rm -fv ./build"
							}						
						}						
    				
						echo("link builds to ${build_src}")
						sh('ln -svfb ${build_src}/install ./install')
						sh('ln -svfb ${build_src}/calibrations ./calibrations')

						dir('macros')
						{
							deleteDir()
						}	

						dir('coresoftware') {
							deleteDir()
						}

						dir('report')
						{
							deleteDir()
    					}
    				
						sh('ls -lvhc')
					}
				}
			}
		}
	
		stage('Initialize') 
		{
			
            
			steps {
				timestamps {
					ansiColor('xterm') {
					
						sh('hostname')
						sh('pwd')
						sh('env')
						
						sh('ls -lvhc')
    				
						dir('utilities/jenkins/built-test/') {
							
							sh('$singularity_exec_sphenix_farm3 tcsh -f singularity-check.sh ${build_type}')
						
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
						
						dir('macros')
						{				
							
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
							    	[name: "${sha_macros}"]
							    ], 
							  	userRemoteConfigs: 
							  	[[
							     	credentialsId: 'sPHENIX-bot', 
							     	url: '${git_url_macros}',
							     	refspec: ('+refs/pull/*:refs/remotes/origin/pr/* +refs/heads/master:refs/remotes/origin/master'), 
							    	branch: ('*')
							  	]]
								] //checkout
							)//checkout
							
    				}	
    				
						
					}
				}
			}
		}//stage('SCM Checkout')
		
		stage('Test')
		{
			
			
			steps 
			{
				sh("$singularity_exec_sphenix_farm3 sh utilities/jenkins/built-test/test-default-detector.sh ${detector_name} 2 1")										
			}				
					
		}
		
		stage('report')
		{
			steps 
			{			
				archiveArtifacts artifacts: 'macros/detectors/*/*.valgrind*'
				
				publishValgrind (
				  failBuildOnInvalidReports: true,
				  failBuildOnMissingReports: true,
				  failThresholdDefinitelyLost: '1',
				  failThresholdInvalidReadWrite: '0',
				  failThresholdTotal: '1000',
				  pattern: 'macros/detectors/*/*.valgrind.xml',
				  publishResultsForAbortedBuilds: false,
				  publishResultsForFailedBuilds: false,
				  sourceSubstitutionPaths: '',
				  unstableThresholdDefinitelyLost: '0',
				  unstableThresholdInvalidReadWrite: '0',
				  unstableThresholdTotal: '300'
				)			
			}		
		}
		
		stage('PerformanceAnalysis')
		{
			
			
			steps 
			{
					
				sh("${python_bin} utilities/jenkins/built-test/test-output-parser.py --input_file macros/detectors/${detector_name}/*.log --output_csv test-default-detector.csv")
				
				plot( csvFileName: 'test-default-detector.csv_Time_(s)_Summary.csv', 
					csvSeries: 
					[[
						exclusionValues: 'Time (s)', 
						file: 'test-default-detector.csv_Time_(s).csv', 
						inclusionFlag: 'INCLUDE_BY_STRING', 
						url: "${env.JOB_URL}" + '/%build%/'
					]], 
					description: 'User time (s), from system time tool', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					title: 'User time (s)',
					yaxis: 'Time (s)'			
				)
				plot( csvFileName: 'test-default-detector.csv_Memory_(kB)_Summary.csv', 
					csvSeries: 
					[[
						exclusionValues: 'Memory (kB)', 
						file: 'test-default-detector.csv_Memory_(kB).csv', 
						inclusionFlag: 'INCLUDE_BY_STRING', 
						url: "${env.JOB_URL}" + '/%build%/'
					]], 
					description: 'Maximum resident set size (kbytes), from system time tool', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					title: 'Maximum resident memory',
					yaxis: 'Memory (kB)'			
				)
				plot( csvFileName: 'test-default-detector.csv_STDOUT_Linecount_Summary.csv', 
					csvSeries: 
					[[
						exclusionValues: 'STDOUT Linecount', 
						file: 'test-default-detector.csv_STDOUT_Linecount.csv', 
						inclusionFlag: 'INCLUDE_BY_STRING', 
						url: "${env.JOB_URL}" + '/%build%/artifact/macros/detectors/sPHENIX/Fun4All_G4_sPHENIX.log'
					]], 
					description: 'line count of the text output', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					title: 'Output line count',
					yaxis: 'Line count'			
				)
			}							
		} // stage('PerformanceAnalysis')
	}//stages

	
	post {

		always{
		  
			dir('report')
			{
			  writeFile file: "valgrind-${system_config}-${build_type}.md", text: "* [![Build Status](${env.JENKINS_URL}/buildStatus/icon?job=${env.JOB_NAME}&build=${env.BUILD_NUMBER})](${env.BUILD_URL}) system `${system_config}`, build `${build_type}`: Valgrind test: [build is ${currentBuild.currentResult}](${env.BUILD_URL}), [:bar_chart:valgrind report](${env.BUILD_URL}/valgrindResult/), [trends :bar_chart:](${env.JOB_URL}/plot/) "				
			}
		  		  
			archiveArtifacts artifacts: 'report/*.md'
			
			
			build(job: 'github-commit-checkrun',
				parameters:
				[
					string(name: 'checkrun_repo_commit', value: "${checkrun_repo_commit}"), 
					string(name: 'src_Job_id', value: "${env.JOB_NAME}/${env.BUILD_NUMBER}"),
					string(name: 'src_details_url', value: "${env.BUILD_URL}"),
					string(name: 'checkrun_status', value: "completed"),
					string(name: 'checkrun_conclusion', value: "${currentBuild.currentResult}"),
					string(name: 'output_title', value: "sPHENIX Jenkins Report for ${env.JOB_NAME}"),
					string(name: 'output_summary', value: "* [![Build Status](${env.JENKINS_URL}/buildStatus/icon?job=${env.JOB_NAME}&build=${env.BUILD_NUMBER})](${env.BUILD_URL}) system `${system_config}`, build `${build_type}`: Valgrind test: [build is ${currentBuild.currentResult}](${env.BUILD_URL}), [:bar_chart:valgrind report](${env.BUILD_URL}/valgrindResult/), [trends :bar_chart:](${env.JOB_URL}/plot/) "),
					string(name: 'output_text', value: "${currentBuild.displayName}\n\n${currentBuild.description}")
				],
				wait: false, propagate: false
			) // build(job: 'github-commit-checkrun',
			
			archiveArtifacts artifacts: 'macros/**/*.log'

		}
		// success {
		// 	build(job: 'github-comment-label',
		// 	  parameters:
		// 	  [
		// 			string(name: 'ghprbPullLink', value: "${ghprbPullLink}"), 
		// 			string(name: 'LabelCategory', value: "valgrind"),
		// 			string(name: 'LabelStatus', value: "AVAILABLE")
		// 		],
		// 		wait: false, propagate: false)
		// }
		// failure {
		// 	build(job: 'github-comment-label',
		// 	  parameters:
		// 	  [
		// 			string(name: 'ghprbPullLink', value: "${ghprbPullLink}"), 
		// 			string(name: 'LabelCategory', value: "valgrind"),
		// 			string(name: 'LabelStatus', value: "FAIL")
		// 		],
		// 		wait: false, propagate: false)
		// }
		// unstable {
		// 	build(job: 'github-comment-label',
		// 	  parameters:
		// 	  [
		// 			string(name: 'ghprbPullLink', value: "${ghprbPullLink}"), 
		// 			string(name: 'LabelCategory', value: "valgrind"),
		// 			string(name: 'LabelStatus', value: "AVAILABLE")
		// 		],
		// 		wait: false, propagate: false)
		// }
	}
}//pipeline 
