pipeline {
  agent {
    node { label "docker-host" }
  }

  environment {
    GIT_NAME = "eea.docker.nextcloud"
    registry = "eeacms/nextcloud"
  }

  stages {

    // Each Nextcloud major version lives in its own numbered folder (24/,
    // 25/, ..., 30/), one Dockerfile per folder. Releases are tagged
    // "<nextcloud-version>-apache" (e.g. 30.0.17-apache), matching the
    // existing eeacms/nextcloud Docker Hub tags; the folder to build is just
    // the tag's leading version number, so it's derived from the tag rather
    // than kept in a second place.
    stage('Docker build and push on tag') {
      when {
        buildingTag()
      }
      steps {
        script {
          def folder = env.TAG_NAME.tokenize('.')[0]
          try {
            def dockerImage = docker.build("$registry:$TAG_NAME", "--pull --no-cache -f ${folder}/Dockerfile ${folder}")
            docker.withRegistry('', 'eeajenkins') {
              dockerImage.push()
            }
          }
          finally {
            sh "docker rmi $registry:$TAG_NAME || true"
          }
        }
      }
    }

  }

  post {
    always {
      cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenSuccess: true, cleanWhenUnstable: true, deleteDirs: true)
      script {
        def url = "${env.BUILD_URL}/display/redirect"
        def status = currentBuild.currentResult
        def subject = "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
        def summary = "${subject} (${url})"
        def details = """<h1>${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${status}</h1>
                         <p>Check console output at <a href="${url}">${env.JOB_BASE_NAME} - #${env.BUILD_NUMBER}</a></p>
                      """
        emailext(
          subject: summary,
          body: details,
          attachLog: true,
          compressLog: true,
          recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'CulpritsRecipientProvider']]
        )
      }
    }
  }
}
