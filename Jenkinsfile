pipeline {
  agent any
  stages {
    stage('SCM Checkout') {
      steps {
        echo '>>> Start getting SCM code'
        git 'https://github.com/Mohamedmahrous995/mavenTest.git'
        echo 'getting SCM success'
      }
    }

    stage('build JAR File') {
      steps {
        echo 'start building maven App'
        sh 'mvn -Dmaven.test.skip=TRUE install'
        echo '>>start running jar file'
        sh "java -jar $WORKSPACE'/target/demo1-0.0.1-SNAPSHOT.jar"
        echo 'build complete'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''if docker images -a | grep "pipeline-demo*" | awk \'{print $1":"$2}\' | xargs docker rmi -f; then
            printf \'Clearing old images succeeded\\n\'
            else
            printf \'Clearing old images failed\\n\'
            fi'''
        echo '>>> End clearing old docker images'
        echo '>>> Start building App docker image'
        sh "docker build -t $MyDockerAccountName/$MyDockerReposioryName:$MyTagName$BUILD_NUMBER --pull=true $WORKSPACE"
        echo '>>> End building App docker image'
      }
    }

    stage('upload docker image') {
      steps {
        echo '>>> Login to Docker hub'
        withCredentials(bindings: [string(credentialsId: 'DockHubSecret', variable: 'DockerHubIDSecret')]) {
          sh "docker login -u $MyDockerAccountName -p $DockerHubIDSecret"
        }

        echo '>>> Start uploading the docker image'
        sh "docker push $MyDockerAccountName/$MyDockerReposioryName:$MyTagName$BUILD_NUMBER"
        echo '>>> End uploading the docker image'
      }
    }

    stage('deploy') {
      steps {
        echo 'Start Killing and Removing old continaer'
        sh '''if docker ps -a | grep "Jenkins-pipeline-Demo*" | awk \'{print $1}\' | xargs docker rm -f; then
            printf \'Clearing old containers done succeeded\\n\'
            else
            printf \'Clearing old containers failed\\n\'
            fi'''
        echo 'End Killing and Removing old continaer'
        echo 'Start running a new container of the application'
        sh "docker run -d -p 8090:8090 --name=$MyTagName$BUILD_NUMBER $MyDockerAccountName/$MyDockerReposioryName:$MyTagName$BUILD_NUMBER"
        echo 'End running a new container of the application'
      }
    }

  }
}