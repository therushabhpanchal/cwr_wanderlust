pipeline{
    agent any
    environment{
        SONAR_HOME = tool "Sonar"
    }
    stages{
        stage("Clone Code from GitHub"){
            steps{
                git url:"https://github.com/krishnaacharyaa/wanderlust.git" , branch:"devops"
            }
        }
        stage("SonarQube Quality Analysis"){
             steps{
                withSonarQubeEnv("Sonar"){
                   sh """
                        ${SONAR_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=wanderlust \
                        -Dsonar.projectKey=wanderlust
                    """
                }
            }
        }
        stage("OWASP Dependency Check"){
             steps{
                //withCredentials([string(credentialsId:'NVD_API_KEY',variable:'NVD_KEY')]) --nvdApiKey ${NVD_KEY}  --nvdApiDelay 5000{}
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern:'**/dependency-check-report.xml' 
            }
        }
        stage("Sonar Quality Gate Scan"){
            steps{
                timeout(time:2,unit:"MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Trivy File System Scan"){
             steps{
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage("Deploy Using Docker compose"){
            steps{
                sh "docker-compose up -d"
            }
        }
    }
}