node() { 
  def redisImage = "redis"
  def feImage = "fe"


  stage('Pre') { // Run pre-build steps, stop and remove the already running containers
    cleanWs()
    sh 'docker ps -f name=frontend -q | xargs --no-run-if-empty docker container stop'
    sh 'docker container ls -a -fname=frontend -q | xargs -r docker container rm'
    sh 'docker ps -f name=redis-master -q | xargs --no-run-if-empty docker container stop'
    sh 'docker container ls -a -fname=redis-master -q | xargs -r docker container rm'
  }

  
  stage('Git') { // Get code from GitLab repository

    git branch: 'master',

      url: 'https://github.com/shlomisasportas/MidPro-Jenkins.git'

  }

  stage('Build') { // Run the docker build

    sh "docker build --tag $redisImage ."
    sh "docker build --tag $feImage ."

  }

  stage('Run') { // Run the built image

    sh "docker run -d --name fe --rm -p 8081:5000 $feImage; sleep 5"
    sh "docker run -d --name redis --rm -p 8081:5000 $redisImage; sleep 5"

  }

  

  stage('Test') { // Run tests on container

    def dockerOutput = sh (

        script: 'curl http://192.168.15.30:8081/goaway',

        returnStdout: true

        ).trim()


    if ( dockerOutput == 'GO AWAY!' ) {

        currentBuild.result = 'SUCCESS'

    } else {

        currentBuild.result = 'FAILURE'

        sh "echo Webserver returned ${dockerOutput}"

    }

    return

  }

 stage('Push') { // Push the built image to docker hub

    withDockerRegistry(credentialsId: 'docker_hub') {

        sh "docker tag $feImage shlomisasportas/$feImage"
        sh "docker push shlomisasportas/$feImage"
        sh "docker tag $redisImage shlomisasportas/$redisImage"
        sh "docker push shlomisasportas/$redisImage"

    }

 }

}
