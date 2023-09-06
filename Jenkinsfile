pipeline {
  agent any

  options {
    timeout(time: 2, unit: 'MINUTES')
  }

  environment {
    ARTIFACT_ID = "elbuo8/webapp:${env.BUILD_NUMBER}"
  }

  stages {
    stage('Build') {
      steps{
        sh "docker build -t testapp ."
      }
    }
    stage('Run tests') {
      steps {
        sh "docker run testapp npm test"
      }
    }
    stage('Deploy Image') {
      steps{
        sh '''
        docker tag testapp 127.0.0.1:5000/grupo5/testapp
        docker push 127.0.0.1:5000/grupo5/testapp
        '''
        }
    }
    stage('Vulnerability Scan - Docker ') {
      steps {
          script {
            // Verificar el resultado del análisis de seguridad con Trivy
            def trivyResult = sh(script: "docker run  -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --severity=CRITICAL 127.0.0.1:5000/grupo5/testapp", returnStatus: true)
            echo "RESULTADO DEL ANALISIS ${trivyResult}"
            if (trivyResult == 0) {
                echo "ESTADO OK: SE EJECUTA LA APP"
            } else {
                echo "DENEGADO: Imagen tiene vulnerabilidades según Trivy"
            }
          }
      }
    }
    // stage('Pass To K8s'){
    //   steps {
    //     sh '''
    //       sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "kubectl create deployment testapp --image=127.0.0.1:5000/mguazzardo/testapp"
    //       echo "Wait"
    //       sleep 10
    //       sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "kubectl expose deployment testapp --port=3000"
    //       sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "wget https://raw.githubusercontent.com/tercemundo/platzi-scripts-integracion/master/webapp/nodePort.yml"
    //       sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "kubectl apply -f nodePort.yml" 
    //     '''
    //   }
    // }
    stage('Run app') {
      steps {
        sh '''
          docker run -d -p 3000:3000 127.0.0.1:5000/grupo5/testapp
        '''
      }
    }
  }
}