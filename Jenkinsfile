def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label) {
node(label) {
  stage ('cloning the repository'){
      git 'https://github.com/tapansirol/petclinic.git'
  }
	
  //stage('SonarQube analysis') {
    // requires SonarQube Scanner 2.8+
    //def scannerHome = tool 'sonar-new';
    //withSonarQubeEnv('My SonarQube Server') {
     // sh "${scannerHome}/bin/sonar-scanner"
    //}
  //}

  stage ('Build') {
      withMaven(jdk: 'JDK_local', maven: 'MVN_Local') {
      sh "echo JAVA_HOME=$JAVA_HOME"
      sh 'mvn clean package'
	      
    }
  }

stage('Publish artificats to UrbanCode Deploy'){
   step([$class: 'UCDeployPublisher',
        siteName: 'ucd-server',
        component: [
            $class: 'com.urbancode.jenkins.plugins.ucdeploy.VersionHelper$VersionBlock',
            componentName: 'PetClinic-cmp',
            createComponent: [
                $class: 'com.urbancode.jenkins.plugins.ucdeploy.ComponentHelper$CreateComponentBlock',
                componentTemplate: '',
                componentApplication: 'PetClinic'
            ],
            delivery: [
                $class: 'com.urbancode.jenkins.plugins.ucdeploy.DeliveryHelper$Push',
                pushVersion: 'Ver.${BUILD_NUMBER}',
                baseDir: '/home/jenkins/workspace/PetClinic/target',
                fileIncludePatterns: '*.war',
                fileExcludePatterns: '',
               // pushProperties: 'jenkins.server=Jenkins-app\njenkins.reviewed=false',
                pushDescription: 'Pushed from Jenkins'
            ]
        ]
    ])
	step([$class: 'UCDeployPublisher',
        	siteName: 'ucd-server',
        	deploy: [
            	$class: 'com.urbancode.jenkins.plugins.ucdeploy.DeployHelper$DeployBlock',
            	deployApp: 'PetClinic',
            	deployEnv: 'Dev',
            	deployProc: 'Deploy-PetClinic',
            	createProcess: [
                	$class: 'com.urbancode.jenkins.plugins.ucdeploy.ProcessHelper$CreateProcessBlock',
                	processComponent: 'Deploy'
            	],
            	deployVersions: 'PetClinic-cmp:Ver.${BUILD_NUMBER}',
		//deployVersions: 'SNAPSHOT=Base Configuration',
            	deployOnlyChanged: false
        ]
    ])
} 
	
	
	}
}
