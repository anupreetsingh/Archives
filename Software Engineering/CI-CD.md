# CI-CD Pipeline

CI/CD is the automated process that takes code from a developer’s commit and moves it through testing, packaging, and eventually deployment to production.

CI Systems such as Github Actions, GitLab CI, Jenkins automate the work of CI/CD pipeline by having preconfigured workflows in `.yaml` files that start when a **trigger** event happens. Each trigger event is has multiple jobs that start independent runners(VMs) to perform sequential steps.

We will use this example single python application structure to learn about the CI/CD pipeline:

```txt
my-project/
├── src/
├── tests/
├── requirements.txt
├── Dockerfile
├── README.md
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
└── .github/
    └── workflows/
        ├── ci.yaml 
        └── cd.yaml
```

## Container

What containers and why they are needed and then introduction for docker. From that point onwards a discussion about docker.
→ What containers/images are conceptually
→ Dockerfile
→ build/run/tag/push
→ container vs image
→ why ECR stores images

## Container Orchestrator

You can have an automated CD pipeline without kubernetes but Kubernetes gives you things like rolling deployments, self-healing, replicas, scaling, service discovery, and declarative configuration.

What a container orchestrator is? How kubernetes is a very common option and how it could be run directly on cloud infra, what EKS is and how it differs and what ECS is and how it differs in setup and configuration.
→ Cluster, control plane, worker nodes
→ Pods, Deployments, Services
→ scheduling
→ replicas
→ rolling updates
→ Consequences of choosing ECS vs EKS vs Plain kubernetes run on EC2 and cost associated with each of them
→ EKS / EC2 worker nodes / Fargate
→ conceptual relationship between all of them

## CI

**CI = Continuous Integration**  The goal is to continuously verify that new code integrates correctly with the existing codebase. It mainly involes running tests and building the docker image.

Each `.yaml` file describes one workflow. This one describes a simple workflow for standard CI tasks of one application which are:

test job:

- Can dependencies install?
- Does the code follow linting rules?
- Do the tests pass?

build job:

- Can we successfully build the application/container?
- If yes and this is push to main trigger, we update the image on Amazon ECR(Elastic Container Registry)

### ci.yaml

Example content of `ci.yaml` file:

```yaml
# Name of the workflow
name: CI

# Trigger Events
# CI Job is triggered on a pull request or push to the main branch.
on:
  pull_request:
    branches:
      - main

  push:
    branches:
      - main

jobs:
  # test is the name of a job
  # github actions indents 2 spaces for next level
  test:
    # Creates a fresh Github-hosted Ubuntu Runner(VM) for the test job
    runs-on: ubuntu-latest
    # Everything under steps then executes on that same runner sequentially
    # The new VM starts without your repository files.
    steps:
        
      - name: Checkout code # Custom name for the step
        uses: actions/checkout@v4 # A standard GitHub Action that checks out(downloads) the commit that triggered the workflow.

      - name: Set up Python 
        uses: actions/setup-python@v5 # A standard Github Action
        with: # Passes configuration options into that action
          python-version: "3.12" # tells it to make Python 3.12 available on the runner
          cache: "pip" # enables caching for pip dependencies, so future workflow runs can often install packages faster.

      - name: Install dependencies
        run: pip install -r requirements.txt # Runs shell command for pip(which came with the setup-python@v5)
    
      - name: Lint code
        run: ruff check . # ruff would have been listed in the requirements.txt. Runs a Lint check across all python files

      - name: Run tests
        run: pytest # pytest runs the automated tests in the tests/ folder that are named test_users.py, test_api.py, etc.

  # Another job name
  build:
    needs: test # Requires the test job to successfully finish
    runs-on: ubuntu-latest  # Creates separate runner for build job

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

        # This build the docker image locally on the runner.
        # Builds the image tagged with SHA because later docker tag expects it.
      - name: Build Docker image
        run: docker build -t my-app:${{ github.sha }} . # Ubuntu-latest comes with docker. 

      # The steps below only happen after code is actually
      # pushed/merged into main.

      - name: Configure AWS credentials
        if: github.event_name == 'push' # Only run this step if the workflow was triggered by a push.
        uses: aws-actions/configure-aws-credentials@v6 # aws-actions are made by AWS - Github organization.
        with:
          # AWS_ROLE_ARN contains the AWS account ID as part of the ARN, conceptually: arn:aws:iam::<AWS_ACCOUNT_ID>:role/<ROLE_NAME>
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }} # from the secrets saved on github, AWS_ROLE_ARN contains the ARN of an IAM role GitHub Actions is allowed to assume.

          aws-region: us-east-1 # Push image to ECR on AWS region in northen virginia

      
      - name: Login to Amazon ECR
        if: github.event_name == 'push'
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2


      - name: Tag Docker image for ECR
        if: github.event_name == 'push'
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: my-app
        run: |
          docker tag my-app:${{ github.sha }} \
          $ECR_REGISTRY/$ECR_REPOSITORY:${{ github.sha }}


      - name: Push Docker image to ECR
        if: github.event_name == 'push'
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: my-app
        run: |
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:${{ github.sha }}

```

