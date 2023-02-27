## OCBC RED Masterclass Hands-on Lab
#

### **Introduction**
This lab covers the fundamentals of DevOps and cloud-native application development on OCI.

### **Pre-requisites**
1. Login credentials for your assigned users.
2. Basic understanding of Terraform, Docker and Kubernetes.

### **Lab instructions**

### **(a) Provision a Container Engine for Kubernetes (OKE) cluster via Resource Manager**

1. On the search bar, type "OKE" and click on "Kubernetes Clusters (OKE)" under Services.
2. Ensure you're in the right compartment. Eg. If you're user100, choose "user100" under "ocbc_participants" compartment.
![image](img/1.%20select_compartment.png)
3. Click on "Create cluster"
4. Select "Quick create" and press "Submit"
![image](img/2.%20quick_create_cluster.png)
5. Insert the following configurations

    - Name: `<allocated_user>-cluster`, Eg. user100-cluster
    - Compartment: `Your allocated compartment`, Eg. user100
    - Kubernetes version: `v1.25.4`
    - Kubernetes API endpoint: `Private endpoint`
    - Kubernetes worker nodes: `Private workers`. Default.
    - Pod shape: `VM.Standard.E3.Flex`. Default.
    - OCPUs: `1`
    - Memory: `4`
    - Image: Default
    - Node count: `1`

6. Press "Next".
7. Check the configuration and press **"Save as stack"**. [**WARNING**] Do not press "Create cluster".
![image](img/3.%20save%20as%20stack.png)
8. Change the stack name to `<allocated_user>-stack` eg.user100-stack and press "Save".
9. On the seach bar, type "stacks" and click on "Stacks" under Services.
10. Click on the stack.
![image](img/4.%20view_stack_oke.png)
11. Click "Apply" to provision the cluster.
![image](img/5.%20apply_stack_job_oke.png)
12. It'll take about 10mins - 15mins to complete the provisioning. While waiting, go to [Section (b)](redme.md#).
13. [**Optional**] Meanwhile, to inspect the Terraform configuration, go back to Stack Details, press "Edit" and select "Edit Terraform configuration in code editor". 
14. [**Optional**] When Code Editor session is created, on the left pane, click on the Oracle logo and you should see the list of supported OCI Plugins. 
15. [**Optional**] Expand "RESOURCE MANAGER" list and continue to expand it (ocbcredsg -> Compartments -> ocbc_participants -> <your_user_compartment> -> Stacks -> <stack_name> -> main.tf ). This TF script includes provisioning of all the resources required for a functional Kubernetes cluster such as VCN, route table, security list, subnets, master nodes and worker nodes.

### **(b) Create DevOps Project**

1. On the search bar, type "devops" and click on "Projects" under Services.
2. Ensure you're in the right compartment. Eg. If you're user100, choose "user100" under "ocbc_participants" compartment.
3. Click on "Create devops project"
4. Insert the following configurations
    - Project name: `<<allocated_user>-devops-project>` Eg. user100-devops-project
    - Click "Select topic", change the Compartment to "ocbc_participants" and select ocbc_common_topic under "Topic" and press "Select topic"
5. Press **"Save as stack"**. [**WARNING**] Do not press "Create devops project".
6. Append `-stack` to the "Name". Eg. user100-devops-project-stack and press "Save".
7. On the seach bar, type "stacks" and click on "Stacks" under Services.
8. Click on the stack.
9. Click "Apply" to create the DevOps project.
10. It'll take about 1min to complete the provisioning.
11. When it's completed, go back to DevOps project and click on the project to verify the project.
12. On the main project page, click on "Enable log" on the Enable Logging message and enable log.
13. Keep the default configurations and press "Enable Log".
12. Now the DevOps project is created, go to Section (c) to inspect the OKE cluster.

### **(c) Secure Access to a private OKE Cluster**

