pipeline {
  agent { label 'linux' }

  options {
    timestamps()
    timeout(time: 20, unit: 'MINUTES')
    buildDiscarder(logRotator(
      numToKeepStr: '10',
      daysToKeepStr: '30',
    ))
  }

  parameters {
    choice(
      name: 'IMAGE_TAG',
      description: 'Name of Docker tag to push. Chose wisely.',
      choices: genChoices(params.IMAGE_TAG, ['latest', 'deploy-app-dev', 'deploy-mod-dev', 'deploy-app-test', 'deploy-mod-test']),
    )
    string(
      name: 'IMAGE_NAME',
      description: 'Name of Docker image to push.',
      defaultValue: params.IMAGE_NAME ?: 'statusteam/spiffworkflow-connector',
    )
  }

  stages {
    stage('Build') {
      steps { script {
        image = docker.build(
          "${params.IMAGE_NAME}:${env.GIT_COMMIT.take(8)}",
          "--label=commit='${env.GIT_COMMIT.take(8)}' ."
        )
      } }
    }

    stage('Push') {
      steps { script {
        withDockerRegistry([credentialsId: "dockerhub-statusteam-auto", url: ""]) {
          image.push()
          image.push(env.IMAGE_TAG)
        }
      } }
    }
  } // stages
  post {
    always { sh 'docker image prune -f' }
  } // post
} // pipeline

/* Helper that generates list of available choices for a parameter
 * but re-orders them based on the currently set value. First is default. */
def List genChoices(String previousChoice, List defaultChoices) {
  if (previousChoice == null) {
     return defaultChoices
  }
  choices = defaultChoices.minus(previousChoice)
  choices.add(0, previousChoice)
  return choices
}
