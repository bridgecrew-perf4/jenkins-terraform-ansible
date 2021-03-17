//Pipeline flow:
//Agent: Launch Jenkins agent in Docker container with Terraform and Ansible (base image: mas0lik/terraform-jenkins-agent);
//Stage 1: Clone Terraform manifest and Ansible Playbook form GitHub;
//Stage 2: Copy GCP authentication json form Jenkins secrets to working directory;
//Stage 3: Encrypt Dockerhub password with Ansible Vault;
//Stage 4: Execute Terraform Init, Plan and Apply.

//Terraform and Ansible flow:
// Terraform deploys two VM instances (Staging and Production) at Google Cloud (GCP), forms custom inventory for Ansible
// and then invokes Ansible playbook with roles.
// Ansible playbook sets Staging and Production VM configurations according to designated roles:
// STAGING environment: VM 'terraform-staging' is used to build web application "Boxfuse" inside Docker container
// and push docker image with artifact to Dockerhub
// PRODUCTION environment: VM 'terraform-production' is used to pull Docker image with artifact and start docker container

pipeline {

  agent {

    docker {
      image 'mas0lik/terraform-jenkins-agent'
      //Use arguments to map sockets and user Root
      args  '-v /var/run/docker.sock:/var/run/docker.sock -u 0:0'
      //Use to automate authentication at Docker Hub
      registryCredentialsId 'a085e5ff-0db2-450e-83f5-d67081d88561'
    }
  }

  stages {

    //Stage 1
    stage('Clone Terraform manifest and Ansible Playbook form GitHub') {
      steps{
        //Use 'git: Git' to clone Terraform manifest and Ansible playbook
        git 'https://github.com/mas0lik/terraform-docker-v2.git'
      }
    }

    //Stage 2
    stage('Copy GCP authentication json to workspace on Jenkins agent') {
      steps {
        //Inject GCP authentication json to agent
        withCredentials([file(credentialsId: 'da4a30f9-38c6-47e0-a6ac-5a0409c54e45', variable: 'gcp_auth')]) {
          sh 'cp \$gcp_auth DevOps-gcp.json'
        }
      }
    }


    //Stage 3
    stage('Encrypt Dockerhub password with Ansible Vault') {
      steps {
        //Encrypt Dockerhub password with Ansible Vault and export output to Ansible Role defaults
        withCredentials([string(credentialsId: '817ad7a9-cc79-4b0f-8a84-fa362f049a33', variable: 'encrypt')]) {
          sh 'ansible-vault encrypt_string $encrypt --name dockerhub_token --vault-password-file vault_pass >>./roles/dockerhub_connect/defaults/main.yml'
        }
      }
    }

    //Stage 4
    stage('Execute Terraform Init, Plan and Apply') {
      steps {
        // Execute init, plan and apply for Terraform main.tf
        sh 'terraform init'
        sh 'terraform plan'
        sh 'terraform apply -auto-approve'
      }
    }

  }
}