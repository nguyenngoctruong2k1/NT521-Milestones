pipeline {
  agent { label 'linux' }

  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    HEROKU_API_KEY = credentials('java-heroku-api-key')
  }
  parameters { 
    string(name: 'APP_NAME', defaultValue: 'milestones521', description: 'What is the Heroku app name?') 
  }
  stages {
    stage('Build') {
      steps {
        // sh 'docker --version'
        sh 'docker build -t butterflies/milestones521:latest .'
      }
    }
    
//     stage ('OWASP Dependency-Check Vulnerabilities') {
//         steps {
//             dependencyCheck additionalArguments: ''' 
//                 -o "./" 
//                 -s "./"
//                 -f "ALL" 
//                 --prettyPrint''', odcInstallation: 'OWASP-DC'

//             dependencyCheckPublisher pattern: 'dependency-check-report.xml'
//         }
//     }
    
    stage('Anchore analyse') {
      steps {
        sh '''
          curl -s https://ci-tools.anchore.io/inline_scan-v0.6.0 | bash -s -- -d Dockerfile butterflies/milestones521:latest
        '''
      }
    }
    
    stage('Push to Heroku registry') {
      steps {
        sh '''
          echo $HEROKU_API_KEY | docker login --username=_ --password-stdin registry.heroku.com
          
          docker tag butterflies/milestones521:latest registry.heroku.com/$APP_NAME/web
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
