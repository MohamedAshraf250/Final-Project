       pipeline {
    agent any
    triggers {
        githubPush()
    }

    environment {
        DOCKER_REGISTRY = 'mohamedashraf14'
        NODE_IMAGE = "${DOCKER_REGISTRY}/back:latest"
        REACT_IMAGE = "${DOCKER_REGISTRY}/front:latest"
        ANSIBLE_INVENTORY = 'ansible/inventories/hosts.yaml'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Node.js Docker Image') {
            steps {
                script {
                    // Build Node.js Docker image
                    def nodeImage = docker.build("${NODE_IMAGE}", './backend')
                }
            }
        }

        stage('Build React Docker Image') {
            steps {
                script {
                    // Build React Docker image
                    def reactImage = docker.build("${REACT_IMAGE}", './frontend')
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-credentials') {
                        // Push Node.js image to Docker Hub
                        docker.image("${NODE_IMAGE}").push()
                        // Push React image to Docker Hub
                        docker.image("${REACT_IMAGE}").push()
                    }
                }
            }
        }

        // Uncomment and adjust the following stage if you are deploying via Ansible to Kubernetes
        /*
        stage('Deploy to Kubernetes via Ansible') {
            steps {
                ansiblePlaybook(
                    playbook: 'ansible/playbooks/deploy-app.yaml',
                    inventory: ANSIBLE_INVENTORY,
                    extras: "-e 'nodejs_image=${NODE_IMAGE} react_image=${REACT_IMAGE}'"
                )
            }
        }
        */
    }

    post {
        always {
            cleanWs() // Clean up workspace after each run
        }

        // Optional Slack notification in case of failures (uncomment if needed)
        /*
        failure {
            slackSend channel: '#devops-alerts', message: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed. Check Jenkins: ${env.BUILD_URL}"
        }
        */
    }
}
