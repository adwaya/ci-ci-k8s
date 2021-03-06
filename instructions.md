# Lab instructions

## Deploying to Kubernetes with Jenkins Pipelines


### Introduction
- Jenkins Pipelines provides a powerful way to implement a continuous delivery process, and Kubernetes provides incredible benefits in the management and orchestration of containers. In this learning activity, you will learn how to leverage both of these technologies together. You will integrate a Jenkins pipeline with Kubernetes orchestration by deploying to Kubernetes as part of a Jenkins pipeline.

- On the hands-on lab page, locate the Jenkins Server Public IP and copy it to your clipboard. 
- Open a new tab in your browser and paste the IP address, followed by the port number 8080:`<JENKINS_SERVER_PUBLIC_IP>;:8080`
- Using this same IP address, open a terminal window to connect to the server using SSH:`ssh cloud_user@<JENKINS_SERVER_PUBLIC_IP>;`
- We need to show the password for the admin user to log in to our Jenkins web interface:

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
- Copy the string that is output and paste it into the Administrator password field in your browser. Click Continue.

- For the Create First Admin User form, provide the following information:

Username: jenkins
Password: random
Confirm password: random
Full name: jenkins
Email address: test@test.com
- Click Save and continue. Next, click Start using Jenkins.

Add Docker Hub Credentials in Jenkins
Click Credentials in the menu on the left of the page and then click global. Click Add Credentials in the menu on the left of the page. Provide the following information:

Note: You will need a DockerHub account for this step.
```
Username: Provide your DockerHub username
Password: Provide your DuckerHub password
ID: docker_hub_login
Description: Docker Hub Login
```
- Click OK.
- Click Add Credentials in the menu on the left of the page.

- Back in the Jenkins tab in your browser, add your GitHub information as follows:
```
Username: Provide your GitHub username
Password: Paste the API token from your clipboard.
ID: github_key
Description: GitHub Key
```
- Click OK.

- Add the Kubeconfig from the Kubernetes master as a credential in Jenkins
- We will need to view the contents of our Kubeconfig for this step. Log in to the Kubernetes master node by navigating to the hands-on lab page, copying the Kubernetes Master Public IP, and using the credentials for that instance to log in via SSH:

`ssh cloud_user@<KUBERNETES_MASTER_PUBLIC_IP>;`
- Next, display the contents of our Kubeconfig:

`cat ~/.kube/config`
- Copy the output of this file to your clipboard. We will need to paste this into Jenkins, so navigate back to the Jenkins tab in your browser.

- Click Add Credentials in the menu on the left of the page.

- Add credentials with the following information:
```
Kind: Kubernetes configuration (kubeconfig)
ID: kubeconfig
Description: Kubeconfig
Kubeconfig: Enter directly
Content: Paste the contents of ~/.kube/config
```
- Click OK.

- Create a Jenkins project called train-schedule and successfully run the build from your GitHub fork
- Navigate to the Jenkins home page and click New Item in the menu on the left of the page.

- The name of our project will be "train-schedule" and it will be a Multibranch Pipeline.

- Click OK.

Under Branch Sources, click Add source and then select GitHub. For the new source, provide the following information:

Credentials: Select the GitHub Key option
Owner: Provide your GitHub username
Repository: Select the cicd-pipeline-train-schedule-kubernetes fork
Click Save.

Click train-schedule in the menu at the top of the page. Click master to display the builds for the master branch.

Successfully deploy the train-schedule app to the Kubernetes cluster via the Jenkins pipeline
Create a Kubernetes template file defining a Service and Deployment for the app
Back in the GitHub tab in your browser, click Create new file. Name this file "train-schedule-kube.yml". Insert the following YAML as the contents of the new file:
```
kind: Service
apiVersion: v1
metadata:
  name: train-schedule-service
spec:
  type: NodePort
  selector:
    app: train-schedule
  ports:
  - protocol: TCP
    port: 8080
    nodePort: 8080

---

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: train-schedule-deployment
  labels:
    app: train-schedule
spec:
  replicas: 2
  selector:
    matchLabels:
      app: train-schedule
  template:
    metadata:
      labels:
        app: train-schedule
    spec:
      containers:
      - name: train-schedule
        image: $DOCKER_IMAGE_NAME:$BUILD_NUMBER
        ports:
        - containerPort: 8080
```

- Click Commit new file at the bottom of the page.

- Add a stage to the Jenkinsfile to perform the deployment to the Kubernetes cluster
- Click Jenkinsfile to configure this file. Click the Edit icon in the top-right of this window.

- At the top of this file, be sure to change the following line to use your Docker Hub username instead of the instructor's:

DOCKER_IMAGE_NAME = "willbla/train-schedule"
At the bottom of this file, replace the DeployToProduction section with the following:
```
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
```
- Click Commit changes at the bottom of the page.

- Run the build and successfully deploy the app to the cluster
- Back in the Jenkins tab in your browser, navigate to the train-schedule app and click master to show the master branch.

- Click Build Now in the menu on the left of the page.

- When your deployment reaches the DeployToProduction stage, be sure to hover over the blue box and click Proceed to approve the push to production.

- Verify everything is working by accessing the app in your browser
- On the hands-on lab page, copy the Kubernetes Node Public IP, open a new tab in your browser, and navigate to that IP address, using port 8080:`<KUBERNETES_NODE_PUBLIC_IP>;:8080`
- The application page should load and you should see the text "Find your train!".

- Conclusion
- Congratulations, you've completed this hands-on lab!
