pipeline {
    agent any
    
    parameters{
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker tag')
    }
    
    tools {
        
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ShetManoj/Multi-Tier-BankApp-CI.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=bankapp -Dsonar.projectKey=bankapp -Dsonar.java.binaries=target'''
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-globalsettings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy -DskipTests=true'
                }
                
            }
        }
        stage('Docker Build and Tag Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker build -t 1manojshet/multitierbankapp:${params.DOCKER_TAG} ."
                    }
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker push 1manojshet/multitierbankapp:${params.DOCKER_TAG}"
                    }
                }
            }
        }
        
       stage('Update YAML File') {
    steps {
        script {
            withCredentials([gitUsernamePassword(credentialsId: 'git-credentials', gitToolName: 'Default')]) {
                sh '''
                    # Remove existing directory if it exists
                    if [ -d "Multi-Tier-BankApp-CD" ]; then
                        rm -rf Multi-Tier-BankApp-CD
                    fi
                    
                    # Clone repository 
                    git clone https://github.com/ShetManoj/Multi-Tier-BankApp-CD.git
                    cd Multi-Tier-BankApp-CD
                    
                    # List the contents of the bankapp directory
                    ls -l bankapp
                    
                    # Update the image version in the YAML file
                    sed -i 's|image: 1manojshet/multitierbankapp:.*|image: 1manojshet/multitierbankapp:'${DOCKER_TAG}'|' bankapp/bankapp-ds.yml

                    # Display the updated YAML file
                    cat bankapp/bankapp-ds.yml
                '''
                
                // Set Git user details
                sh '''
                    cd Multi-Tier-BankApp-CD
                    git config user.email "manojrsnet@gmail.com"
                    git config user.name "ShetManoj"
                '''
                
                // Add, commit, and push the changes to Git
                sh '''
                    cd Multi-Tier-BankApp-CD
                    git add bankapp/bankapp-ds.yml 
                    git commit -m "Updated image ${DOCKER_TAG}"
                    git push origin main 
                '''    
            }
        }
    }
}

        
        
        
    }
    
}
