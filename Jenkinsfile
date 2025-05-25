pipeline {
     agent { label 'Jenkins-Agent' }

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven 'Maven3'
        jdk 'Java17'
		
    }
	 environment {
        // Replace with the name of your SonarQube server in Jenkins config
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS = 'docker' // Your Docker Hub creds ID
        IMAGE_NAME = 'vinao/blogging-app'
        IMAGE_TAG = 'latest'
    }

    stages {
        
        stage('Git Checkout') {
            steps {
                // Get some code from a GitHub repository
                git branch: 'main', changelog: false, credentialsId: 'github', poll: false, url: 'https://github.com/VinoaN/FullStack-Blogging-App.git'
              }
        }
        
          stage('Compile') {
            steps {
                // Get some code from a GitHub repository
                         sh "mvn clean compile"
                         sh 'ls -l target/classes'

 }
        }
		stage('Sonarqube Analysis') {
           steps {
		withSonarQubeEnv('MySonarQube') {
sh ''' 
$SCANNER_HOME/bin/sonar-scanner \
  -Dsonar.projectKey=Blogging-App \
  -Dsonar.projectName="Blogging App" \
  -Dsonar.projectVersion=1.0 \
  -Dsonar.sources=src/main/java \
  -Dsonar.java.binaries=target/classes \
  -Dsonar.sourceEncoding=UTF-8
'''
    }
}
}

stage('OWASP Dependency Check') {
    steps {
     
          dependencyCheck additionalArguments: ' -- scan ./', 
      odcInstallation: 'DP'
      dependencyCheckPublisher pattern: '*/dependency-check-report.xml'

    }
}

stage('Build Application') {
    steps {
     
     sh "mvn clean install"
    }
}
 stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        def app = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                        app.push()
                    }
                }
            }

        }
        stage('Triggger CD Pipeline') {
    steps {
     
    build job:"Java_Project_CD" ,wait : true
    }
}
		}
		}
