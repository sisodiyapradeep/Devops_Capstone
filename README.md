
# Sample App for adding event

This is a simple app that allows users to add event, comment and like. External App defines the frontend and Internal app defines the backend.


## Building and deploying the App manually

The steps below will guide you through building and deploying the App manually using Docker and Kubernetes and Jenkins Pipeline.

### Infrastructure Setup

Deploy the EKS cluster using `eksctl`. This is going to take a while, so you can continue with the next steps.

```bash
eksctl create cluster -f cluster.yaml
```

Create two ECR repositories in your AWS account:

```bash
aws ecr create-repository --repository-name external
aws ecr create-repository --repository-name internal
EXTERNAL_IMAGE = '933199133172.dkr.ecr.us-east-1.amazonaws.com/external
INTERNAL_IMAGE = '933199133172.dkr.ecr.us-east-1.amazonaws.com/internal'
```

### Application Build

Build the backend image and push to ECR,

```bash
cd internal
docker build -t ${INTERNAL_IMAGE}:${IMAGE_TAG} .
docker push ${INTERNAL_IMAGE}:${IMAGE_TAG}
```

Build the frontend image and push to ECR,

```bash
cd external
docker build -t ${EXTERNAL_IMAGE}:${IMAGE_TAG} .
docker push ${EXTERNAL_IMAGE}:${IMAGE_TAG}
```

### Running the application locally with Docker

To run the application locally using Docker, ensure you have Docker installed and then execute the following command:

```bash
docker run -d -p 8080:8080
```

### Deploy the Application as pods on kubernetes cluster (Manually/Not using pipeline)

Make sure the EKS cluster is up and running, then deploy the application using Kubernetes manifests.

Create a namespace for the application:

```bash
cd kubernestes
NAMESPACE=devops
kubectl create namespace $NAMESPACE
kubectl config set-context --current --namespace=$NAMESPACE
```

Deploy the application:

```bash
 cd kubernetes
 kubectl config set-context --current --namespace=$NAMESPACE
 kubectl apply -f ./kubernetes/internal-deployment.yaml
 kubectl apply -f ./kubernetes/external-deployment.yaml
 kubectl apply -f ./kubernetes/internal-service.yaml
 kubectl apply -f ./kubernetes/external-service.yaml
```

You can check the status of deployement, services,and pods using:

```bash
kubectl get all
```

Check the loadbalancer service to get the external IP:

### Accessing the Application

Access the frontend at the external IP address provided by the LoadBalancer service.

### Cleanup

To clean up the resources created during this setup, you can delete the namespace and the EKS cluster:

```bash
kubectl delete namespace $NAMESPACE
kubectl delete namespace devops
eksctl delete cluster -f cluster.yaml
```

## Deploy with Jenkins

To deploy the sample event App using Jenkins, follow these steps:

1. **Set up Jenkins**: Ensure you have a Jenkins server running and configured with the necessary plugins for Terraform and Kubernetes.
2. **Create a Jenkins Pipeline**: Create a Jenkins pipeline that will:
   - Clone the repository containing the sample event App code.
   - Use cluster.yaml to provision the EKS cluster and deploy the application.
   - Build and push Docker images to ECR.
   - Apply Kubernetes manifests to deploy the application.
   - **Configure Jenkins Credentials**: Ensure that Jenkins has the necessary AWS credentials and permissions to interact with EKS and ECR
   - **Configure Jenkins Environment Variables**: Set environment variables for the ECR repository URIs and other necessary configurations.
   - **Trigger the Pipeline**: Run the Jenkins pipeline to deploy the Sample event App.

Following is a sample Jenkins pipeline view,

![Jenkin Pipelines](./documentation/jenkins-pipelines.png)

Following is a sample execution of the Jenkins pipeline,

![Jenkins Execution](./documentation/jenkins-execution.png)

### Notes

1. A sample Jenkinsfile is provided in the repository to help you get started with the pipeline configuration.
2. A sample Terraform configuration is also provided to set up the EKS cluster.
# events-sample-node
 
1. Getting started:  
    1. Using the deployment server:  
        prerequisites: node and npm  
        these should be installed from a previous lab.  
        1. open two separate terminal sessions:  
            * in one terminal **cd sample-master/internal**  
            * in the other terminal **cd sample-master/external**  
        1. run **npm install** in each terminal
        1. run **node server.js** in the *internal* terminal
        1. run **npm start** in the *external* terminal
        1. Open a browser and navigate to http://localhost:8080
    1. Using Your own machine (Windows users, see note at bottom this file):  
        prerequisites: node and npm  
        1. open two separate terminal sessions:  
            * in one terminal **cd sample-master/internal**  
            * in the other terminal **cd sample-master/external**  
        1. run **npm install** in each terminal
        1. run **node server.js** in the *internal* terminal
        1. run **npm start** in the *external* terminal
        1. Open a browser and navigate to http://localhost:8080

1. The sample app  
Uses nodejs with the express web server on both server and client microservices.  
The internal service receives REST requests on port 8082 and returns mock data.  
The external service unpacks json from the internal service into a html template in the Views folder and returns it to the browser on port 8080.

1. Dependencies  
The internal and external both use the following npm packages:

   * express: a web server  
     * https://www.npmjs.com/package/express  
   * body-parser to convert json and form data in the request into parameters.  
     * https://www.npmjs.com/package/body-parser  
   * mocha, chai and supertest (for unit testing)  
     * https://www.npmjs.com/package/mocha  
     * https://www.npmjs.com/package/chai  
     * https://www.npmjs.com/package/supertest  
   * nyc for code coverage reporting  
     *  https://www.npmjs.com/package/nyc  
     
   The external service uses the following additional libraries:

   * express-handlebars ( a templating library)  
     * https://github.com/ericf/express-handlebars  
   * nock (for mocking the api call)  
     * https://www.npmjs.com/package/nock

1. Windows 
The external/package.json file uses linux-style syntax for environment variables.
Windows users will need to modify the code in order to run the sample locally during development, e.g.
    *     **"start": "set SERVER=http://localhost:8082&& node server.js"**,
    *     **"test": "set SERVER=http://localhost:8082&& nyc mocha"**
