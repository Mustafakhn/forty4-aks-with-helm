# Assignment

**Objective:** Deploy a simple application on Azure Kubernetes Service (AKS) using Helm and
expose it to the public internet. Use Terraform to deploy the AKS cluster.

**Requirements:**
1. Create an AKS Cluster:
    - Deploy an AKS cluster with at least one node using terraform. 
2. Deploy a Sample Application Using Helm:
    - Use Helm to deploy a simple application, such as a "Hello World" web server.
    - Create a custom Helm chart.
3. Expose the Application to the Public Internet:
    - Configure the application to be accessible from the public internet using a Kubernetes Ingress.

# Solution

## Setup
1. create an account on azure and install azure cli [Azure CLI download and initialisation ](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
2. install terraform [Terraform download and installation](https://developer.hashicorp.com/terraform/install)
3. install helm [Helm download and installation](https://helm.sh/docs/intro/install/)
## Azure CLI Setup
- Follow the steps below:
    - ```az login```
    - ```az account list```
    - Copy the `id` field from the output of the above command and run
      - ```az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<PASTE THE ID HERE>"```
    - Output will be something like this
      - ```
        {
            "appId": "00000000-0000-0000-0000-000000000000",
            "displayName": "azure-cli-2024-06-26-20-01-37",
            "name": "http://azure-cli-2024-06-26-20-01-37",
            "password": "0000-0000-0000-0000-000000000000",
            "tenant": "00000000-0000-0000-0000-000000000000"
          }
        ```
    - Export the following variable values from the output of the previous command
      - ```
        export ARM_CLIENT_ID=<insert the appId from above>
        export ARM_SUBSCRIPTION_ID=<insert your subscription id>
        export ARM_TENANT_ID=<insert the tenant from above>
        export ARM_CLIENT_SECRET=<insert the password from above>
        ```
 ## Cluster Creation
  - Run the following commands after declaring the environment variables
    - `cd terraform`
    - `terraform init`
    - `terraform plan -out main.tfplan`
    - `terraform apply main.tfplan`
    - come out of the terraform directory with the following command `cd ..`
    this command will ask you for an approval if you wish to skip the manual approval you can add `-auto-approve` at the end of the command

## Kubernetes config
  - Once your cluster is ready run `export KUBECONFIG="${PWD}/kubeconfig"`

## App deployment with public access
  - Run the following commands:
    - To deploy nginx ingress controller :
      ```
      helm repo add nginx-stable https://helm.nginx.com/stable
      helm repo update
      helm install nginx-ingress nginx-stable/nginx-ingress --set rbac.create=true
      ```
    - Check if the nginx controller pod is deployed or not ```kubectl get pods --all-namespaces```
      - You should see a pod with `nginx-ingress` in it's name
    - Check if the service is running or not by running the following command ```kubectl get services```
      - You should see a service with `nginx-ingress` in it's name
      - Copy it's external IP and add the `A record` for the DNS you want to map to the application
    - After the successful ingress deployment now deploy the application by running the commands below:
      - ```
        helm upgrade -i basic-application basic-application --set ingress.hosts[0].host=<YOUR DNS NAME> \
        --set ingress.hosts[0].paths[0].path="/" \
        --set ingress.hosts[0].paths[0].pathType=ImplementationSpecific
        ```
        Don't forget to replace `<YOUR DNS NAME>` with your actual DNS
    - Check if the application and the service is deployed by running following commands :
      - `kubectl get pods --all-namespaces` you should see a pod with `basic-application` in it's name
      - `kubectl get services --all-namespaces` you should see a service with `basic-application` in it's name
    - Now visit your DNS endpoint in the browser `http://<YOUR DNS ENDPOINT>/` and you should see the weather application
