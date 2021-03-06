pipeline {
  agent none

  environment {
    MAJOR_VERSION = 1
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '1'))
  }

  stages {
    stage('Say hello') {
      agent any
      steps {
        sayHello "awesome developer"
      }
    }

    stage('Git information') {
      agent any
      steps {
        echo "My Branch Name: ${env.BRANCH_NAME}"
        script {
          def myLib = new linuxacademy.git.gitStuff();
          echo "My Commit: ${myLib.gitCommit("${env.WORKSPACE}/.git")}"
        }
      }
    }

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
        sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
      }
    }

    // stage('Test on Mac OSX') {
    //   agent {
    //     label 'osx'
    //   }
    //   steps {
    //     sh "curl https://d23ca0c4.ngrok.io/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar -o rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
    //     sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
    //   }
    // }  

    stage('Test on Debian') {
      agent {
        docker 'openjdk:8u151-jre'
      }
      steps {
        sh "wget https://d23ca0c4.ngrok.io/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
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
        sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
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
        echo "Checking out of the development branch"
        sh "git checkout development"
        echo "Pulling the development branch"
        sh "git pull"
        echo "Checking out of the master branch"
        sh "git checkout master"
        echo "Merging development branch into master branch"
        sh "git merge development"
        echo "Pushing to origin master"
        sh "git push origin master"
        echo "Tagging the release"
        sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
        sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
      }
      post {
        success {
          emailext(
            subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Development Branch Promoted to Master.",
            body: """<p><b>${env.JOB_NAME} [${env.BUILD_NUMBER}]</b> - Development Branch Promoted to Master:</p>
            <p>Check console output at <a href='${env.BUILD_URL}/console'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>.</p>""",
            to: "lucas.zanferrari@gmail.com"
          )
        }
      }
    }
  }

  post {
    failure {
      emailext(
        subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!",
        body: """<p><b>${env.JOB_NAME} [${env.BUILD_NUMBER}]</b> Failed!</p>
        <p>Check console output at <a href='${env.BUILD_URL}/console'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>.</p>""",
        to: "lucas.zanferrari@gmail.com"
      )
    }
  }
}
