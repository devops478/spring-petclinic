pipeline {
    agent none
	
	environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "ec2-34-201-102-23.compute-1.amazonaws.com:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "spring-petclinic"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
        // Tomcat deployment using SSH Agent.
        ARTIFACT_ID_PKG = "spring-petclinic.war"
    }
	
    stages {
        stage('clone'){
            agent {
                label "master"
            }
            tools {
              maven "MAVEN3.8.1"
            }
            steps {
			 script {
			    // Let's clone the source
                git credentialsId: '976c8a11-ab25-4ef8-9344-39d3de2f67db', url: 'https://github.com/devops478/spring-petclinic.git'
            }
          }
	    }
        stage('build'){
            agent {
                label "master"
            }
            steps {
			  script {
                sh 'mvn install'
            }
		  }
        }
       stage('Run Junit test cases') {
          agent {
              label "master"
          }
          steps {
            junit '**/target/surefire-reports/*.xml'
            }
        }
	   stage("publish to nexus") {
		    agent {
		        label "master"
		    }
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;

                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],

                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );

                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
       			  }
		      } 
	    
		  }
		 stage('Nexus artificat download') {
		     agent { 
		         label "master"
		     }
		     steps {
		       script {
			    withCredentials([usernameColonPassword(credentialsId: 'nexus-credentials', variable: 'nexuscred')]) {
                sh "curl -v -u ${nexuscred} ${NEXUS_PROTOCOL}://${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/org/springframework/samples/${pom.artifactId}/${pom.version}/${pom.artifactId}-${pom.version}.${pom.packaging} > ${pom.artifactId}.${pom.packaging}"
		           }
	            }
		     }
		 }
		 // Need to install 'SSH Agent Plugin' and create add 'SSH Username with private key' credentials(Generate ssh-keygen on target server and add private key in jenkins).
         stage("deploy to tomcat"){
             agent {
		         label "master"
		     }
             steps{
               sshagent(['devops-tomcat-deploy-sshkey']) {
               sh '''
                scp -o StrictHostKeyChecking=no ${ARTIFACT_ID_PKG} devops@ec2-100-26-50-45.compute-1.amazonaws.com:/u01/devops-tools/apache-tomcat-8.5.64/webapps
                ssh devops@ec2-100-26-50-45.compute-1.amazonaws.com /u01/devops-tools/apache-tomcat-8.5.64/bin/shutdown.sh
                ssh devops@ec2-100-26-50-45.compute-1.amazonaws.com /u01/devops-tools/apache-tomcat-8.5.64/bin/startup.sh
               '''
               }
             }
         }
         /*stage('deploy to tomcat') {
		     agent {
		         label "master"
		     }
		     steps {
		       script {
		        withCredentials([usernameColonPassword(credentialsId: 'tomcat_credentials', variable: 'mycred')]) {
                sh "curl -v -u ${mycred} http://ec2-100-26-50-45.compute-1.amazonaws.com:8081/manager/text/undeploy?path=/${pom.artifactId}"
                sh "curl -v -u ${mycred} -T ${pom.artifactId}.${pom.packaging} http://ec2-100-26-50-45.compute-1.amazonaws.com:8081/manager/text/deploy?path=/${pom.artifactId}&update=true"
                sh "curl -v -u ${mycred} http://ec2-100-26-50-45.compute-1.amazonaws.com:8081/manager/text/start?path=/${pom.artifactId}"
	           	  }
		        }
	         }
         }*/
        }
      
    /*post {
       success {
         mail to: 'ksandy.katta@gmail.com',
         subject: "SUCCESS: Status of pipeline: ${currentBuild.fullDisplayName}",
         body: "${env.BUILD_URL} has result ${currentBuild.result}"
       }
       failure {
          mail to: 'ksandy.katta@gmail.com',
          subject: "FAILURE: Status of pipeline: ${currentBuild.fullDisplayName}",
          body: "${env.BUILD_URL} has result ${currentBuild.result}"
       }
    }*/
}