1. When the Apply job is completed for OKE cluster, on the search bar, type "OKE" and click on "Kubernetes Clusters (OKE)"under Services and you should see the OKE cluster.
2. Click on the OKE cluster
3. On the Cluster details page, click on the subnet "oke-k8sApiEndpoint-subnet-quick*" under "Kubernetes API endpoint subnet".
4. Click on the security list under "Security Lists".
5. Click on "Egress Rules" and press "Add Egress Rules".
6. Insert the following configurations
    - Destination CIDR: `10.0.0.0/28`
    - Description: `for cloud shell access`
7. Click on "Add Egress Rules".
8. Go back to OKE Cluster details page, click on "Access Cluster".
9. Copy the content of (2) and press "close"
10. On the top right corner, press on the logo of Developer Tools and select "Cloud Shell".
11. When the cloud shell session is created, expand the cloud shell by pressing "Maximize" on the top right corner of cloud shell.
12. Paste the content of step 9 on the shell and press enter. A kubeconfig file will be created.
13. Notice the Network is set as "Public" but the OKE cluster is a private cluster so we need to access the cluster through the private network. 
14. Expand the network setting and select "Ephemeral Private Network Setup". 
15. Insert the following configurations
    - VCN: `oke-vcn-quick-*`
    - Subnet: `oke-k8sApiEndpoint-subnet-quick*`
16. Click on "Use as active network". The cloud shell network will be connecting to the ephemeral private network.
17. When the ephemeral network is connected, type `kubectl get nodes` and you should see the worker nodes under this cluster.
18. Now the cluster is ready, we're ready to deploy our applications to the cluster.

### **(d) Deploy application to OKE via DevOps Continuous Delivery pipeline**

1. On the search bar, type "devops" and click on "Projects" under Services.
2. Click on the devops project.
3. On the left pane, click on "Environments" and "Create environment"
4. Insert the following configurations
    - Environment type: `Oracle Kubernetes Engine`
    - Name: `<allocated_user>-cluster`, Eg. user100-cluster
5. Press "Next" and insert the following configurations
    - Region:  `Singapore`
    - Compartment: Your compartment
    - Cluster: Your OKE cluster
    - VCN: `oke-vcn-quick-*`
    - Subnet: `oke-k8sApiEndpoint-subnet-quick*`
6. Press "Create environment"
7. Go back to devops project page and click on "Artifacts" and "Add artifact"
8. Insert the following configurations
    - Name: `Nginx K8s Manifest`
    - Type: `Kubernetes manifest`
    - Artifact source: `Artifact Registry repository`
    - Artifact registry repository: Click "Select", change compartment to `ocbc_participants`, tick the box on `ocbc-artifact-repository`, and press "Select".
    - Artifact Location: `Select Existing Location`
    - Artifact: Click "Select", tick the box on `ocbc_lab/nginx_k8s:v0.1.0`, and press `Select`.
    - Tick the box on `Allow parameterization`
9. Press "add".
10. On the left pane, click on "Deployment Pipelines" and "Create pipeline"
11. Insert the following configuration and press "Create pipeline"
    - Name: `nginx-deployment-pipeline`
12. On the pipeline configuration page, press "Add Stage", select "Apply manifest to your Kubernetes cluster" and press "next".
13. Insert the following configurations
    - Stage name: `Nginx deployment`
    - Environment: Choose the environment created in step (d).4
    - Artifact: Click "Select artifact", tick the box on the `Nginx K8s Manifest` artifact and press "Save changes"
14. Press "Add".
15. On the pipeline configuration page, press "Parameters" on the pipeline top menu pane.
16. Insert the following configurations
    - Name: `VERSION`
    - Default value: `0.1.0`
    - Description: `release version`
17. Press the plus button on the right.
18. Go back to the pipeline configuration page by pressing "Pipeline" on the pipeline top menu pane.
19. Click on "Run pipeline" on the top right corner, and press "Start manual run" to run the deployment.
20. The deployment should be executed successfully. Now let's access the application.

### **(e) Access the application**

1. Return to Cloud Shell. Refer to step (c).10.
2. Run `kubectl get svc -n nginx-webapp` and note down the "EXTERNAL-IP".
3. Go to your favourite browser and run "http://<EXTERNAL-IP>:5000" 
4. You should see a simple Nginx Webpage.