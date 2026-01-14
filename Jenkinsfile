pipeline {
  agent any

  environment {
    APP_PORT = "8081"
    HEALTH_PATH = "/api/hello"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'mvn -B -DskipTests package'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('Start App') {
      steps {
        // Start the produced jar in background and write pid to app.pid
        sh '''
          JAR=$(ls target/*.jar | head -n 1)
          if [ -z "$JAR" ]; then echo "No jar found"; exit 1; fi
          nohup java -jar "$JAR" > app.log 2>&1 &
          echo $! > app.pid
          sleep 1
          echo "Started $JAR (pid $(cat app.pid))"
        '''
      }
    }

    stage('Wait For Server') {
      steps {
        // Poll the health endpoint until it succeeds or timeout
        sh '''
          URL="http://localhost:${APP_PORT}${HEALTH_PATH}"
          echo "Waiting for ${URL} ..."
          timeout=60
          elapsed=0
          while ! curl -sSf "$URL" >/dev/null; do
            sleep 2
            elapsed=$((elapsed+2))
            if [ $elapsed -ge $timeout ]; then
              echo "Server did not start within ${timeout}s"
              cat app.log || true
              exit 1
            fi
          done
          echo "Server is up"
        '''
      }
    }

    stage('Functional Smoke Test') {
      steps {
        sh 'curl -sfS http://localhost:${APP_PORT}${HEALTH_PATH} | sed -n "1,200p"'
      }
    }

    stage('Stop App') {
      steps {
        sh '''
          if [ -f app.pid ]; then
            kill $(cat app.pid) || true
            rm -f app.pid
            echo "Stopped app"
          else
            echo "No app.pid found, attempting to kill java processes"
            pkill -f 'java -jar' || true
          fi
        '''
      }
    }
  }

  post {
    always {
      sh 'echo "---- Logs (last 200 lines) ----"; tail -n 200 app.log || true'
      sh 'if [ -f app.pid ]; then kill $(cat app.pid) || true; rm -f app.pid; fi'
      junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
    }
    failure {
      sh 'echo "Build failed — dumping app.log"; cat app.log || true'
    }
  }
}
