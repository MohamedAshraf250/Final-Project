pipeline {
    agent any

    environment {
        // Define your environment variables here
        KUBE_CONFIG = credentials('kubeconfig') // Jenkins credentials ID for kubeconfig
        ANSIBLE_HOST_KEY_CHECKING = 'False' // Disable host key checking
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from your repository
                git 'https://github.com/your-repo/your-project.git' // Change this to your repo URL
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install any dependencies, e.g., Ansible, if not installed
                sh 'pip install ansible' // Ensure Ansible is installed
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Set up kubectl with the kubeconfig
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG_FILE')]) {
                    sh 'export KUBECONFIG=$KUBE_CONFIG_FILE'
                    
                     // Apply the frontend deployment
                    sh 'kubectl apply -f k8s/frontend-deployment.yaml -n myapp'

                    // Apply the backend deployment
                    sh 'kubectl apply -f k8s/backend-deployment.yaml -n myapp'
                }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                // Run the Ansible playbook to configure or manage the deployment
                sh 'ansible-playbook ansible.yaml'
            }
        }

        stage('Post-deployment Validation') {
            steps {
                // Validate that the deployment was successful, e.g., check service availability
                script {
                    def serviceIP = sh(script: "kubectl get svc frontend-service -n myapp -o jsonpath='{.status.loadBalancer.ingress[0].ip}'", returnStdout: true).trim()
                    def serviceHostname = sh(script: "kubectl get svc frontend-service -n myapp -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'", returnStdout: true).trim()

                    if (serviceIP) {
                        echo "Service URL: http://${serviceIP}:80"
                    } else if (serviceHostname) {
                        echo "Service URL: http://${serviceHostname}:80"
                    } else {
                        error "Service is not yet assigned an External IP or Hostname."
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed. Check the logs for details.'
        }
    }
}
