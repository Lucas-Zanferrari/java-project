pipeline {
  agent none

  options {
    buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '1'))
  }

  stages {

    stage('Unit tests') {
      agent {
        label 'apache'
      }
      steps {
        sh 'ant -f test.xml -v'
        junit 'reports/result.xml'
      }
    }

    stage('Build') {
      agent {
        label 'apache'
      }
      steps {
        sh 'ant -f build.xml -v'
      }
      post {
        success {
          archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
        }
      }
    }

    stage('Deploy') {
      agent {
        label 'apache'
      }
      steps {
        sh "if [ ! -d '/var/www/html/rectangles/all/${env.BRANCH_NAME}/' ]; then\nmkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}/\nfi"
        sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
      }
    }

    // stage('Test on Mac OSX') {
    //   agent {
    //     label 'osx'
    //   }
    //   steps {
    //     sh "curl https://9594d372.ngrok.io/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar -o rectangle_${env.BUILD_NUMBER}.jar"
    //     sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
    //   }
    // }  

    stage('Test on Debian') {
      agent {
        docker 'openjdk:8u151-jre'
      }
      steps {
        sh "wget https://9594d372.ngrok.io/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
      }
    }

    stage('Promote to green') {
      agent {
        label 'apache'
      }
      when {
        branch 'master'
      }
      steps {
        sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.BUILD_NUMBER}.jar"
      }
    }

    stage('Promote development branch to master') {
      agent {
        label 'apache'
      }
      when {
        branch 'development'
      }
      steps {
        echo "Stashing local changes"
        sh "git stash"
        echo "Checking out the development branch"
        sh "git checkout development"
        echo "Checking out the master branch"
        sh "git checkout master"
        echo "Merging development branch into master branch"
        sh "git merge development"
        echo "Pushing to origin master"
        sh "git push origin master"
      }
    }
    
  }
  
}