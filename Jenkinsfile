pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'master', url: 'https://github.com/azaiz00/API-Test-Automation.git'
      }
    }

    stage('Lire URL de la collection (sans clé)') {
      steps {
        script {
          env.POSTMAN_COLLECTION_URL = (readFile('postman/collections/UserTestCollection_url.txt').replace("372bd9aab491",environnement)).trim()
        }
      }
    }

    stage('Ajouter la clé Postman (Credentials)') {
      steps {
        withCredentials([string(credentialsId: 'POSTMAN_KEY', variable: 'POSTMAN_KEY')]) {
          script {
            def sep = env.POSTMAN_COLLECTION_URL.contains('?') ? '&' : '?'
            env.POSTMAN_COLLECTION_URL = "${env.POSTMAN_COLLECTION_URL}${sep}access_key=${POSTMAN_KEY}"
          }
        }
      }
    }

    stage('Créer dossier rapports') {
      steps {
        bat 'if not exist newman\\reports mkdir newman\\reports'
      }
    }

    stage('Exécuter Newman via npx') {
      steps {
       /* nodejs('NodeJS18') {*/
          bat """
            npx --yes --package newman --package newman-reporter-htmlextra newman run ^
              "${env.POSTMAN_COLLECTION_URL}" ^
              -e postman\\environments\\PreProd.postman_environment.json ^
              --reporters cli,htmlextra ^
              --reporter-htmlextra-export newman\\reports\\report.html 
          """
        /*}*/
      }
    }
  }

  post {
    always {
      // afficher le rapport directement dans Jenkins (onglet "Newman HTML Report")
      publishHTML(target: [
        reportName: 'Newman HTML Report',
        reportDir: 'newman/reports',
        reportFiles: 'report.html',
        keepAll: true,
        alwaysLinkToLastBuild: true,
        allowMissing: true
      ])

    }
  }
}
// test add 