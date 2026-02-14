pipeline {
  agent any

  environment {
    DOCKER    = "/Applications/Docker.app/Contents/Resources/bin/docker"

    APP_NAME  = "cicd-app"
    BRANCH    = "${env.BRANCH_NAME}"

    // main -> 3000, dev -> 3001
    PORT      = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"

    IMAGE     = "${APP_NAME}:${env.BRANCH_NAME}"
    CONTAINER = "${APP_NAME}-${env.BRANCH_NAME}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Debug Docker') {
      steps {
        sh '''
          echo "BRANCH_NAME=$BRANCH_NAME"
          echo "Using DOCKER=$DOCKER"
          $DOCKER --version
          $DOCKER ps >/dev/null || true
        '''
      }
    }

    stage('Build') {
      steps {
        sh '''
          $DOCKER run --rm \
            -v "$PWD":/app -w /app \
            node:20-alpine sh -lc "node -v && npm -v && npm ci && npm run build"
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          $DOCKER run --rm \
            -v "$PWD":/app -w /app \
            node:20-alpine sh -lc "npm test"
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          $DOCKER build -t "$IMAGE" .
        '''
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          $DOCKER rm -f "$CONTAINER" || true

          # Host port differs by branch:
          # main: http://localhost:3000
          # dev:  http://localhost:3001
          # Container port assumed 3000 (change the right side if your app listens on another internal port)
          $DOCKER run -d --name "$CONTAINER" -p "$PORT":3000 "$IMAGE"

          echo "✅ Deployed '$BRANCH' -> http://localhost:$PORT"
        '''
      }
    }
  }

  post {
    always {
      sh '''
        $DOCKER ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}" || true
      '''
    }
  }
}
