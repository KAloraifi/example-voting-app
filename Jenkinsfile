pipeline {
  agent none

  stages {
    stage("worker-build") {
      when {
        changeset "**/worker/**"
      }
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }
      }

      steps {
        echo 'Compiling worker app'
        dir('worker') {
         sh 'mvn compile'
        }
      } 
    }
    stage("worker-test") {
      when {
        changeset "**/worker/**"
      }
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }
      }

      steps {
        echo 'Running Unit Tests on worker app'
        dir('worker') {
          sh 'mvn clean test'
        }
      }
    }
    stage("worker-package") {
      when {
        branch 'master'
        changeset "**/worker/**"
      }
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }
      }

      steps {
        echo 'Packaging worker app'
        dir('worker') {
          sh 'mvn package -DskipTests'
        }
      }
    }
    stage('worker-docker-package'){
      agent any
      when {
        changeset "**/worker/**"
        branch 'master'
      }
      steps{
        echo 'Packaging worker app with docker'
        script{
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
              def workerImage = docker.build("kaloraifi/worker:v${env.BUILD_ID}", "./worker")
              workerImage.push()
              workerImage.push("${env.BRANCH_NAME}")
              workerImage.push("latest")
          }
        }
      }
    }
    stage('result-build') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }
      }
      when {
        changeset '**/result/**'
      }
      steps {
        dir('result') {
          sh 'npm install'
        }
      }
    }
    stage('result-test') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }
      }
      when {
        changeset '**/result/**'
      }
      steps {
        dir('result') {
          sh 'npm install'
          sh 'npm test'
        }
      }
    }
    stage('result-package-docker') {
      agent any
      when {
        changeset '**/result/**'
        branch 'master'
      }
      steps {
        echo 'packaging result app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def resultImage = docker.build("kaloraifi/result:v${env.BUILD_ID}", "./result")
            resultImage.push()
            resultImage.push("${env.BRANCH_NAME}")
            resultImage.push("latest")
          }
        }
      }
    }
    stage('vote-build') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }
      }
      when {
        changeset '**/vote/**'
      }
      steps {
        dir('vote') {
          sh 'pip install -r requirements.txt'
        }
      }
    }
    stage('vote-test') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }
      }
      when {
        changeset '**/vote/**'
      }
      steps {
        dir('vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }
      }
    }
    stage('vote-docker-package') {
      agent any
      when {
        changeset '**/vote/**'
        branch 'master'
      }
      steps {
        echo 'packaging vote app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def voteImage = docker.build("kaloraifi/vote:v${env.BUILD_ID}", "./vote")
            voteImage.push()
            voteImage.push("${env.BRANCH_NAME}")
            voteImage.push("latest")
          }
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
      echo 'Building multibranch monopipeline is completed..'
    }
  }
}