If any step in a job fails, that job stops and is marked as failed. The CI workflow is considered failed if any required job fails. Other independent jobs(running on separate runners/VMs) may still continue running. A properly configured CD pipeline will only deploy if the required CI jobs succeed, preventing bad code from being deployed.

> The CI workflow runs twice during a PR -> merge lifecycle here (Once on creating the PR and once when it merges on approval and a commit is pushed on main)

In a more complex **multi container** application we would have different containers for different services(Frontend, backend, database) each with its own image. All build at once in the ci.yaml by running a central docker compose.

Now if you want that complex application to be actually **microservice** architecture then there deployments should also be separated by having different deployment.yaml file for each microservice.

## CD

**CD = Continuous Deployment** The goal is to take code that has passed CI and reliably release it to an environment such as staging or production. It mainly involves taking that updated image from ECR and update the kubernetes deployment to reference the new image on fresh pods.

GitHub Actions automates this process, while Kubernetes handles the actual deployment behavior such as replacing the old Pods with Pods running the new image.

Before the CD workflow can deploy the application, the deployment infrastructure must already exist. For this example, we assume the following have already been created in AWS:

- An Amazon ECR repository for storing Docker images.
- An Amazon EKS cluster.
- Worker nodes for the EKS cluster.
- The required AWS IAM roles and permissions.

The EKS cluster and worker nodes could have been created manually through the AWS Management Console, through the AWS CLI, or preferably using Infrastructure as Code tools such as **Terraform** or **CloudFormation**.

### k8s/

We also assume the Kubernetes resources for the application have already been created from the files in the `k8s/` directory. For the initial deployment these could be applied using:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

`deployment.yaml` describes how the application should run.

```yaml
# Which version of the Kubernetes API defines this resource.
apiVersion: apps/v1
# The type of Kubernetes resource this file defines.
kind: Deployment
metadata:
  # Name of the Kubernetes Deployment
  name: my-app

# Spec is the desired configuration for the kubernetes resource
spec:
  # Desired number of application Pods
  replicas: 3
  # This Deployment manages Pods that have the label my-app
  selector:
    matchLabels:
      app: my-app
  # Template Kubernetes uses whenever it creates a new Pod
  template:
    metadata:
      labels:
        app: my-app # name for the Pod
    spec: # Specs of the pod 
      containers: # Container configuration for this pod
        - name: my-app # container named my-app inside this pod
          # Docker image each Pod should run
          image: <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-app:<INITIAL_TAG> 
          ports: # Application inside this container listens on port 8000
            - containerPort: 8000
```

`service.yaml` describes how traffic should reach the application's Pods.

