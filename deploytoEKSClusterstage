
stage('Deploy to EKS Cluster') {
    steps {
        script {
            // Set the Kubernetes cluster context
            sh "kubectl config use-context nsws-prod-cluster"

            // Set the Kubernetes namespace
            sh "kubectl create namespace prod --dry-run=client -o yaml | kubectl apply -f -"

            // Deploy the application to the EKS cluster
            sh "kubectl apply -f path/to/kubernetes/deployment.yaml -n prod"

            // Wait for the deployment to be ready
            sh "kubectl rollout status deployment/workshop-threetier-backend -n prod"
        }
    }
}




## Explanation:

Set Kubernetes Cluster Context:

Use the kubectl config use-context command to set the Kubernetes cluster context to nsws-prod-cluster.
Set Kubernetes Namespace:

Use the kubectl create namespace command to create the prod namespace if it doesn't exist.
Deploy the Application:

Use the kubectl apply -f command to apply the Kubernetes manifest file for your deployment. Replace path/to/kubernetes/deployment.yaml with the actual path to your Kubernetes deployment manifest.
Wait for Deployment to be Ready:

Use the kubectl rollout status command to wait for the deployment to be ready in the prod namespace.
Make sure to replace path/to/kubernetes/deployment.yaml with the actual path to your Kubernetes deployment manifest, and adjust any other deployment configurations as needed.

This assumes you have a Kubernetes deployment manifest that defines your application deployment. Adjustments may be needed based on your specific deployment setup and requirements.





