
pipeline {
  agent any

  environment {
    APP_NAME   = "cicd-app"
    BRANCH     = "${env.BRANCH_NAME}"
    PORT       = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
    IMAGE      = "${APP_NAME}:${env.BRANCH_NAME}"
    CONTAINER  = "${APP_NAME}-${env.BRANCH_NAME}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        sh '''
          docker run --rm \
            -v "$PWD":/app -w /app \
            node:20-alpine sh -lc "node -v && npm -v && npm ci && npm run build"
        '''
      }
    }
    
    stage('Test') {
      steps {
        sh '''
          docker run --rm \
            -v "$PWD":/app -w /app \
            node:20-alpine sh -lc "npm test"
        '''
      }
    }


    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE .'
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          docker rm -f $CONTAINER || true
          # Host port differs by branch, container port is usually 3000
          docker run -d --name $CONTAINER -p $PORT:3000 $IMAGE
          echo "Deployed $BRANCH on http://localhost:$PORT"
        '''
      }
    }
  }
}
