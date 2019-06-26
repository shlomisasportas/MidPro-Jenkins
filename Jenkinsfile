node() { 
  def DockerImage = "flask:v1.1"
  
  stage('Pre') { // Run pre-build steps
    cleanWs()
    sh "docker rm -f web || true"
  }
  
  stage('Git') { // Get code from GitLab repository
    git branch: 'master',
      url: 'https://github.com/shlomisasportas/MidPro-Jenkins.git'
  }
  
  stage('Build') { // Run the docker build
    sh "docker build --tag ${DockerImage} ."
  }
  stage('Run') { // Run the built image
    sh "docker run -d --name web --rm -p 4000:80 ${DockerImage}; sleep 5"
  }
  
  stage('Test') { // Run tests on container
    def dockerOutput = sh (
        script: 'curl http://localhost:4000/',
        returnStdout: true
        ).trim()

    if ( dockerOutput == 'Hello World!' ) {
        currentBuild.result = 'SUCCESS'
    } else {
        currentBuild.result = 'FAILURE'
        sh "echo Webserver returned ${dockerOutput}"
    }
    return
  }
 stage('Push') { // Push the built image to docker hub
    withDockerRegistry(credentialsId: 'docker_hub') {
        sh "docker tag ${DockerImage} shlomis92/${DockerImage}"
        sh "docker push shlomis92/${DockerImage}"
    }
 }
}
