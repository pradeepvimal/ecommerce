@Library('Shared') _

pipeline {
    agent any
    
    environment {
        // Update the main app image name to match the deployment file
        DOCKER_IMAGE_NAME = 'vimalpradeep/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'vimalpradeep/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GITHUB_CREDENTIALS = credentials('github-credential')
        GIT_BRANCH = "master"
    }
    
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    clean_ws()
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                script {
                    clone("https://github.com/pradeepvimal/ecommerce.git","master")
                }
            }
        }
        
        stage('Build Docker Images') {
            parallel {
                stage('Build Main App Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'Dockerfile',
                                context: '.'
                            )
                        }
                    }
                }
                
                stage('Build Migration Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'scripts/Dockerfile.migration',
                                context: '.'
                            )
                        }
                    }
                }
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                script {
                    run_tests()
                }
            }
        }
        
        stage('Security Scan with Trivy') {
            steps {
                script {
                    // Create directory for results
                  
                    trivy_scan()
                    
                }
            }
        }
        
        // stage('Push Docker Images') {
        //     parallel {
        //         stage('Push Main App Image') {
        //             steps {
        //                 script {
        //                     docker_push(
        //                         imageName: env.DOCKER_IMAGE_NAME,
        //                         imageTag: env.DOCKER_IMAGE_TAG,
        //                         credentials: 'docker-hub-credentials'
        //                     )
        //                 }
        //             }
        //         }
                
        //         stage('Push Migration Image') {
        //             steps {
        //                 script {
        //                     docker_push(
        //                         imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
        //                         imageTag: env.DOCKER_IMAGE_TAG,
        //                         credentials: 'docker-hub-credentials'
        //                     )
        //                 }
        //             }
        //         }
        //     }
        // }

       stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'docker logout'
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                }
            }
        }

        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: '' // No need for credentials here
                            )
                        }
                    }
                }
                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: '' // No need for credentials here
                            )
                        }
                    }
                }
            }
        }
        
        // Add this new stage
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'github-credential',
                        gitUserName: 'Jenkins CI',
                        gitUserEmail: 'pkv2000_1@yahoo.com'
                    )
                }
            }
        }
    }
}
