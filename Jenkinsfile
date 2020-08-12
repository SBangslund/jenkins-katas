pipeline {
  agent any
  environment {
    docker_username = 'saban17'
  }
  stages {
    stage('Clone Down') {
      steps {
        stash(excludes: '.git/', name: 'code')
      }
    }

    stage('Parallel Execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('Build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          post {
            always {
              sh 'ls'
              deleteDir()
              sh 'ls'
            }

          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash 'code'
          }
          options {
            skipDefaultCheckout(true)
          }
        }

        stage('Test app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          post {
            always {
              sh 'ls'
              deleteDir()
              sh 'ls'
            }

          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
      }
    }
    stage('Push Docker app') {
      when { branch "master" }
      environment {
        // Retrieve credentials from the Jenkins server
        DOCKERCREDS = credentials('docker_login')
      }
      steps {
        unstash 'code'
        // Run the build-docker script
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
        sh 'ci/push-docker.sh'
      }
    }

    // Should not run on dev/ branches
    stage('Component test') {
      when { not { branch 'dev/' } 
        beforeAgent true
      }
      steps {
        sh 'ci/component-test.sh'
      }
    }
  }
}