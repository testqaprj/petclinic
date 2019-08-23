node {
	currentBuild.displayName = "1.${BUILD_NUMBER}"
	def GIT_COMMIT
	stage ('SCM'){
		checkout([
			$class: 'GitSCM', branches: [[name: '*/<BRANCH_NAME>']],
			userRemoteConfigs: [[url: '<GIT_REPO_URL>'],[credentialsId:'<GIT_CREDENTIAL_ID>']]
		])
		GIT_COMMIT = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
	}

	stage ('Build') {
		withMaven(jdk: 'JDK_local', maven: 'MVN_Local') {
			sh 'mvn clean package'
		}
	}

	stage ('Cucumber'){
		withMaven(jdk: 'JDK_local', maven: 'MVN_Local') {
			sh 'mvn test -Dtest=<CUCUMBER_TEST_CLASS_NAME>'
		}
		cucumber buildStatus: "Success",
		fileIncludePattern: "**/cucumber.json",
		jsonReportDirectory: '<Path/to/report/folder>'
	}

	stage('SonarQube Analysis'){
		def mvnHome = tool name : 'MVN_Local', type:'maven'
		withSonarQubeEnv('sonar-server'){
			"SONAR_USER_HOME=/opt/bitnami/jenkins/.sonar ${mvnHome}/bin/mvn sonar:sonar"
			sh  "${mvnHome}/bin/mvn sonar:sonar -Dsonar.projectKey=<PROJECT_KEY> -Dsonar.projectName=<PROJECT_NAME>"
		}
	}


	stage ("Appscan"){
		sleep 40
		appscan application: '<APPSCAN_APPLICATION_NAME>',
				credentials: '<ASOC_CREDENTIAL_ID>',
				failBuild: true,
				failureConditions: [failure_condition(failureType: 'high', threshold: 100)],
				name: '<APPSCAN_APPLICATION_NAME>',
				scanner: static_analyzer(hasOptions: false, target: '/var/jenkins_home/jobs/project_name'),
				type: 'Static Analyzer',
				wait: true
	}

	stage('Publish Artificats to UCD'){
		step([$class: 'UCDeployPublisher',
			siteName: '<UCD_SERVER_NAME>',
			component: [
				$class: 'com.urbancode.jenkins.plugins.ucdeploy.VersionHelper$VersionBlock',
				componentName: '<COMPONENT_NAME>',
				createComponent: [
					$class: 'com.urbancode.jenkins.plugins.ucdeploy.ComponentHelper$CreateComponentBlock',
					componentTemplate: '',
					componentApplication: '<APPLICATION_NAME>'
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
		def newComponentVersionId = "${JpetComponent_VersionId}"
		step(
			$class: 'UploadBuild',
			tenantId: "<UCD_TENANTID>",
			revision: "${GIT_COMMIT}",
			appName: "<APPLICATION_NAME>", requestor: "admin", id: "${newComponentVersionId}"
		)
		sleep 25
		step([$class: 'UCDeployPublisher',
			deploy: [ createSnapshot: [deployWithSnapshot: true,
					snapshotName: "1.${BUILD_NUMBER}"],
				deployApp: '<APPLICATION_NAME>',
				deployDesc: 'Requested from Jenkins',
				deployEnv: '<PROJECT_DEPLOYMENT_ENVIRONMENT>',
				deployOnlyChanged: false,
				deployProc: 'Deploy-<PROJECT_NAME>',
				deployReqProps: '',
				deployVersions: "<COMPONENT_NAME>:1.${BUILD_NUMBER}"],
			siteName: '<UCD_SERVER_NAME>']
		)
	}

	stage ('HCL One Test') {
		sleep 25
		echo 'Executing HCL One test ... '
		sh '/var/jenkins_home/onetest/hcl-onetest-command.sh <WORKSPACE_NAME> <APPLICATION_ENVIRONMENT_URL>'
	}
}
