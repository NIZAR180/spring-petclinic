pipeline {
  agent any

  tools {
    maven 'Maven 3'        // Le nom que tu as défini dans Jenkins (Manage Jenkins > Global Tool Configuration)
    jdk 'JDK 17'           // Si défini, sinon supprime cette ligne
  }

  environment {
    DOCKER_IMAGE = 'spring-petclinic-app'
  }

  stages {
    stage('Git Checkout') {
      steps {
        git 'https://github.com/NIZAR180/spring-petclinic.git'
      }
    }

    stage('OWASP Dependency-Check') {
      steps {
        dependencyCheck additionalArguments: '--scan . --format XML', odcInstallation: 'DC'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      }
    }

    stage('Build Maven') {
      steps {
        bat 'mvn clean package'
      }
    }

    stage('Trivy Scan') {
      steps {
        bat 'C:\\Tools\\Trivy\\trivy.exe fs --format table --output trivy-report.txt .'
      }
    }

    stage('Docker Build') {
      steps {
        bat "docker build -t ${DOCKER_IMAGE} ."
      }
    }

    stage('Docker Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin"
          bat "docker tag ${DOCKER_IMAGE} %DOCKER_USER%/${DOCKER_IMAGE}"
          bat "docker push %DOCKER_USER%/${DOCKER_IMAGE}"
        }
      }
    }

    stage('Docker Run') {
      steps {
        bat "docker run -d -p 8080:8080 ${DOCKER_IMAGE}"
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: '**/*.txt, **/*.xml', fingerprint: true
    }
  }
}
