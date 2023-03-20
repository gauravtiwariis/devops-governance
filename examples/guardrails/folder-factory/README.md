# Folder Factory

This is a template for a DevOps folder factory.

It can be used with [https://github.com/google/devops-governance/tree/main/examples/guardrails/project-factory](https://github.com/google/devops-governance/tree/main/examples/guardrails/project-factory) and is intended to house the folder configurations:

![Screenshot 2022-05-10 12 00 19 PM](https://user-images.githubusercontent.com/94000358/224204373-f17024c0-1a2c-474b-82c2-affd8119cc05.png)

Using Keyless Authentication the project factory connects a defined Github repository with a target service account and project within GCP for IaC.

The idea is to enable developers of the "skunkworks" repository to deploy into the "skunkworks" project via IaC pipelines on Github.

Setting up folders
The folder factory will:

create a folders with defined organisational policies
It uses YAML configuration files for every folder with the following sample structure:

parent: folders/XXXXXXXXX
org_policies:
  policy_boolean:
    constraints/compute.disableGuestAttributesAccess: true
    constraints/iam.disableServiceAccountCreation: false
    constraints/iam.disableServiceAccountKeyCreation: false  
    constraints/iam.disableServiceAccountKeyUpload: false
    constraints/gcp.disableCloudLogging: false 
  policy_list:
    constraints/compute.vmExternalIpAccess:
      inherit_from_parent: null
      status: true
      suggested_value: null
      values:
iam:
  roles/resourcemanager.projectCreator:
    - serviceAccount:XXXXX@XXXXXX


Every folder is defined with its own yaml file located in the following Folder. Copy "folder.yaml.sample" to "folder_name.yaml"; Name of the yaml file will be used to create folder with the same name. Once folder_name.yaml file is created update yaml file

parent - can be another folder or organization
ServiceAccount data/folders can have multiple yaml files and a folder will be created for each yaml file.
How to run this stage
Prerequisites
Workload Identity setup between the folder factory gitlab repositories and the GCP Identity provider configured with a service account containing required permissions to create folders and their organizational policies. There is a sample code provided in “folder.yaml.sample” to create a folder and for terraform to create a folder minimum below permissions are required. “Folder Creator” or “Folder Admin” at org level “Organization Policy Admin” at org level

Installation Steps
From the folder-factory Gitlab project page

CICD configuration file path Navigate to Settings > CICD > expand General pipelines Update “CI/CD configuration file” value to the relative path of the gitlab-ci.yml file from the root directory e.g. .gitlab/workflows/.gitlab-ci.yml

CI/CD variables Navigate to Settings > CICD > expand Variables Add the variables to the pipeline as described in the table below. The same can be accessed from the README.md file under .gitlab/workflows in folder-factory.

Terraform config validator
The pipeline has an option to utilise the integrated config validator (gcloud terraform vet) to impose constraints on your terraform configuration. You can enable it by setting the CI/CD Variable $TERRAFORM_POLICY_VALIDATE to "true" and providing the policy-library repo URL to $POLICY_LIBRARY_REPO variable. See the below for details on the Variables to be set on the CI/CD pipeline.
      

Every folder is defined with its own yaml file located in the following [Folder](data/folder).

Pipeline Workflow Overview
The complete workflow comprises of 4-5 stages and 2 before-script jobs

before_script jobs :
gcp-auth : creates the wif credentials by impersonating the service account.
terraform init : initializes terraform in the specified TF_ROOT directory
Stages:
setup-terraform : Downloads the specified TF_VERSION and passes it as a binary to the next stages
validate: Runs terraform fmt check and terraform validate. This stage fails if the code is not run against terraform fmt command
plan: Runs terraform plan and saves the plan and json version of the plan as artifacts
policy-validate: Runs gcloud terraform vet against the terraform code with the constraints in the specified repository.
apply: This step is currently set as manual to be triggered from the Gitlab pipelines UI once the plan is successful. Runs terraform apply and creates the infrastructure specified.
