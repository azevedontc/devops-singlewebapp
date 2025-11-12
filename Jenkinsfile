pipeline {
  agent any
  environment {
    APP_NAME   = 'singleweb'
    IMAGE_NAME = 'singleweb'
    CONTAINER  = 'singleweb'
    HOST_PORT  = '8082'   // altere se precisar
    CONT_PORT  = '80'
    DOCKER_NET = 'apps_net'
  }
  triggers { cron('20 2 * * *') }     // roda todo dia 02:20
  options  { timestamps() }

  stages {
    stage('Checkout') {
      steps {
        // se criar o job como "Pipeline from SCM", o Jenkins já faz checkout;
        // ainda assim, deixar explícito aqui não atrapalha:
        checkout scm
      }
    }

    stage('Cleanup anterior') {
      steps {
        sh '''
          set -eux
          docker rm -f $CONTAINER || true
          docker rmi -f $IMAGE_NAME:latest || true
        '''
      }
    }

    stage('Build da imagem') {
      steps {
        sh '''
          set -eux
          docker build -t $IMAGE_NAME:${BUILD_NUMBER} -t $IMAGE_NAME:latest .
        '''
      }
    }

    stage('Subir container') {
      steps {
        sh '''
          set -eux
          docker network create $DOCKER_NET || true
          docker run -d --name $CONTAINER --restart unless-stopped \
            --network $DOCKER_NET \
            -p ${HOST_PORT}:${CONT_PORT} \
            $IMAGE_NAME:latest
        '''
      }
    }

    stage('Higiene') {
      steps { sh 'docker image prune -f || true' }
    }
  }

  post {
    success {
      emailext to: 'azevedo.fidelis.silva@gmail.com',
               subject: "[OK] ${env.JOB_NAME} #${env.BUILD_NUMBER}",
               body: "Front publicado em http://SEU_IP:${HOST_PORT}\nImagem: ${IMAGE_NAME}:latest"
    }
    failure {
      emailext to: 'azevedo.fidelis.silva@gmail.com',
               subject: "[FALHA] ${env.JOB_NAME} #${env.BUILD_NUMBER}",
               body: "Ver console: ${env.BUILD_URL}"
    }
  }
}
