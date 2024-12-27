This repository contains a CI/CD pipeline for building, analyzing, and pushing Docker images. 
The pipeline is designed to automate the entire process from code compilation to Docker image deployment.


![image](https://github.com/user-attachments/assets/18525933-5ad7-4da0-9378-3262f3343773)



![image](https://github.com/user-attachments/assets/6ab8ce77-81ca-429a-a13d-586137a69e6f)

![image](https://github.com/user-attachments/assets/3445bb12-d5a0-4089-bfac-a3bd83dfb571)

## Pipeline Stages

The pipeline includes the following stages:

1. **Tool Install**  
   Installs all the necessary tools and dependencies required for the pipeline.

2. **Git Checkout**  
   Pulls the latest code from the repository.

3. **Compile**  
   Compiles the source code and ensures the build is successful.

4. **SonarQube Analysis**  
   Runs SonarQube to analyze code quality and identify potential issues.

5. **OWASP**  
   Performs a security scan to detect vulnerabilities in the code.

6. **Build**  
   Builds the application and prepares it for Docker packaging.

7. **Docker Build and Push**  
   - Builds the Docker image.  
   - Pushes the Docker image to the configured container registry.

---

## Key Features

- **JDK 17**: The pipeline uses JDK 17 for building and running the application.
- **Code Quality**: Integrated with SonarQube for static code analysis.
- **Security**: Includes OWASP security scans to ensure a secure codebase.
- **Efficiency**: Completes in less than 2 minutes for most builds.

---

## Usage

### Clone the Repository
```bash
git clone <repository-url>
cd <repository-name>
Configure the Pipeline
Update the configuration files for your CI/CD tool (e.g., Jenkins, GitHub Actions, etc.) with the appropriate credentials and environment settings.

Trigger the Pipeline
Push your changes to the repository. The pipeline will automatically run.

Logs and Debugging
Detailed logs for each stage of the pipeline can be accessed via the CI/CD tool's dashboard. For example:

Docker Build and Push: Took 20 seconds to complete successfully.
To view logs as plain text, use the "View as plain text" link in the respective stage.

Contributing
Feel free to submit issues or pull requests to enhance the pipeline.

Acknowledgments
SonarQube for code analysis.
OWASP Dependency Check for security scanning.


Groovy scripting:

pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/akshaya1993/docker_home_page.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                sh '''
                    $SCANNER_HOME/bin/sonar-scanner -X \
                    -Dsonar.url=http://35.92.163.159:9000/ \
                    -Dsonar.login=squ_a7d87c6ad34a66e212c357cff47c9944927a3651 \
                    -Dsonar.projectName=dockerpublish \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=dockerpublish
                '''
            }
        }
        stage('OWASP') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean install"
            }
        }
        stage('Docker Build and Push') {
            steps {
	    script{
                withDockerRegistry(credentialsId: 'docker') {
                    sh "docker build -t dockerpublish ."
                    sh "docker tag dockerpublish vaidikprabhu/dockerpublish:latest"
                    sh "docker push vaidikprabhu/dockerpublish:latest"
                }
              }
            }
        }

 stage('Docker RUN') {
            steps {
	    script{
                withDockerRegistry(credentialsId: 'docker') {
                    sh "docker run -d  -p 8081:8081 --name dockerpublish vaidikprabhu/dockerpublish:latest"                
              }
              }
            }
        }

    }
}


