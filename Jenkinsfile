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
        sh 'docker build -t darinpope/java-web-app:latest .'
      }
    }
    stage ('OWASP Dependency-Check Vulnerabilities') {
        steps {
            dependencyCheck additionalArguments: ''' 
                -o "./" 
                -s "./"
                -f "ALL" 
                --prettyPrint''', odcInstallation: 'OWASP-DC'

            dependencyCheckPublisher pattern: 'dependency-check-report.xml'
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
    stage('Anchore analyse') {
      steps {
//         writeFile file: 'anchore_images', text: 'docker.io/darinpope/java-web-app:latest'
//         anchore name: 'anchore_images'
//         sh 'apk add bash curl'
        sh 'curl -s https://ci-tools.anchore.io/inline_scan-v0.6.0 | bash -s -- -d Dockerfile -b .anchore_policy.json darinpope/java-web-app:latest'
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