```yaml
# Which version of the Kubernetes API defines this resource.
# Services are part of the core Kubernetes API, so this is just v1.
apiVersion: v1
# The type of Kubernetes resource this file defines.
kind: Service
metadata:
  name: my-app-service # Name of this Kubernetes Service

# Desired configuration for this Service
spec:
  # Send traffic to Pods with this label
  selector:
    app: my-app # Helps finds the pods with the fitting name
  ports:
    - protocol: TCP
      # Port exposed by the Service
      # Clients send requests to the Service on port 80.
      port: 80
      # Port where the application is listening inside the Pods
      targetPort: 8000 # Helps finds the container at one pod.

  # Creates an external load balancer so traffic from outside
  # the Kubernetes cluster can reach this Service.
  # On AWS EKS, this causes AWS to provision a load balancer
  # that forwards traffic to this Kubernetes Service.
  type: LoadBalancer
```

After this initial setup, later deployments usually do not need to recreate the Kubernetes Deployment and Service. The purpose of CD is now to tell the existing Kubernetes Deployment to use the newly created image.

### cd.yaml

`cd.yaml` is triggered after ci.yaml finishes without failure because of the configurations it has.

Example content of the `cd.yaml` file:

```yaml
# Name of the workflow
name: CD

on:
  # Trigger event is another workflow run in the same GitHub repository
  workflow_run:
    workflows:
      - CI
    types:
      - completed
    branches:
      - main

# Permission given to github actions
permissions: 
  id-token: write # Allowed to request an OIDC identity token from GitHub which serves as to verify identity to AWS for assuming the IAM role.
  contents: read # Read the repository contents

jobs:
  # deploy is the name of the job
  deploy:
    if: github.event.workflow_run.conclusion == 'success'
    runs-on: ubuntu-latest

    # env defines the environment variables
    env:
      AWS_REGION: us-east-1
      ECR_REPOSITORY: my-app
      EKS_CLUSTER: my-cluster
      IMAGE_TAG: ${{ github.event.workflow_run.head_sha }}

    # Sequential steps
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
    
      # Tells Github actions to assume this IAM role in AWS and use its credentials for subsequent aws commands.
      # After this step, the runner is authenticated into a particular AWS account.
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}
    
      # Given the credentials I'm currently using, what AWS account am I logged into?
      - name: Get AWS account ID
        id: aws-account
        run: |
          echo "account_id=$(aws sts get-caller-identity \
            --query Account \
            --output text)" >> "$GITHUB_OUTPUT"

        # Installing kubectl(open-source Kubernetes CLI program)
      - name: Set up kubectl
        uses: azure/setup-kubectl@v4 # azure team makes this Github Action for installing kubectl

      # Connect kubectl to the EKS cluster in the currently authenticated AWS account
      - name: Connect kubectl to EKS
        run: |
          aws eks update-kubeconfig \
            --region "$AWS_REGION" \
            --name "$EKS_CLUSTER"

        # Find the deployment named my-app
        # Finds the container named my-app inside that deployment and changes its image to the new ECR image
      - name: Update Kubernetes deployment
        env:
          ECR_REGISTRY: ${{ steps.aws-account.outputs.account_id }}.dkr.ecr.us-east-1.amazonaws.com
        run: |
          kubectl set image deployment/my-app \
            my-app="$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"

        # Now Kubernetes starts rolling the update
        # New pods are created with new image and old pods are gradually terminated

        # Watches rollout status for the specific Kubernetes deployment resource named my-app.
      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/my-app
```

Gradual update of ECR images on the containers of the pods:

```text
Pod 1 → v1
Pod 2 → v1
Pod 3 → v1

       ↓

Pod 1 → v2
Pod 2 → v1
Pod 3 → v1

       ↓

Pod 1 → v2
Pod 2 → v2
Pod 3 → v1

       ↓

Pod 1 → v2
Pod 2 → v2
Pod 3 → v2
```
