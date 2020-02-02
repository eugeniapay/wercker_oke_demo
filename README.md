# Wercker with OKE Demo

This demo showcases how you can use Wercker, Oracle Kubernetes Engine and OCI Registry for complete DevOps

## Demo Overview

This demo uses a demo springboot REST API to illustrate how you can configure Wercker to integrate with GitHub's Springboot repository, and define the CI/CD pipeline with the following steps:

- Build SpringBoot REST API
- Generate Docker Image
- Push Docker Image to OCIR
- Pull Docker Image from OCIR and Deploy to OKE

This steps have been tested on Oracle Cloud Infrastructure Gen 2 as of 3 Feb 2020. Should you encounter any problems, please feel free to feedback to Yang Yang (y.yeung@oracle.com).

[Special Note] This is a work-in-progress version, more details will be provided in the near future. Please stay tuned.

## Pre-Requisite

Before you can proceed with the following demo, please ensure that you have the following accounts:

- OCI Gen 2 Account with Container Pipeline (Wercker), OKE, OCI Registry Access
- GitHub Account
- Wercker Account

In addition, you should also have gone through the guide in http://github.com/yyeung13/oke_ocir_demo to setup OCIR/OKE as well as the OCI CLI to manage the kubernetes cluster.

## Demo Details

1. Fork the demo from Github https://github.com/yyeung13/spring-rest-service.git into your own GitHub. You will be using your own GitHub to see the effect of CI/CD

2. Inspect the demo REST API demo before proceeding to subsequent steps

- wercker.yml: This is the file that configure various stages of the build process. Before you can use Wercker UI to compose a workflow, all pipelines within the workflow must be defined and configured in this werkcer.yml
- spring_rest.yml: This is the YAML file to pull the image from OCIR and deploy to OKE. It contains the configuration of the spring REST API app deployment as well as a load balancer service that routes request to the REST API deployment
- Dockerfile: this is the Dockerfile we will be using later to build Docker image from SpringBoot executables
- Other files are from the original spring REST API sample from https://spring.io/guides/tutorials/rest/

3. Obtain the following environment variables and keep it handy for subsequent steps

- OCIR URL: My sample runs in us-ashburn-1 and it's region code is iad, so my OCIR URL is iad.ocir.io. If you are on different region, please look for the corresponding code based on your region
- OCIR Username: This is the user name to login to OCIR. If you need help locating this, refer to the guide in http://github.com/yyeung13/oke_ocir_demo
- OCIR Password: This is the auth token to login to OCIR. You would have gotten this token from the previous demo in http://github.com/yyeung13/oke_ocir_demo
- Tenancy Namespace: this is the OCIR tenancy namespace you got from the previous demo in http://github.com/yyeung13/oke_ocir_demo
- OKE Master: This is the cluster server URL in kubeconfig. You can find it by running 'kubectl config view' and check under cluster -> server
- OKE Token: This is the token you used to login to Kubernetes Dashboard. If you need more help getting this, go to OKE console and click on 'Access Kubernetes Dashboard'. It will provide details on how to obtain this value

4. Configure wercker.yml in GitHub

- From Github web console, click on the file wercker.yml and edit it
- This YAML file include configuration for a few build steps: dev, build, push, deploy. Take time to read through the various build steps to understand what it is doing
  - dev: ignore this as I'm not using it for this demo
  - build: compile the SpringBoot application
  - push: build docker image and push to OCIR
  - deploy: pull from OCIR and deploy to OKE
  
5. Configure Build Workflow in Wercker

- From app.wercker.com, create a new application by clicking the + icon on top right hand corner of main page and select New Application
- Select the default user and GitHub as SCM and click 'Next'
- From the list of repositories shown, select spring_rest_service and click 'Next'
- Accept default recommendation to let wercker check out code without using an SSH Key and click 'Next'.
- Review the setting and click 'Create' to create a new application
- Click on the new application created: spring_rest_service and click on 'Environments' tab
- Specify the following environment parameters
  - DOCKER_USERNAME: <tenancy_namespace>/<OCI user name>, e.g., mytenancy/oracleidentitycloudservice/y.yeung@oracle.com
  - DOCKER_PASSWORD: This is the auth token generated to access Docker
  - DOCKER_REPO: <ocir_url>/<tenancy_namespace>/<repo_name>/<image_name>, e.g., iad.ocir.io/mytenancy/demo/spring_rest
  - OKE_MASTER: E.g., https://random.us-ashburn-1.clusters.oci.oraclecloud.com:6443
  - OKE_TOKEN: This is the token for accessing Kubernetes Dashboard
  
  It's recommended to configure Docker Password and OKE Token as 'protected' values so that it won't be shown on screen for security purpose
- Now click on 'Workflow' tab and configure the build process
- Scroll down to locate Pipeline section and click on 'Add new Pipeline'
- Enter the following:
  - Name: deploy
  - YAML Pipeline Name: deploy
  - Hook Type: Default
 - Click on 'Create'
 - Repeat the same to create another pipeline called 'push-to-OCIR' and YAML Pipeline name as 'push'
 - Click on 'Workflow' tab again to see the visual representation of the build pipeline
 - Click on the + sign after build stage, scroll to the bottom to select execute pipeline as 'push', the one you have just configured earlier
 - Click on the + sign after push stage and add pipeline 'deploy' after 'push'
 - You should see a workflow that execute the following pipelines sequentially: build -> push -> deploy
- Now go back to Github and make some changes to readme.MD file, any changes in any files under spring-rest-service should trigger a build in werkcer automatically. Upon commiting the changes to GitHub, go back to wercker Web Console and click on 'Runs' tab. You should see the build starting. If there is any error in the build process, you can see the error log by clicking on the error pipeline, details of each steps of the error pipeline will be shown
- Upon successful build, you should be able to verify that the image spring_rest is pushed to OCIR, and its pods are created in OKE, as well as its load balancer services
- From OCI CLI terminal, run 'kubectl get po' to confirm the Pods are running
- At the same console, run 'kubectl get services' to confirm the service for spring_rest_service is running and identify its public IP. In case the public IP (or external IP as it shows on screen) is shown as <pending>, try again later as sometimes it takes time to secure the public IP. If the status is still pending after 5 minutes, please check if you have any other pods not running smoothly. You may need to delete those pods first and try again as I encounter similar problem with trial OCI Gen 2 accounts
- From a browser, acccess http://<public_ip>:8080/greeting?user=Yang. You should see something like '{"id":3,"content":"Good Morning, Yang!"}'. Note that the ID may change as well as its greetings as I may have changed the greeting for various demos
- You got the CI/CD process running now!
  
6. Test with Changes

- Let's delete the spring-rest-deployment and spring-rest-service from Kubernetes. Note that you need to do this because I have not configured incremental build yet
  - Delete deployment by running 'kubectl delete deployment spring-rest-deployment'
  - Delete service by running 'kubectl delete service spring-rest-service'
- From GitHub, locate the file src/main/java/com/example/restservice/GreetingController.java and Edit line 12 and change the greeting from 'Hello/Good Morning' to anything of your choice. Commit the chagnes
- You can go back to Wercker console to see a new build is triggered. Wait for the build to complete
- Check the public IP again and navigate to the same URL http://<public_ip>:8080/greeting?user=Yang. You should see the new greetings have taken effect
- This demonstrates any changes will trigger a build/push/deploy as configured in the Wercker Workflow

## Wrapping Up

You can follow the same guideline to build any applciations such as Springboot, Tomcat, Helidon. Should you require further assistance, please feel free to let us know.
