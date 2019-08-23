node {
	currentBuild.displayName = "1.${BUILD_NUMBER}"
	def GIT_COMMIT
	stage ('SCM'){
		checkout([
			$class: 'GitSCM', branches: [[name: '*/master']],
			userRemoteConfigs: [[url: 'https://github.com/testqaprj/petclinic.git'],[credentialsId:'kuchv']]
		])
		GIT_COMMIT = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
	}

	stage ('Build') {
		withMaven(jdk: 'JDK_local', maven: 'MVN_Local') {
			sh 'mvn clean package'
		}
	}

	/*stage ('Cucumber'){
		withMaven(jdk: 'JDK_local', maven: 'MVN_Local') {
			sh 'mvn test -Dtest=<CUCUMBER_TEST_CLASS_NAME>'
		}
		cucumber buildStatus: "Success",
		//fileIncludePattern: "**///cucumber.json",
		//jsonReportDirectory: '<Path/to/report/folder>'
	//}

	stage('SonarQube Analysis'){
		def mvnHome = tool name : 'MVN_Local', type:'maven'
		withSonarQubeEnv('sonar-server'){
			"SONAR_USER_HOME=/opt/bitnami/jenkins/.sonar ${mvnHome}/bin/mvn sonar:sonar"
			sh  "${mvnHome}/bin/mvn sonar:sonar -Dsonar.projectKey=PETCL12 -Dsonar.projectName=Sonar_pet_project"
		}
	}


	/*stage ("Appscan"){
		sleep 40
		appscan application: '<APPSCAN_APPLICATION_NAME>',
				credentials: '<ASOC_CREDENTIAL_ID>',
				failBuild: true,
				failureConditions: [failure_condition(failureType: 'high', threshold: 100)],
				name: '<APPSCAN_APPLICATION_NAME>',
				scanner: static_analyzer(hasOptions: false, target: '/var/jenkins_home/jobs/project_name'),
				type: 'Static Analyzer',
				wait: true
	}*/

	stage('Publish Artificats to UCD'){
		step([$class: 'UCDeployPublisher',
			siteName: 'ucd-server',
			component: [
				$class: 'com.urbancode.jenkins.plugins.ucdeploy.VersionHelper$VersionBlock',
				componentName: 'pet-component',
				createComponent: [
					$class: 'com.urbancode.jenkins.plugins.ucdeploy.ComponentHelper$CreateComponentBlock',
					componentTemplate: '',
					componentApplication: 'pet-clinic-app'
				],
				delivery: [
					$class: 'com.urbancode.jenkins.plugins.ucdeploy.DeliveryHelper$Push',
					pushVersion: '1.${BUILD_NUMBER}',
					baseDir: '/var/jenkins_home/workspace/${JOB_NAME}/target',
					fileIncludePatterns: '*.war',
					fileExcludePatterns: '',
					pushDescription: 'Pushed from Jenkins'
				]
			]
		])
		def newComponentVersionId = "${pet-component_VersionId}"
		step(
			$class: 'UploadBuild',
			tenantId: "5ade13625558f2c6688d15ce",
			revision: "${GIT_COMMIT}",
			appName: "pet-clinic-app", requestor: "admin", id: "${newComponentVersionId}"
		)
		sleep 25
		step([$class: 'UCDeployPublisher',
			deploy: [ createSnapshot: [deployWithSnapshot: true,
					snapshotName: "1.${BUILD_NUMBER}"],
				deployApp: 'pet-clinic-app',
				deployDesc: 'Requested from Jenkins',
				deployEnv: 'dev',
				deployOnlyChanged: false,
				deployProc: 'pet-app-process',
				deployReqProps: '',
				deployVersions: "pet-component:1.${BUILD_NUMBER}"],
			siteName: 'ucd-server']
		)
	}

	stage ('HCL One Test') {
		sleep 25
		echo 'Executing HCL One test ... '
		sh '/var/jenkins_home/onetest/hcl-onetest-command.sh <WORKSPACE_NAME> <APPLICATION_ENVIRONMENT_URL>'
	}
}
