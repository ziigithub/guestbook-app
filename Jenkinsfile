node('Jenkins_slave_ansible_docker') {
  def DockerImage = "hezil/guestbook"
  def ContainerName = "guestbook"

  stage('Pre') { // Run pre-build steps
    cleanWs()
    sh "docker rm -f ${ContainerName} || true"
  }

  stage('Git') { // Get code from GitLab repository
    git branch: 'master',
      url: 'https://github.com/hezil/guestbook-app.git'
  }

  stage "build docker image"
  dir ('php-redis/') {
    sh "docker build --tag ${DockerImage} ."
  }

  stage('Run') { // Run the built image

    sh "docker run -d --name ${ContainerName} --rm -p 8088:80 ${DockerImage}; sleep 5"
  }

  stage('Test') { // Run tests on container
    def dockerOutput = sh (
        script: 'curl -s --head  --request GET http://172.17.0.1:8088 | grep "200 OK" | wc -l',
        returnStdout: true
        ).trim()
    sh "docker rm -f ${ContainerName}"

    if ( dockerOutput == "1" ) {
        currentBuild.result = 'SUCCESS'
    } else {
        currentBuild.result = 'FAILURE'
        sh "echo webserver returned ${dockerOutput}"
    }
    return
  }

  stage('Push') { // Run tests on container
          // This step should not normally be used in your script. Consult the inline help for details.
        withDockerRegistry(credentialsId: 'hezil_dockerhub') {
            // some block
            sh "docker push ${DockerImage}"
            sh "docker rmi ${DockerImage}"
        }
  }
  stage('Git') { // Get code from GitLab repository
    git branch: 'master',
      url: 'https://github.com/hezil/ansible_job.git'
  }
  
  stage "deploy guestbook_webapp in k8s culster"
  dir ('.') {
      sh('sudo ansible -i ./ec2.py -m ping -u ubuntu tag_Name_k8s_m1')
      sh('sudo ansible-playbook -i  ./ec2.py -l tag_Name_k8s_m1 guestbook_webapp')
  }
}
