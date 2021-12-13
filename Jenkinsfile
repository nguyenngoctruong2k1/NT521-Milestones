pipeline {
  agent { label 'linux' }
  tools {
    maven 'mvn-3.6.3'
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    HEROKU_API_KEY = credentials('java-heroku-api-key')
  }
  parameters { 
    string(name: 'APP_NAME', defaultValue: '', description: 'What is the Heroku app name?') 
  }
  stages {
    stage('Build') {
      steps {
        // sh 'docker --version'
        sh 'docker build -t darinpope/java-web-app:latest .'
      }
    }
    
    stage ('OWASP Dependency-Check Vulnerabilities') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh 'mvn dependency-check:check'
        }
        dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      }
    }
    
    stage('Push to Heroku registry') {
      steps {
        sh '''
          echo $HEROKU_API_KEY | docker login --username=_ --password-stdin registry.heroku.com
          
          docker tag darinpope/java-web-app:latest registry.heroku.com/$APP_NAME/web
          docker push registry.heroku.com/$APP_NAME/web
        '''
      }
    }
    stage('Release the image') {
      steps {
        sh '''
          curl https://cli-assets.heroku.com/install.sh | sh
          heroku container:release web --app=$APP_NAME
        '''
      }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}
