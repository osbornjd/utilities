pipeline 
{
	agent any
    
//    environment { 
//        JenkinsBase = 'jenkins/test/'
//    }
    options {
        timeout(time: 10, unit: 'HOURS') 
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
		    			// parameters:
		    			// [
		    			// 	string(name: 'ghprbPullLink', value: "${ghprbPullLink}"), 
			    		// 	string(name: 'LabelCategory', value: "track-pythia-QA"),
			    		// 	string(name: 'LabelStatus', value: "PENDING")
			    		// ],
		    			// wait: false, propagate: false)
					
						script {
						
							currentBuild.description = "${upstream_build_description}" 
			
							if (fileExists('./install'))
							{
								sh "rm -frv ./install"
							}
							if (fileExists('./calibrations'))
							{
								sh "rm -frv ./calibrations"
							}						
							if (fileExists('./build'))
							{
								sh "rm -frv ./build"
							}							
						}						
    				
						echo("link builds to ${build_src}")
						sh('ln -svfb ${build_src}/install ./install')
						sh('ln -svfb ${build_src}/calibrations ./calibrations')

						dir('macros')
						{
							deleteDir()
						}	

						dir('coresoftware') 
						{
							deleteDir()
						}

						dir('report')
						{
							deleteDir()
    					}
						
						dir('QA-gallery')
						{
							deleteDir()
    					}
    					
						dir('qa_html')
						{
							deleteDir()
    					}
    					
						dir('reference')
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
							
							sh('$singularity_exec_sphenix_farm3  tcsh singularity-check.sh ${build_type}')
						
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
							  				mergeTarget: 'QA-tracking-pythiajet'
							  			]
							    	],   
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
    				
				
						dir('QA-gallery')
						{			
							
							checkout(
								[
						 			$class: 'GitSCM',
						   		extensions: [               
							   		[$class: 'CleanCheckout'],  
									[$class: 'CloneOption', timeout: 60, shallow: true, noTags: true]
						   		],
							  	branches: [
							    		[name: "${sha_QA_gallery}"]
							    	], 
							  	userRemoteConfigs: 
							  	[[
							     	credentialsId: 'sPHENIX-bot', 
							     	url: '${git_url_QA_gallery}',
							     	refspec: ('+refs/pull/*:refs/remotes/origin/pr/* +refs/heads/main:refs/remotes/origin/main'), 
							    	branch: ('*')
							  	]]
								] //checkout
							)//checkout
							
    						}	
						
					}
				}
			}
		}//stage('SCM Checkout')
		
		stage('Copy reference')
		{
			
			when {
    			// case insensitive regular expression for truthy values
					expression { return use_reference ==~ /(?i)(Y|YES|T|TRUE|ON|RUN)/ }
			}
			steps 
			{
				timestamps { 
					ansiColor('xterm') {
						
						dir('reference')
						{
    					copyArtifacts(projectName: "test-tracking-pythiajet-qa-reference", selector: lastSuccessful());

							sh('ls -lvhc')
    				}
						
					}
				}
			}
		}//stage('SCM Checkout')
		
		stage('Test')
		{
			
			
			steps 
			{
					
				sh('$singularity_exec_sphenix_farm3  sh utilities/jenkins/built-test/test-tracking-qa.sh $num_event $number_jobs')
												
			}				
					
		}
		
		
		stage('html-report')
		{
			steps 
			{	
			
				dir('qa_html')
				{					
					sh('ls -lhv')
				}
			
				sh("cp -fv QA-gallery/*.html qa_html/");
				
				script {
					def html_files = findFiles(glob: 'qa_html/*.html').join(',')
					echo("all html_files: $html_files");
					publishHTML (target: [
					      allowMissing: false,
					      alwaysLinkToLastBuild: false,
					      keepAll: true,
					      reportDir: 'qa_html',
					      reportFiles: "$html_files",
					      reportName: "QA Report"
					    ])
				}
			}			// steps	
					
		}
		
		stage('PerformanceAnalysis')
		{
			
			
			steps 
			{
					
				sh("${python_bin} utilities/jenkins/built-test/test-output-parser.py --input_file macros/detectors/sPHENIX/G4sPHENIX_job*.log --output_csv test-default-detector.csv")
				
				plot( csvFileName: 'test-default-detector.csv_Time_(s)_Summary.csv', 
					csvSeries: 
					[[
						exclusionValues: 'Time (s),min,max', 
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
						exclusionValues: 'Memory (kB),min,max', 
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
						exclusionValues: 'STDOUT Linecount,min,max', 
						file: 'test-default-detector.csv_STDOUT_Linecount.csv', 
						inclusionFlag: 'INCLUDE_BY_STRING', 
						url: "${env.JOB_URL}" + '/%build%/'
					]], 
					description: 'line count of the text output', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					title: 'Output line count',
					yaxis: 'Line count'			
				)
				
				
				plot( csvFileName: 'Upsilon_count_Summary.csv', 
					csvSeries: 
					[[
						file: 'QA-gallery/Upsilon_count.csv', 
						exclusionValues: '', 
						inclusionFlag: 'OFF', 
						url: "${env.JOB_URL}" + '/%build%/'
					]], 
					description: 'Upsilon count from the Crystal Ball fit parameter 0', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					title: 'Upsilon reconstructed',
					yaxis: 'Reconstructed yield'			
				)
				plot( csvFileName: 'Upsilon_mean_Summary.csv', 
					csvSeries: 
					[[
						file: 'QA-gallery/Upsilon_mean.csv', 
						exclusionValues: '', 
						inclusionFlag: 'OFF', 
						url: "${env.JOB_URL}" + '/%build%/'
					]], 
					description: 'Upsilon peak position from the Crystal Ball fit parameter 1', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					title: 'Upsilon reconstructed peak position',
					yaxis: 'peak position (GeV)'			
				)
				plot( csvFileName: 'Upsilon_width_Summary.csv', 
					csvSeries: 
					[[
						file: 'QA-gallery/Upsilon_width.csv', 
						exclusionValues: '', 
						inclusionFlag: 'OFF', 
						url: "${env.JOB_URL}" + '/%build%/'
					]], 
					description: 'Upsilon peak width from the Crystal Ball fit parameter 2', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					title: 'Upsilon reconstructed peak width',
					yaxis: 'peak width (GeV)'		
				)				
				
				plot( csvFileName: 'test-default-detector.csv_Module_per_event_time_(ms)_Summary.csv', 
					csvSeries: 
					[[
						file: 'test-default-detector.csv_Module_per_event_time_(ms).csv', 
						exclusionValues: '', 
						inclusionFlag: 'OFF', 
						url: "${env.JOB_URL}" + '/%build%/'
					]], 
					description: 'per-event time (ms) for each of the Fun4All modules', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					title: 'per-event time (ms)',
					yaxis: 'time (ms)'			
				)		
				
				plot( csvFileName: 'QAG4SimulationTracking_pTRecoGenRatio_pTGen_errorbar_Summary.csv', 
					csvSeries: 
					[[
						file: 'QA-gallery/QAG4SimulationTracking_pTRecoGenRatio_pTGen_errorbar.csv', 
						exclusionValues: '', 
						inclusionFlag: 'OFF', 
						url: "${env.JOB_URL}" + '/%build%/'
					]], 
					description: 'pT resolution from tracking QA. Binned in truth p_T in GeV/c', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					logarithmic: 'true',
					title: 'pT resolution',
					yaxis: 'Resolution'			
				)
				
				plot( csvFileName: 'QAG4SimulationTracking_DCArPhi_errorbar_Summary.csv', 
					csvSeries: 
					[[
						file: 'QA-gallery/QAG4SimulationTracking_DCArPhi_errorbar.csv', 
						exclusionValues: '', 
						inclusionFlag: 'OFF', 
						url: "${env.JOB_URL}" + '/%build%/'
					]], 
					description: 'DCA rphi resolution from vertex QA. Binned in truth p_T in GeV/c', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					logarithmic: 'true',
					title: 'DCA rphi',
					yaxis: 'DCA rphi (cm)'			
				)
				
				plot( csvFileName: 'QAG4SimulationTracking_DCAZ_errorbar_Summary.csv', 
					csvSeries: 
					[[
						file: 'QA-gallery/QAG4SimulationTracking_DCAZ_errorbar.csv', 
						exclusionValues: '', 
						inclusionFlag: 'OFF', 
						url: "${env.JOB_URL}" + '/%build%/'
					]], 
					description: 'DCA z resolution from vertex QA. Binned in truth p_T in GeV/c', 
					exclZero: true, 
					group: 'Analysis', 
					numBuilds: '40', 
					style: 'line',
					logarithmic: 'true',
					title: 'DCA z',
					yaxis: 'DCA z (cm)'			
				)
			}								
		}// 		stage('PerformanceAnalysis')
	}//stages

	
	post {
	
		always{
		  
			//  writeFile file: "QA-tracking-low-occupancy-qa.md", text: "* [![Build Status ](${env.JENKINS_URL}/buildStatus/icon?job=${env.JOB_NAME}&build=${env.BUILD_NUMBER})](${env.BUILD_URL}) Tracking QA at low occupancy: [build is ${currentBuild.currentResult}](${env.BUILD_URL}), [:bar_chart: trends](${env.JOB_URL}/plot/)"				
			dir('report')
			{
				echo("start report building to ...");
				sh ('pwd');
			}
			script
			{								
				currentBuild.description = "${currentBuild.description}\n## Result QA reports:"
				
				def report_content = "* [![Build Status ](${env.JENKINS_URL}/buildStatus/icon?job=${env.JOB_NAME}&build=${env.BUILD_NUMBER})](${env.BUILD_URL}) Tracking QA for Pythia D0-jet: [build is ${currentBuild.currentResult}](${env.BUILD_URL}), [:bar_chart: trends](${env.JOB_URL}/plot/)";	        

				def files = findFiles(glob: 'QA-gallery/report*.md')
				echo("all reports: $files");
				// def testFileNames = files.split('\n')
				for (def fileEntry : files) 
				{    			
					String file = fileEntry.path;    				

					String fileContent = readFile(file).trim();

					echo("$file  -> ${fileContent}");

					// update report summary
					report_content = "${report_content}\n  ${fileContent}"		//nested list for child reports

					// update build description
					currentBuild.description = "${currentBuild.description}\n${fileContent}"		
				}    			

				writeFile file: "report/QA-tracking-pythiajet-qa.md", text: "${report_content}"	

			}//script
			
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
					string(name: 'output_summary', value: "* [![Build Status ](${env.JENKINS_URL}/buildStatus/icon?job=${env.JOB_NAME}&build=${env.BUILD_NUMBER})](${env.BUILD_URL}) Tracking QA for Pythia D0-jet: [build is ${currentBuild.currentResult}](${env.BUILD_URL})"),
					string(name: 'output_text', value: "${currentBuild.displayName}\n\n${currentBuild.description}")
				],
				wait: false, propagate: false
			) // build(job: 'github-commit-checkrun',			
			
			archiveArtifacts artifacts: 'QA-gallery/G4sPHENIX_*_Sum*_qa.root*'		
			archiveArtifacts artifacts: 'macros/detectors/sPHENIX/*_1.log'
		}

		success {
			script {
				currentBuild.description = "${currentBuild.description}<br><button onclick=\"window.location.href='${JENKINS_URL}/job/sPHENIX/job/test-tracking-pythiajet-qa-reference/parambuild/?ref_build_id=${BUILD_ID}';\">Use as QA reference</button>" 
			}
			
			// build(job: 'github-comment-label',
			//   parameters:
			//   [
			// 		string(name: 'ghprbPullLink', value: "${ghprbPullLink}"), 
			// 		string(name: 'LabelCategory', value: "track-pythia-QA"),
			// 		string(name: 'LabelStatus', value: "AVAILABLE")
			// 	],
			// 	wait: false, propagate: false)
				
		}
		failure {
			// build(job: 'github-comment-label',
			//   parameters:
			//   [
			// 		string(name: 'ghprbPullLink', value: "${ghprbPullLink}"), 
			// 		string(name: 'LabelCategory', value: "track-pythia-QA"),
			// 		string(name: 'LabelStatus', value: "FAIL")
			// 	],
			// 	wait: false, propagate: false)
			
			archiveArtifacts artifacts: 'macros/**/*.log'
		}
		// unstable {
		// 	build(job: 'github-comment-label',
		// 	  parameters:
		// 	  [
		// 			string(name: 'ghprbPullLink', value: "${ghprbPullLink}"), 
		// 			string(name: 'LabelCategory', value: "track-pythia-QA"),
		// 			string(name: 'LabelStatus', value: "AVAILABLE")
		// 		],
		// 		wait: false, propagate: false)
		// }
	}
}//pipeline 
