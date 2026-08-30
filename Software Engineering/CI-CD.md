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

An application needs more than its source code to run. It also depends on a particular runtime, system libraries, installed packages, configuration, and startup command. These dependencies should remain consistent between development and production environments. Otherwise, differences in versions, libraries, or configuration can cause an application that works in development to fail in production.

A **container** packages the application and its runtime dependencies into a standardized unit. The application process is isolated from other processes, but it still shares the host machine's OS kernel. This generally makes a container smaller and faster to start than a virtual machine, because a VM includes and boots an entire guest OS. But this also means that multiple applications with conflicting dependencies can run in isolation on the same host machine.

Containers do not remove every environmental difference. CPU architecture, kernel capabilities, external configuration, secrets, networking, and persistent storage still have to be managed separately.

### Docker

**Docker** is a common toolset for building container images and running containers. It introduces two closely related objects:

- An **image** is an immutable, reusable template for *making* containers and not a running process. It stores the application filesystem as a layered package along with metadata such as its default command.

    Consider an image built from this `Dockerfile`:

    ```dockerfile
    FROM python:3.13-slim
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install -r requirements.txt
    COPY . .
    CMD ["python", "app.py"]
    ```

    Docker builds a stack of layers for the application file system:

    ```text
    Top
    ┌──────────────────────────────┐
    │ Application source code      │  COPY . .
    ├──────────────────────────────┤
    │ Installed Python packages    │  RUN pip install ...
    ├──────────────────────────────┤
    │ requirements.txt             │  COPY requirements.txt .
    ├──────────────────────────────┤
    │ Python and base filesystem   │  FROM python:3.13-slim
    └──────────────────────────────┘
    Bottom
    ```

    They are stacked because it make builds more efficient. If only the application source code changes, Docker can reuse the unchanged lower layers and rebuild only the final source-code layer. Instructions such as `CMD` do not add application files; they add metadata such as "the default process to run" to the image configuration stored alongside the layered filesystem.

- A **container** is a runtime instance of an image. It adds a small writable layer to the image and executes the configured process in isolation. The container's writable layer should be treated as temporary. Durable data such as database files or user uploads should be stored in a volume or external service rather than only inside the container.

    ```text
    Container view:  /app/config.json
                            |
                combined filesystem
                            |
            +-------------------------------+
            | Writable layer                | ← container changes
            +-------------------------------+
            | Application image layer       |
            +-------------------------------+
            | Runtime image layer           |
            +-------------------------------+
            | Base OS image layer           |
            +-------------------------------+

    ```

    Conceptually:

    ```text
    Dockerfile + application files
                |
                | docker build
                v
            Image: my-app:a1b2c3
            /        |        \
    docker run   docker run   docker push
        |            |             |
    Container A   Container B       v
                                ECR
    ```

    Many containers can be created from the same image. Deleting a container does not delete its image. Conversely, replacing an image does not modify containers that are already running from the old image. A container orchestrator deploys a new version by starting new containers from the new image and terminating the old containers.

### Dockerfile

A `Dockerfile` is a text file containing the instructions Docker uses to build an image. For the example Python application, it could contain:

```dockerfile
# Start with an existing image that already contains Python.
FROM python:3.12-slim

# Makes an absolute directory /app inside the image's filesystem
WORKDIR /app

# Copy and install dependencies into /app(denoted by the . here) before copying frequently changing source code.
# This lets Docker reuse the dependency layer when requirements.txt is unchanged.
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy contents of the src/ directory on the same level as the Dockerfile into the /app/src directory 
COPY src/ ./src/

# Documents the port the application expects to use.
EXPOSE 8000

# Default process started when a container is created from the image.
CMD ["python", "-m", "src.app"]
```

Each build instruction produces or contributes to a cached image layer. Ordering stable inputs before frequently changing inputs allows unchanged layers to be reused and makes builds faster. A `.dockerignore` file should exclude files that do not belong in the build context, such as `.git/`, local virtual environments, caches, and secrets.

`EXPOSE` documents the intended container port; it does not publish that port on the host. Port mapping happens when the container is run.

### Build, Run, Tag, and Push

The same image can move through the complete pipeline:

