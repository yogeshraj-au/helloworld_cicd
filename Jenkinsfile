pipeline {
    agent {
        label 'docker'
    }

    
    environment {
        DOCKER_USERNAME = 'imageimpressario'
        GITHUB_USERNAME= 'yogshraj-au'
        REPO1_NAME = 'nginx_hello_world'
        REPO2_NAME = 'nginx_hello'
        valuesFile = "/var/jenkins_home/workspace/nginx/charts/nginx-hello-world/values.yaml"
    }
    
    stages {
        stage('Clone Repositories') {
            steps {
                // Clone Repository 1
                git branch: 'main', url: "https://github.com/yogeshraj-au/nginx_hello_world.git"
                    
                // Clone Repository 2
                git branch: 'main', url: "https://github.com/yogeshraj-au/nginx_hello.git"
            }
        }
        
        stage('Build and Tag Repository 1') {
            when {
                anyOf {
                    changeset "**/${env.REPO1_NAME}/**"
                    changeset "**/*"
                }
            }
            steps {
                script {
                    // Build Repository 1
                    docker.build("${env.DOCKER_USERNAME}/${env.REPO1_NAME}")
                    
                    // Tag Repository 1 based on Git commit
                    docker.image("${env.DOCKER_USERNAME}/${env.REPO1_NAME}").tag("${env.BUILD_NUMBER}")
                    
                    // Push Repository 1 to Docker Hub
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("${env.DOCKER_USERNAME}/${env.REPO1_NAME}").push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        
        stage('Build and Tag Repository 2') {
            when {
                anyOf {
                    changeset "**/${env.REPO2_NAME}/**"
                    changeset "**/*"
                }
            }
            steps {
                script {
                    // Build Repository 2
                    docker.build("${env.DOCKER_USERNAME}/${env.REPO2_NAME}")
                    
                    // Tag Repository 2 based on Git commit
                    docker.image("${env.DOCKER_USERNAME}/${env.REPO2_NAME}").tag("${env.BUILD_NUMBER}")
                    
                    // Push Repository 2 to Docker Hub
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("${env.DOCKER_USERNAME}/${env.REPO2_NAME}").push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('clone cicd repo') {
            steps {
                git branch: 'main', url: "https://github.com/yogeshraj-au/helloworld_cicd.git"
            }
        }

        stage('Update Helm Values') {
            steps {
                script {
                    //sh "sleep 5000"
                     // Check if the file exists
                    def valuesFile = "${WORKSPACE}/charts/nginx-hello-world/values.yaml"

                    // Create a File object
                    def file = new File(valuesFile)

                    // Check if the file exists
                    if (file.exists()) {
                        echo "File $valuesFile exists. Deleting..."

                        // Delete the file
                        file.delete()
                    }

                    // Read existing YAML data (if needed)
                    def helmValues = readYaml file: valuesFile

                    // Update tag for nginxhello image
                    helmValues.nginxhello.image.tag = "${env.BUILD_NUMBER}" // Update the tag as needed

                    // Write updated values back to values.yaml
                    writeYaml file: valuesFile, data: helmValues
                }
            }
        }

        stage('Package Helm Chart') {
            steps {
                script {
                    // Package Helm chart after updating values
                    sh "helm package charts/${env.HELM_CHART_NAME}"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Build and Push to Docker Hub successful!'
        }
        failure {
            echo 'Build and Push to Docker Hub failed!'
        }
    }
}
