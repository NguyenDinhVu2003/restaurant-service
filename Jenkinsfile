pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIAL')
    GITHUB_TOKEN = credentials('github-token')
    VERSION = "${env.BUILD_ID}"

  }

  tools {
    maven "Maven"
  }

  stages {

    stage('Maven Build'){
        steps{
        sh 'mvn clean package  -DskipTests'
        }
    }

     stage('Run Tests') {
      steps {
        sh 'mvn test'
      }
    }

    stage('SonarQube Analysis') {
  steps {
    sh 'mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install sonar:sonar -Dsonar.host.url=http://44.222.178.39/:9000/ -Dsonar.login=squ_32789bcdadb6e4337e432d6cbc100c2a1a14fde5'
  }
}


   stage('Check code coverage') {
            steps {
                script {
                    def token = "squ_1949abf442ade3af1e3c95f4337d9e17ced70bb5"
                    def sonarQubeUrl = "http://44.222.178.39:9000/api"
                    def componentKey = "com.codeddecode:restaurantlisting"
                    def coverageThreshold = 80.0

                    def response = sh (
                        script: "curl -H 'Authorization: Bearer ${token}' '${sonarQubeUrl}/measures/component?component=${componentKey}&metricKeys=coverage'",
                        returnStdout: true
                    ).trim()

                    def coverage = sh (
                        script: "echo '${response}' | jq -r '.component.measures[0].value'",
                        returnStdout: true
                    ).trim().toDouble()

                    echo "Coverage: ${coverage}"

                    if (coverage < coverageThreshold) {
                        error "Coverage is below the threshold of ${coverageThreshold}%. Aborting the pipeline."
                    }
                }
            }
        } 


      stage('Docker Build and Push') {
      steps {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker build -t nguyendinhvu/restaurant-server:${VERSION} .'
          sh 'docker push nguyendinhvu/restaurant-server:${VERSION}'
      }
    } 


     stage('Cleanup Workspace') {
      steps {
        deleteDir()
       
      }
    }



    stage('Update Image Tag in GitOps') {
        steps {
            script {
                sh """
                    rm -rf deployment-service
                    git clone https://${GITHUB_TOKEN}@github.com/NguyenDinhVu2003/deployment-service.git
                    cd deployment-service
                    git config user.email "jenkins@ci.com"
                    git config user.name "Jenkins CI"
                    sed -i "s|image:.*|image: nguyendinhvu/restaurant-server:${VERSION}|" aws/restaurant-manifest.yml
                    git add .
                    git commit -m "Update image tag to ${VERSION}"
                    git push
                """
            }
        }
    }

  }

}