```bash
# Build from the Dockerfile and current directory, then give the image a local tag.
docker build -t my-app:a1b2c3 .

# Create and start a detached container. Host port 8000 forwards to container port 8000.
docker run -d --name my-app-local -p 8000:8000 my-app:a1b2c3

# Add the fully qualified ECR name to the same local image.
docker tag my-app:a1b2c3 \
  <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-app:a1b2c3

# Upload the image layers and manifest after authenticating to ECR.
docker push \
  <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-app:a1b2c3
```

The parts of the final image reference are:

```text
<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com / my-app : a1b2c3
|                                                    |          |
registry                                         repository    tag
```

A **tag** is a convenient name for an image version. A commit SHA makes a useful CI tag because it connects the built image to the exact source revision. Tags can be movable unless the registry enforces tag immutability; the image's content digest, such as `sha256:...`, is its immutable content identity.

### Container Registry

A **container registry** is a centralized service for storing and distributing Docker and OCI (Open Container Initiative) images. Images can be pushed to a registry and later pulled onto other machines. **Docker Hub** and **Amazon Elastic Container Registry (ECR)** are examples of container registries.

ECR can form the handoff between CI and deployment:

```text
CI runner                         ECR                       Production worker
build image  -- docker push -->  stored image  -- pull --> start container
                                  tag/digest
```

The CI runner does not deploy a container running on the CI machine. Instead, it publishes an image, and the production worker pulls that image when the orchestrator creates a new workload.

An image digest identifies immutable image content. A tag such as `my-app:v1` or `my-app:latest` is a convenient name that may be reassigned unless tag immutability is enabled.

ECR also provides AWS IAM-based access control, image scanning, lifecycle policies, and regional storage. It can store Docker images, OCI images, and other OCI-compatible artifacts, but it does not run them.

## Container Orchestrator

An automated CD pipeline does not require a other orchestrator. For a small application, CD could connect to one server, pull the new image, stop the old container, and start a new one. The pipeline must then implement availability, replacement, scaling, networking, and rollback behavior itself.

A **container orchestrator** manages containerized workloads across a pool of compute resources. Instead of telling a particular machine to run a particular command, the operator(developer) declares a desired state such as "run three healthy instances of this image." The orchestrator continually compares the actual state with the desired state and takes action to reconcile them. In context of CD pipeline, the orchestrator usually pulls images and runs them as directed by CD.

Typical orchestration responsibilities include:

- **Scheduling:** choosing compute capacity with enough CPU, memory, and other required capabilities for each workload.
- **Self-healing:** replacing failed containers and rescheduling workloads from failed machines.
- **Replicas and scaling:** maintaining or changing the desired number of application instances.
- **Rolling updates/deployments and rollback:** gradually replacing one application version with another while preserving availability.
- **Service discovery and load balancing:** giving changing workload instances a stable way to find and reach one another.
- **Declarative configuration:** storing the intended runtime state in version-controlled configuration rather than as a sequence of manual commands.

### Kubernetes

**Kubernetes** is a widely used open-source container orchestrator. It provides a standard API and a set of controllers that reconcile the declared state of resources with the cluster's actual state. Kubernetes can run on physical machines, on-premises VMs, or cloud VMs such as Amazon EC2 instances.

#### Cluster Architecture

A **cluster** is the complete Kubernetes environment. It has two main parts:

```text
Kubernetes cluster
├── Control plane
│   ├── API server: receives kubectl and automation requests
│   etcd
│   │ └── stores Kubernetes resource definitions/state
│   ├── scheduler: assigns unscheduled Pods to suitable nodes
│   └── controllers: reconcile actual state with desired state
└── Worker nodes (data plane)
    ├── kubelet: makes sure assigned Pods are running
    ├── container runtime: pulls images and runs containers
    └── Pods: application workloads
```

Both **control plane** and **worker nodes** ultimately run on physical machines or VMs. The difference is their job:
The control plane manages the cluster;
Worker nodes provide the CPU, memory and other resources where application containers actually run.

```text
Kubernetes Cluster
│
├── Control Plane
│
├── Worker Node 1
│   ├── Pod 1
│   │   └── Container A
│   │
│   ├── Pod 2
│   │   ├── Container B
│   │   └── Container C
│   │
│   └── Pod 3
│       └── Container D
│
└── Worker Node 2
    └── Pod 4
        ├── Container E
        ├── Container F
        └── Container G
```

A worker node could further have one or more **Pods** as per configuration. A pod is the smallest deployable unit in kubernetes. A pod acts as a wrapper around one or more **containers** that kubernetes treats as one unit.

