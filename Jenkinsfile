@Library('Shared') _
pipeline {
    agent {label 'agent-1'}
    
    environment{
        SONAR_HOME = tool "Sonar"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    }
    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }
        stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        
        stage('Git: Code Checkout') {
            steps {
                script{
                    code_checkout("https://github.com/Akash-Verma-11/Wanderlust-Mega-Project.git","main")
                }
            }
        }
        
        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("Sonar","wanderlust","wanderlust")
                }
            }
        }
        
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        
        stage('Exporting environment variables') {
            parallel{
                stage("Backend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatebackendnew.sh"
                            }
                        }
                    }
                }
                
                stage("Frontend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatefrontendnew.sh"
                            }
                        }
                    }
                }
            }
        }
        
        stage('Docker: Build Images') {
            parallel {
                stage('Build Backend') {
                    steps {
                        dir('backend') {
                            script {
                                docker_build(
                                    imageName: 'akash11v/wanderlust-backend-beta',
                                    imageTag: params.BACKEND_DOCKER_TAG
                                )
                            }
                        }
                    }
                }
                stage('Build Frontend') {
                    steps {
                        dir('frontend') {
                            script {
                                docker_build(
                                    imageName: 'akash11v/wanderlust-frontend-beta',
                                    imageTag: params.FRONTEND_DOCKER_TAG
                                )
                            }
                        }
                    }
                }
            }
        }
        
        stage('Docker: Push to DockerHub') {
            parallel {
                stage('Push Backend') {
                    steps {
                        script {
                            docker_push(
                                imageName: 'akash11v/wanderlust-backend-beta',
                                imageTag: params.BACKEND_DOCKER_TAG,
                                docker: 'docker-hub-credentials'
                            )
                        }
                    }
                }
                stage('Push Frontend') {
                    steps {
                        script {
                            docker_push(
                                imageName: 'akash11v/wanderlust-frontend-beta',
                                imageTag: params.FRONTEND_DOCKER_TAG,
                                docker: 'docker-hub-credentials'
                            )
                        }
                    }
                }
            }
        }
    }
    post{
        success{
            // Added allowEmptyArchive: true to prevent build failure
            archiveArtifacts artifacts: '*.xml', followSymlinks: false, allowEmptyArchive: true
            
            build job: "Wanderlust-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
            ]
        }
    }
}
