pipeline {
  agent any

  environment {
    IMAGE_NAME      = "sum-py:ci"
    CONTAINER_NAME  = "sum-ci-container"
    TEST_FILE_PATH  = "test_variables.txt"
  }

  stages {

    stage('Check workspace') {
      steps {
        sh 'ls -la'
      }
    }

    stage('Build') {
      steps {
        sh 'docker build -t $IMAGE_NAME .'
      }
    }

    stage('Run') {
      steps {
        // conteneur persistant pour docker exec
        sh 'docker rm -f $CONTAINER_NAME || true'
        sh 'docker run -d --name $CONTAINER_NAME $IMAGE_NAME'
      }
    }

    stage('Test') {
      steps {
        script {
          def testLines = readFile(env.TEST_FILE_PATH).trim().split("\n")

          for (line in testLines) {
            line = line.trim()
            if (!line) { continue }

            def vars = line.split("\\s+")
            def arg1 = vars[0]
            def arg2 = vars[1]
            def expected = vars[2].toBigDecimal()

            def out = sh(
              script: "docker exec $CONTAINER_NAME python /app/sum.py ${arg1} ${arg2}",
              returnStdout: true
            ).trim()

            def result = out.toBigDecimal()

            if (result == expected) {
              echo "✅ OK: ${arg1} + ${arg2} = ${result}"
            } else {
              error "❌ FAIL: ${arg1} + ${arg2} attendu=${expected} obtenu=${result}"
            }
          }
        }
      }
    }

    stage('Deploy') {
      when {
        branch 'main'
      }
      steps {
        echo "TODO: Push DockerHub (selon consignes prof)"
        // On l’active seulement quand tu me confirmes ton repo DockerHub + credentials Jenkins
      }
    }
  }

  post {
    always {
      sh 'docker rm -f $CONTAINER_NAME || true'
    }
  }
}