Why that wrapper is useful:

- Containers in the same pod are always scheduled onto the same worker node.
- They can share the same network identity/IP address.
- They can share volumes/storage.
- Kubernetes can start, stop, replace, and scale the whole group together.
- It supports patterns where one container assists another, such as a main application container plus a logging or proxy sidecar.

When a Pod is created, the scheduler filters and scores available nodes using resource requests and constraints. It records the selected node, and that node's kubelet asks its container runtime to pull the specified image from a registry such as ECR and start the containers.

During a **rolling update**, a Deployment gradually creates Pods using the new image and removes Pods using the old image. Readiness checks determine when new Pods are eligible to receive traffic. The rolling-update strategy can control how many extra Pods may be created and how many desired Pods may be unavailable during the transition. Kubernetes performs this rollout behavior; the CD pipeline only changes the desired image and waits for the result.

```text
Pod A → v1
Pod B → v1
Pod C → v1

    ↓

Pod A → v1
Pod B → v1
Pod C → v1
Pod D → v2   ← new Pod

    ↓

Pod B → v1
Pod C → v1
Pod D → v2

    ↓

Pod C → v1
Pod D → v2
Pod E → v2

    ↓

Pod D → v2
Pod E → v2
Pod F → v2
```

#### Resource Model

In Kubernets, a **resource** is an API object that defines the desired state of something in the cluster. It represents something you want Kubernetes to create, manage, or keep track of in the cluster. They are defined declaratively in `.yaml` files.

For example, when you create a Deployment, Service, ConfigMap, or Secret, you are creating Kubernetes resources.

### Orchestration Options on AWS

The orchestrator and the compute it controls are separate choices. Kubernetes or ECS determines the orchestration API and scheduling behavior, while EC2 or Fargate supplies the CPU and memory on which application containers run.

Before an orchestrator can run application workloads, the team must provision and configure the AWS-side platform resources on which it depends. Depending on the architecture, these can include an EKS or ECS cluster, networking such as VPCs and subnets, IAM roles and permissions, EC2 worker capacity and scaling rules, or Fargate configuration. These resources must be managed so the platform can be created, scaled, upgraded, replaced, and reproduced consistently across environments. This infrastructure configuration is separate from describing application workloads through Kubernetes manifests or ECS task and service definitions.

AWS infrastructure can be managed through the AWS Console, AWS CLI, `eksctl` for EKS-specific tasks, or infrastructure-as-code tools such as Terraform. A team may use the Console while learning, use a CLI for inspection or one-off operations, and keep its authoritative production configuration in infrastructure as code. Regardless of the interface, a resource such as an EKS cluster or managed node group exists in AWS independently of the tool that created it.

With that foundation, there are three common approaches to running a container orchestrator on AWS. Each places a different amount of operational responsibility on AWS and the team:

#### Self-Managed Kubernetes on EC2

With **self-managed Kubernetes on EC2**, the team provisions EC2 instances for the control plane and worker nodes and installs Kubernetes, commonly using a tool such as `kubeadm` or a Kubernetes distribution. The team is responsible for control-plane availability, `etcd` backups, certificates, upgrades, security patches, networking plugins, worker-node lifecycle, monitoring, and AWS integrations.

This provides the most control and uses standard Kubernetes APIs, but has the greatest setup and operational burden. Choose it only when requirements justify control that EKS cannot provide and the team can reliably operate the complete platform. For production availability, multiple control-plane instances are normally required.

There is no EKS cluster fee, but this does not make the control plane free. The team pays for the EC2 instances, storage, and other AWS resources used by both the control plane and worker nodes, including spare capacity kept for availability and failure recovery. The total cost must also account for the engineering time, monitoring, backups, patching, upgrades, and incident risk involved in operating Kubernetes.

#### Amazon EKS

**Amazon Elastic Kubernetes Service (EKS)** is AWS's managed Kubernetes service. It runs conformant Kubernetes, so applications are still described and operated through standard Kubernetes resources and tools such as Pods, Deployments, Services, manifests, Helm, `kubectl`, and the Kubernetes API.

Compared with self-managed Kubernetes, EKS shifts responsibility for the control plane to AWS. AWS supplies its underlying resources and operates, scales, patches, and keeps the control plane highly available. The team remains responsible for defining its Kubernetes workloads and selecting the data-plane compute on which their Pods run.

EKS charges an hourly fee for the managed control plane of each cluster. This fee includes the control-plane resources and their management, so the team does not provision or pay separately for control-plane EC2 instances. Data-plane compute is charged separately.

For a standard EKS cluster, the common data-plane choices are:

- **EC2 worker nodes:** Kubernetes schedules multiple Pods onto EC2 instances according to their available capacity. Through one of the AWS configuration interfaces described above, the team creates a managed node group and chooses its permitted EC2 instance types and minimum, desired, and maximum number of nodes. These are AWS infrastructure settings; Pod manifests instead declare CPU and memory requests, which the Kubernetes scheduler uses when placing Pods. Managed node groups automate node provisioning, replacement, and updates, but do not automatically change the node count in response to unschedulable Pods. Cluster Autoscaler can adjust the size of a managed node group within its configured limits. Alternatively, Karpenter can use Kubernetes `NodePool` and `EC2NodeClass` resources to select and launch suitable EC2 instances directly from pending Pod requirements instead of scaling a predefined node group. The cost is **EKS cluster fee + EC2 data-plane resources**. EC2 instances are billed while provisioned even when some capacity is idle, although they can be economical for steady workloads that use node capacity efficiently.

- **Fargate:** AWS provides data-plane compute on demand for selected Pods, so the team does not manage a fleet of worker nodes. Each Pod declares its CPU and memory requirements and runs in an isolated Fargate environment. The cost is **EKS cluster fee + the vCPU, memory, and storage allocated to each running Fargate Pod**. Fargate reduces node operations and can suit variable workloads, but it provides less host-level control, has workload constraints that must be checked, and may cost more than well-utilized EC2 at sustained scale.

Choose EKS when the team needs Kubernetes APIs, tooling, ecosystem, or transferable skills without assuming responsibility for operating the Kubernetes control plane.

#### Amazon ECS

**Amazon Elastic Container Service (ECS)** is a managed, AWS-native container orchestrator. It is not a Kubernetes distribution: applications are described and operated through ECS resources and APIs rather than Kubernetes manifests, tools, or APIs. Its primary abstractions are:

| ECS concept | Purpose | Rough Kubernetes analogy |
|---|---|---|
| Task definition | Versioned blueprint describing images, resources, ports, roles, and configuration | Pod template |
| Task | Running instance of a task definition containing one or more containers | Pod |
| Service | Maintains a desired task count and performs deployments | Deployment and Service behavior |
| Cluster | Logical grouping of services, tasks, and capacity | Cluster |

The analogies are only conceptual; the APIs and features do not map one-to-one. ECS usually requires fewer platform decisions and integrates directly with AWS services such as IAM, Elastic Load Balancing, and CloudWatch. In exchange, ECS workload configuration and operational knowledge are AWS-specific rather than portable Kubernetes configuration.

ECS shifts responsibility for the orchestration control plane to AWS. AWS operates the control plane, while the team defines task definitions and services and selects the data-plane compute on which its tasks run.

Unlike EKS, the standard ECS options do not have a separate per-cluster or orchestration fee. AWS operates the ECS control plane without an additional charge; data-plane compute is charged separately.

For an ECS cluster, the common data-plane choices are:

- **EC2 container instances:** ECS schedules multiple tasks onto a fleet of EC2 instances according to their available capacity. The team can manage the fleet directly or connect it to ECS through a capacity provider. The cost is **EC2 data-plane resources without a separate ECS fee**. EC2 instances are billed while provisioned even when some capacity is idle, although they can be economical for steady workloads that use instance capacity efficiently.

- **Fargate:** AWS provides data-plane compute on demand for each task, so the team does not manage a fleet of container instances. Each task declares its required vCPU, memory, and storage and runs in an isolated Fargate environment. The cost is **the resources allocated to each running Fargate task without a separate ECS fee**. Fargate reduces infrastructure operations and can suit variable workloads, but it provides less host-level control and may cost more than well-utilized EC2 at sustained scale.

Choose ECS when the workload will remain on AWS and the team values a smaller platform surface and direct AWS integration over Kubernetes compatibility.

As with the other orchestration choices, supporting resources can add charges for load balancers, EBS volumes, public IPv4 addresses, NAT gateways, CloudWatch, ECR storage, and data transfer.

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
