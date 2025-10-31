# Certified ARGO Project Associate (CAPA)
Argo is a suite of Kubernetes-native tools that streamline workflow automation and application delivery. It includes:

- **Argo CD:** Declarative GitOps continuous delivery.
- **Argo Workflows:** Orchestrates complex jobs and pipelines.
- **Argo Events:** Automates workflows in response to events.
- **Argo Rollouts:** Enables progressive delivery strategies.

---

# Argo Continuous Delivery (ArgoCD)

Argo CD is a declarative GitOps tool for Kubernetes that automates application and infrastructure deployments. It continuously syncs your cluster state with your Git repository, ensuring consistency and traceability. Key features include:

- **Automated Deployments:** Applies manifests from Git to your cluster.
- **Progressive Delivery:** Supports blue-green and canary strategies.
- **Traffic Management:** Integrates with service meshes and ingress controllers.
- **Safe Rollbacks:** Automates promotion and rollback based on deployment analysis.

Argo CD streamlines and secures the delivery of applications to production environments.

## ArgoCD Components
![alt text](images/image-01.png)

### API Server
The **API Server** in Argo CD acts as the central control plane, providing a unified interface for managing and monitoring your applications. Its core responsibilities include:

- **Application Management:** Manages applications and provides real-time status updates.
- **Operational Control:** Triggers operations (such as sync, rollback, or refresh) on applications as needed.
- **Git Integration:** Handles Git repositories for version control and declarative configuration.
- **Cluster Connectivity:** Manages secure connections with Kubernetes clusters.
- **Authentication & SSO:** Supports authentication mechanisms, including Single Sign-On (SSO).
- **Access Control:** Enforces Role-Based Access Control (RBAC) policies for secure, granular permissions.
- **Central Communication Hub:** Serves as the main point of interaction for the Web UI, CLI, Argo Events, and CI/CD systems.

### Repository Server
The **Repository Server** works closely with the API Server to manage application source code. Its main responsibilities include:

- **Git Retrieval:** Connects to your Git repositories to fetch the latest application manifests and configuration files.
- **Manifest Packaging:** Processes and packages the retrieved files into a format that Kubernetes can understand and apply.
- **Local Caching:** Maintains a secure, up-to-date local cache of repository contents to optimize performance and reduce redundant network calls.
- **Seamless Integration:** Ensures that Argo CD always has access to the desired state of your applications, enabling reliable and efficient deployments.


### Application Controller
The Argo CD Application Controller is another crucial component. It continuously compares the desired application state (as defined in your Git repositories) with the live state in your Kubernetes cluster. If it detects any discrepancies, it will take corrective action to ensure that the live state matches the desired state.

## Reconciliation Loop
![alt text](images/image-02.png)

The Argo CD reconciliation loop embodies the principles of GitOps by maintaining a Git repository as the single source of truth for infrastructure definitions. This process aligns with the GitOps principles of having a declarative desired state, which is versioned and immutable, and is automatically pulled by software agents. The agents continuously observe the actual system state and attempt to reconcile it with the desired state, mirroring the continuous reconciliation principle of GitOps. Thus, the Argo CD reconciliation loop is a practical implementation of GitOps principles.


## Synchronization Principles

The sync phase is a critical operation in Argo CD, and its behavior can be tailored using **resource hooks** and **sync waves**. This section explains both customization methods and how to use them effectively.

### Resource Hooks

A *sync* is the process of transitioning an application to its desired state. Resource hooks allow you to run custom logic at specific points during this process. There are five hook types:

- **PreSync:** Runs *before* the sync phase (e.g., create a backup before syncing).
- **Sync:** Runs *during* the sync phase, after all PreSync hooks succeed (e.g., implement advanced rollout strategies like blue-green or canary).
- **PostSync:** Runs *after* a successful sync, when all resources are healthy (e.g., perform health or integration checks).
- **Skip:** Instructs Argo CD to skip applying the manifest.
- **SyncFail:** Runs *if* the sync fails (e.g., perform cleanup operations).

Resource hooks are typically implemented as Kubernetes `Job` resources, identified by a special annotation. Argo CD uses this annotation to determine when to execute the job.

**Example: Database Schema Migration Hook**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  generateName: schema-migrate-
  annotations:
    argocd.argoproj.io/hook: PreSync
```


### Sync waves

Essentially, sync waves are a convenient way to split and order to-be-synced manifests. Waves can range from negative to positive values. If not defined, the default value is wave zero. Using sync waves is achieved in the same way as resource hooks—by using annotations.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  generateName: schema-migrate-
  annotations:
    argocd.argoproj.io/sync-wave: "5"
```

### Resource hooks and sync waves

Both approaches can be utilized simultaneously. The sync operation in Argo CD initiates with the ordering of resources based on the following criteria:

1. **Resource hook phase annotation**
2. **Sync wave annotation**
3. **Kubernetes kind** (for example, namespaces are prioritized, followed by other Kubernetes resources and then custom resources)
4. **Name**

Afterwards, the next wave number to be applied is identified. This number is always the lowest among any out-of-sync or unhealthy resources. Argo CD then applies this wave.

This process is repeated until all phases and waves are in sync and healthy.

> **Note:** If an application has unhealthy resources in the initial wave, it may prevent the application from ever reaching a healthy state.

For safety reasons, Argo CD adds a **2-second delay** between each sync wave. This can be customized using the `ARGOCD_SYNC_WAVE_DELAY` environment variable.

## CRDs and Resources

### Application

Argo CD introduces the **Application CRD**, serving as a representation of the application instance you intend to deploy in your Kubernetes cluster. Think of it as the blueprint for your application within the specified environment.

**Example:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
    name: guestbook
    namespace: argocd
spec:
    project: default
    source:
        repoURL: 'htt‌ps://github.com/argoproj/argocd-example-apps.git'
        targetRevision: HEAD
        path: guestbook
    destination:
        server: 'htt‌ps://kubernetes.default.svc'
        namespace: guestbook
```


### AppProject

For efficient organization, Argo CD provides the **AppProject CRD**. This allows you to group related applications. An example use case involves segregating applications from utility services, enhancing the clarity of your project structure.

**Example:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
    name: production
    namespace: argocd
spec:
    description: Production applications
    sourceRepos:
        - '*'
    destinations:
        - namespace: production
            server: 'htt‌ps://kubernetes.default.svc'
    clusterResourceWhitelist:
        - group: '*'
            kind: '*'
```

### Repository Credentials
In real-world scenarios, accessing private repositories is common. Argo CD facilitates this by using Kubernetes Secrets and ConfigMaps. To grant Argo CD access, create the necessary Secret Kubernetes resources with a specific label: `argocd.argoproj.io/secret-type: repository`.  
**Example:**

```yaml
apiVersion: v1
kind: Secret
metadata:
    name: private-repo-creds
    namespace: argocd
    labels:
        argocd.argoproj.io/secret-type: repository
stringData:
    url: 'htt‌ps://github.com/private/repo.git'
    username: <username>
    password: <password>
```

---

### Cluster Credentials

In cases where Argo CD manages multiple Kubernetes clusters, additional access may be required. For this purpose, Argo CD utilizes a distinct Secret type with the label: `argocd.argoproj.io/secret-type: cluster`. This ensures secure access to external clusters not initially included in Argo CD's managed environments.  
**Example:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: external-cluster-creds
  labels:
    argocd.argoproj.io/secret-type: cluster
type: Opaque
stringData:
  name: external-cluster
  server: 'ht‌tps://external-cluster-api.com'
  config: |
    {
      "bearerToken": "<token>",
      "tlsClientConfig": {
        "insecure": false,
        "caData": "<certificate encoded in base64>"
        }
    }
```


## Argo CD Extensions & Integrations
### Plugins (deprecated, removed in the new versions)
Argo CD plugins extend the core functionalities of the system, offering additional features beyond the default capabilities.
Plugins in Argo CD follow a specific lifecycle, including registration, initialization, and execution phases. During startup, Argo CD discovers and loads available plugins, initializing them for use throughout the application lifecycle. Critical events, such as application synchronization or deployment, trigger the associated plugins.

### Configuring Plugins with ConfigMaps
Let's explore the configuration of the Notifications plugin using a ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
data:
  context: |
    region: east
    environmentName: staging

  template.a-slack-template-with-context: |
    message: "Something happened in {{ .context.environmentName }} in the {{ .context.region }} data center!"
```

**In this example:**

- The `context` section provides contextual information such as the region and environment name.
- The `template.a-slack-template-with-context` section defines a Slack notification template using Go templating. It references values from the context section to personalize the message.

This ConfigMap enables users to dynamically adjust notification content based on contextual information, providing flexibility in responding to different deployment scenarios.





## Securing Argo CD

### Use RBAC
**Why:** RBAC controls user permissions, ensuring only authorized access to resources and reducing risk.

**How:** Define roles in the `argocd-rbac-cm` ConfigMap and assign them via RoleBindings. Follow least privilege, audit regularly, and test your setup.

### Manage Secrets Securely
**Why:** Protect sensitive data like API keys and passwords from exposure or misuse.

**How:** Store secrets using Kubernetes Secrets, enable encryption at rest and in transit, and consider integrating with external secret managers.

### Keep Argo CD Updated
**Why:** Updates fix vulnerabilities and improve security.

**How:** Regularly check for new releases, test updates in staging, and apply them using standard Kubernetes deployment methods.

> For more on SSO, audit logging, and network policies, see the [Argo CD Security Documentation](https://argo-cd.readthedocs.io/en/stable/operator-manual/security/).




## Practical Part
### Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

To access the Argo CD UI, forward the service port:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open [http://localhost:8080](http://localhost:8080) in your browser.

Get the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### Install ArgoCD cli
```bash
brew install argocd
argocd login localhost:8080
```

### Add new Users

1. **Add a New User:**
    - Edit the `argocd-cm` ConfigMap in the `argocd` namespace.
    - Under the `accounts` section, add your new username (e.g., `newuser: login`).
      ```
      data:
        accounts.newuser: login
        accounts.newuser2: login
      ```
    - Save and apply the updated ConfigMap.

2. **Set or Change the Password:**
    - Use the ArgoCD CLI to set the password:
      ```
      argocd account update-password --account newuser [--new-password 12345678]
      ```
    - You need admin credentials for the initial password setup.

3. **Apply Changes:**
    - If required, restart the ArgoCD server deployment to ensure changes take effect.

4. **Login:**
    - The new user can now log in via the CLI or web UI.

### Define the RBAC rules

Open the ConfigMap for editing RBAC rules with this command:

```bash
kubectl edit cm argocd-rbac-cm -n argocd
```

In here we want to create a new policy:

```yaml
data:
  policy.csv: |
    p,role:synconly,applications,sync,*/*,allow
    g,newuser,role:synconly
  policy.default: role:readonly
```

This policy defines the following:

- **`p,role:synconly,applications,sync,*/*,allow`:**  
    This line specifies a permission rule that allows only the sync action on applications. The role `synconly` is created for this purpose.

- **`g,newuser,role:synconly`:**  
    Associates the `newuser` group with the role `synconly`, granting them permission only to sync applications.

- **`policy.default: role:readonly`:**  
    Specifies the default policy for users who don't fall into any explicitly defined category. In this case, they have read-only access.

In ArgoCD, the RBAC rules can be intricately defined, allowing for granular control over access permissions. For a deeper understanding and detailed information on RBAC configurations, refer to the official documentation.

### Test the new rules

Attempt to login as the `developer` user and evaluate the access permissions. Verify if you can successfully synchronize applications. Additionally, explore whether actions such as deletion or other operations are restricted according to the RBAC rules assigned to the `developer`.


---

# Argo Workflows
Argo Workflows extends Kubernetes with a powerful Workflow CRD, enabling organizations to define, schedule, and manage complex workflows as code. Each workflow step runs as a separate pod, allowing for modular, scalable, and parallel execution—ideal for data processing, automation, and especially machine learning (ML) and data pipelines.

**Key Benefits:**
- **Highly Extensible:** Adaptable to a wide range of use cases, from simple batch jobs to intricate pipelines.
- **Parallelism:** Supports fan-out/fan-in patterns for efficient concurrent task execution and result aggregation.
- **Kubernetes-Native:** Seamlessly integrates with Kubernetes, leveraging its scalability and reliability.

Argo Workflows is a robust solution for orchestrating complex, automated processes in cloud-native environments.


## Core Concept

### Template Types

A template can be containers, scripts, DAGs, or other types depending on the workflow structure and is divided into two groups: **template definitions** and **template invocators**.

#### Template Definitions
**Container**

A Container is the most common template type and represents a step in the workflow that runs a container. It is suitable for executing containerized applications or scripts.

Example:

```yaml
- name: whalesay
  container:
    image: docker/whalesay
    command: [cowsay]
    args: ["hello world"]
```

**Resource**

A Resource represents a template for creating, modifying, or deleting Kubernetes resources. It is useful for performing operations on Kubernetes objects.

Example:

```yaml
- name: k8s-owner-reference
  resource:
    action: create
    manifest: |
      apiVersion: v1
      kind: ConfigMap
      metadata:
        generateName: owned-eg-
      data:
        some: value
```

**Script**

A Script is similar to the container template but allows specifying the script inline without referencing an external container image. It can be used for simple scripts or one-liners.

Example:

```yaml
- name: gen-random-int
    script:
        image: python:alpine3.6
        command: [python]
        source: |
            import random
            i = random.randint(1, 100)
            print(i)
```

**Suspend**

A Suspend is a template that suspends execution, either for a duration or until it is resumed manually. It can be resumed using the CLI, the API endpoint, or the UI.

Example:

```yaml
- name: delay
    suspend:
        duration: "20s"
```


#### Template Invocators

**DAG** 

A DAG allows defining our tasks as a graph of dependencies. It is beneficial for workflows with complex dependencies and conditional execution.

```yaml
- name: diamond
    dag:
        tasks:
        - name: A
            template: echo
        - name: B
            dependencies: [A]
            template: echo
        - name: C
            dependencies: [A]
            template: echo
        - name: D
            dependencies: [B, C]
            template: echo
```

**Steps**

Steps are defining multiple steps within a template as several steps need to be executed sequentially or in parallel.

```yaml
- name: hello-hello-hello
    steps:
    - - name: step1
            template: prepare-data
    - - name: step2a
            template: run-data-first-half
      - name: step2b
            template: run-data-second-half
```


### Outputs

In Argo Workflows, the **outputs** section within a step template allows you to define and capture outputs that can be accessed by subsequent steps or referenced in the workflow definition. Outputs are useful when you want to pass data, values, or artifacts from one step to another.

The Output comprises two key concepts:

- **Defining Outputs:**  
    You define outputs within a step template using the `outputs` section. Each output has a name and a path within the container where the data or artifact is produced.

- **Accessing Outputs:**  
    You can reference the outputs of a step using templating expressions in subsequent steps or the workflow definition.

Let’s consider a simple example where one step generates an output parameter and an output artifact, and another step consumes them:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
    generateName: artifact-passing-
spec:
    entrypoint: artifact-example
    templates:
    - name: artifact-example
        steps:
        - - name: generate-artifact
                template: whalesay
        - - name: consume-artifact
                template: print-message
                arguments:
                    artifacts:
                    - name: message
                        from: "{{steps.generate-artifact.outputs.artifacts.hello-art}}"

    - name: whalesay
        container:
            image: docker/whalesay:latest
            command: [sh, -c]
            args: ["cowsay hello world | tee /tmp/hello_world.txt"]
        outputs:
            artifacts:
            - name: hello-art
                path: /tmp/hello_world.txt

    - name: print-message
        inputs:
            artifacts:
            - name: message
                path: /tmp/message
        container:
            image: alpine:latest
            command: [sh, -c]
            args: ["cat /tmp/message"]
```

- First, the `whalesay` template creates a file named `/tmp/hello-world.txt` by using the `cowsay` command.
- Next, it outputs this file as an artifact called `hello-art`.
- The `artifact-example` template provides the generated `hello-art` artifact as an output of the `generate-artifact` step.
- Finally, the `print-message` template takes an input artifact called `message` and consumes it by unpacking it in the `/tmp/message` path and using the `cat` command to print it into standard output.


### WorkflowTemplate
In Argo Workflows, a **WorkflowTemplate** is a resource that defines a reusable and shareable workflow template, allowing users to encapsulate workflow logic, parameters, and metadata. This abstraction promotes modularity and reusability, enabling the creation of complex workflows from pre-defined templates.

---

**Example: Simple WorkflowTemplate Definition**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
    name: sample-template
spec:
    templates:
        - name: hello-world
            inputs:
                parameters:
                    - name: msg
                        value: "Hello World!"
            container:
                image: docker/whalesay
                command: [cowsay]
                args: ["{{inputs.parameters.msg}}"]
```

**Explanation:**

- The WorkflowTemplate is named `sample-template`.
- It contains a template called `hello-world`.
- The `hello-world` template takes a parameter `msg` (with a default value of "Hello World!") and generates a file with the specified message.

---

Once defined, this WorkflowTemplate can be referenced and instantiated within multiple workflows.

**Example: Referencing a WorkflowTemplate**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
    generateName: hello-world-
spec:
    entrypoint: whalesay
    templates:
        - name: whalesay
            steps:
                - - name: hello-world
                        templateRef:
                            name: sample-template
                            template: hello-world
```

This workflow references the WorkflowTemplate named `sample-template`, effectively inheriting the structure and logic defined in the template.

---

Using WorkflowTemplates is beneficial when you want to standardize and reuse specific workflow patterns, making it easier to manage, maintain, and share workflow definitions within your organization. They also help in enforcing consistency and reducing redundancy across multiple workflows.


## Architecture

Argo Workflows is an open source workflow orchestration platform designed for Kubernetes. It enables users to define, run, and manage complex workflows using Kubernetes as the underlying execution environment.

---

### Argo Server

The **Argo Server** is a central component that manages the overall workflow resources, state, and interactions. It exposes a REST API for workflow submission, monitoring, and management. The server maintains the state of workflows and their execution, and interacts with the Kubernetes API server to create and manage resources.

---

### Workflow Controller

The **Argo Workflows Controller** is a critical component within the Argo Workflows system. It is responsible for managing the lifecycle of workflows, interacting with the Kubernetes API server, and ensuring the execution of workflows according to their specifications.

- Continuously watches the Kubernetes API server for changes related to Argo Workflows Custom Resources (CRs).
- The primary CR involved is the **Workflow**, which defines the workflow structure and steps.
- Upon detecting the creation or modification of a Workflow CR, the controller initiates the corresponding workflow execution.
- Manages the complete lifecycle of a workflow: creation, execution, monitoring, and completion.
- Resolves dependencies between steps within a workflow, ensuring steps are executed in the correct order based on dependencies specified in the workflow definition.

---

Both the Workflow Controller and Argo Server run in the `argocd` namespace. You can opt for either cluster or namespaced installations; the generated Workflows and Pods will run in the respective namespace.

---

### Argo UI

The **Argo UI** is a web-based user interface for visually monitoring and managing workflows. It allows users to:

- View workflow status, logs, and artifacts
- Submit new workflows


### Example
The diagram below shows an overview of a Workflow and also details of a namespace with generated pods.

![alt text](images/image-03.png)
- A user defines a workflow using YAML or JSON files, specifying the sequence of steps, dependencies, inputs, outputs, parameters, and any other relevant configurations.

- The workflow definition file is then submitted to the Kubernetes cluster where Argo Workflows is deployed. This submission can be done via the **Argo CLI**, **Argo UI**, or programmatically through Kubernetes API clients.

- The **Workflow Controller** component of Argo Workflows continuously monitors the Kubernetes cluster for new workflow submissions or updates to existing workflows. When a new workflow is submitted, the Workflow Controller parses the workflow definition to validate its syntax and semantics. If there are any errors or inconsistencies, the Workflow Controller reports them to the user for correction.

- Once the workflow definition is validated, the Workflow Controller creates the necessary Kubernetes resources to represent the workflow, such as **Workflow CRDs** (Custom Resource Definitions) and associated **Pods**, **Services**, **ConfigMaps**, and **Secrets**.

- Finally, the Workflow Controller begins executing the steps defined in the workflow. Each step may involve running containers, executing scripts, or performing other actions specified by the user. Argo Workflows ensures that steps are executed in the correct order based on dependencies defined in the workflow.


### Argo Workflow Overview

Each **Step** and **DAG** in Argo Workflows generates a Pod consisting of three containers:

- **init:**  
    A template that contains an init container performing initialization tasks. In this example, it echoes a message and sleeps for 30 seconds, but you can replace these commands with your actual initialization steps.

- **main:**  
    A template containing the main container that executes the primary process once initialization is complete.

- **wait:**  
    A container responsible for tasks such as cleanup, saving parameters, and handling artifacts.

## Use Case Examples

Argo Workflows is a versatile tool with a wide range of use cases in Kubernetes and containerized environments. Here are some common scenarios where Argo Workflows excels:

- **Data Processing Pipelines:**  
    Orchestrate end-to-end data processing workflows, including extraction, transformation, and loading (ETL) tasks.

- **Machine Learning Automation:**  
    Manage ML pipelines for data preprocessing, model training, evaluation, and deployment.

- **CI/CD Pipelines:**  
    Automate building, testing, and deploying applications in Kubernetes, serving as the backbone for continuous integration and delivery.

- **Batch Processing & Scheduled Tasks:**  
    Run batch jobs and periodic tasks at specified intervals or based on cron schedules—ideal for automating routine operations, report generation, and scheduled jobs.

## Practical
### Install Argo Workflows

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm upgrade -i argo-workflows argo/argo-workflows -n argo --create-namespace -f argo-workflows/values.yaml
```

To access the Argo Workflows UI, forward the server port:

```bash
kubectl -n argo port-forward deployment/argo-workflows-server 2746:2746
```

Argo Workflows will be available at [http://localhost:2746](http://localhost:2746)

### Install argo cli
```bash
ARGO_OS="darwin"; [ "$(uname -s)" != "Darwin" ] && ARGO_OS="linux"
curl -sLO "https://github.com/argoproj/argo-workflows/releases/download/v3.7.0/argo-$ARGO_OS-amd64.gz"
gunzip "argo-$ARGO_OS-amd64.gz"
chmod +x "argo-$ARGO_OS-amd64"
sudo mv "./argo-$ARGO_OS-amd64" /usr/local/bin/argo

source <(argo completion zsh)

# test
argo version
```

Here is a list of the most common commands to operate with Argo Workflows:

| Command                               | Description                                 |
|---------------------------------------|---------------------------------------------|
| `argo list`                           | List workflows                              |
| `argo get <workflow-name>`            | Display details about a workflow            |
| `argo submit <workflow-file.yaml>`    | Submit a workflow                           |
| `argo watch <workflow-name>`          | Watch workflow progress                     |
| `argo template list`                  | List workflow templates                     |
| `argo logs <workflow-name>`           | Display logs for workflow pod(s)            |


### Practical DAG Example
**1. Install Resources**

To successfully deploy this Workflow, we need to temporarily grant admin permissions to the `argo-workflow` ServiceAccount as follows:

```bash
kubectl create rolebinding default-admin --clusterrole=admin \
    --serviceaccount=argo:argo-workflow -n argo
```

> For a production environment, please follow the [official documentation](https://argoproj.github.io/argo-workflows/).

---

**Next steps:**
1. **Create the WorkflowTemplate shown above:**

     ```bash
     kubectl apply -f argo-workflows/examples/dag-workflow-template.yaml
     ```

2. **Submit the DAG workflow to your Kubernetes cluster:**

     ```bash
     kubectl create -f argo-workflows/examples/dag-workflow.yaml
     ```

> In this workflow:
>
> - **Step A** runs first, since it has no dependencies.
> - After **A** finishes, **steps B and C** run simultaneously/parallel.
> - When **B and C** are completed, **step D** starts.
>
> You can also view the completed steps in **Argo UI**.


**2. Inspect Workflow**

Check whether the WorkflowTemplate and Workflow resources have been created:

```bash
kubectl -n argo get workflowtemplates
kubectl -n argo get workflow
```

You can achieve the same by using the Argo Workflow CLI as follows:

```bash
argo -n argo list
```

Now let’s review the details of the Workflow that we just submitted. The output for the command below will be the same as the information shown when you submitted the Workflow:

```bash
argo -n argo get dag-diamond-gvr2w
argo -n argo logs dag-diamond-gvr2w
```

**3. Clean Up the Resources**

We successfully created an Argo Workflow. To keep our working cluster nice and clean, we are going to clean up the resources we created:

```bash
kubectl -n argo delete workflow dag-diamond-gvr2w
kubectl -n argo delete workflowtemplates.argoproj.io echo-template
```


### Practical CI/CD Example
We can start by creating and managing our workflows as code. We will create a Python application workflow with the following steps:

- **Build:** The build step builds the image with the latest changes, using a Python 3 image for this scenario.
- **Tests:** The test step mounts a volume with test files and runs unit tests with the Python unittest library.
- **Deployment:** The deploy step runs the Python container and prints deploy. Normally, this step would involve pushing the tested code to a container registry, like AWS ECR or Harbor, and then deploying it to the production environment.

This lab provides a basic setup for Argo Workflow CI/CD. Depending on your specific requirements, you may want to enhance the workflow with additional steps, such as testing, linting, or pushing the Docker image to a registry.

**1. Deploy Argo Workflow**

Deploy the workflow by running the following command:

```bash
kubectl -n argo create -f argo-workflows/examples/ci-cd-python-workflow.yaml
```

This will create an Argo Workflow named `python-app` along with its respective templates and steps.

---

**2. Inspect the Workflow**

Retrieve information about the workflow, including its status:

```bash
argo -n argo get python-app-sdfsd
argo -n argo-workflows logs python-app-sdfsd
```


# Argo Rollouts

Argo Rollouts is a Kubernetes-native controller that enables advanced progressive delivery strategies, such as blue-green and canary deployments. It integrates with service meshes and ingress controllers for sophisticated traffic management, automates promotion and rollback based on deployment analysis, and ensures safe, reliable releases to production. Argo Rollouts is ideal for teams seeking to enhance deployment safety and flexibility within a GitOps workflow.


## Progressive Delivery

### Continuous Integration

Continuous Integration is a development practice where developers frequently integrate their code into a shared repository, preferably several times daily. Each integration is then verified by an automated build and automated tests.

#### CI Features

- **Frequent code commits**  
    Encourage developers to often integrate their code into the main branch, reducing integration challenges.

- **Automated tests**  
    Cover frequent code commits. Automatically running tests on the new code to ensure it integrates well with the existing codebase. This does not only include unit tests, but also any other higher-order testing method, such as integration- or end-to-end tests.

- **Immediate problem detection**  
    Allows for quick detection and fixing of integration issues.

- **Reduced integration problems**  
    Help to minimize the problems associated with integrating new code.

The main goal of CI is to provide rapid feedback so that if a defect is introduced into the code base, it is identified and corrected as soon as possible.

Once code is in our main branch, it is not deployed in production or even released. This is where the concept of Continuous Delivery comes into play.


### Continuous Delivery

Continuous Delivery is an extension of CI, ensuring the software can be reliably released anytime. It involves the automation of the entire software release process.

#### CD Features

- **Automated release process:**  
    Every change that passes the automated tests can be released to production through an automated process.

- **Reliable deployments:**  
    Ensure that the software is always in a deployable state.

- **Rapid release cycles:**  
    Facilitate frequent and faster release cycles.

- **Close collaboration between teams:**  
    A close alignment between development, QA, and operations teams is required.

The objective of Continuous Delivery is to establish a process where software deployments become predictable, routine, and can be executed on demand.



### Progressive Delivery

Progressive delivery is often described as an evolution of continuous delivery. It focuses on releasing updates of a product in a controlled and gradual manner, thereby reducing the risk of the release, typically coupling automation and metric analysis to drive the automated promotion or rollback of the update.

**Progressive Delivery Features**

- **Canary releases:**  
    Gradually roll out the change to a small subset of users before rolling it out to the entire user base.

- **Feature flags:**  
    Control who gets to see what feature in the application, allowing for selective and targeted deployment.

- **Experiments & A/B testing:**  
    Test different versions of a feature with different segments of the user base.

- **Phased rollouts:**  
    Slowly roll out features to incrementally larger segments of the user base, monitoring and adjusting based on feedback.

The primary goal of Progressive Delivery is to reduce the risk associated with releasing new features and to enable faster iteration by getting early feedback from users.

### Deployment Strategies

Every software system is different, and deploying complex systems often requires additional steps and checks. This is why various deployment strategies have emerged to manage the process of releasing new software versions in production environments.

These strategies are integral to DevOps practices, especially within CI/CD workflows. The choice of deployment strategy can significantly affect the availability, reliability, and user experience of an application or service.

On the following pages, we will present the four most important deployment strategies and discuss their impact on user experience during deployment:

- **Recreate/Fixed Deployment**
- **Rolling Update**
- **Blue-Green Deployment**
- **Canary Deployment**

#### Recreate/Fixed Deployment
A **Recreate deployment** deletes the old version of the application before bringing up the new version. This ensures that two versions of the application never run at the same time, but there is downtime during the deployment. This strategy is also supported by the Kubernetes Deployment object.

![Distribution of old and new versions using the fixed deployment strategy](images/image-04.png)

---

#### Rolling Update

A **Rolling Update** slowly replaces the old version with the new version. As the new version comes up, the old version is scaled down to maintain the overall count of the application. This reduces downtime and risk as the new version is gradually deployed. This is the default strategy of the Kubernetes Deployment object.

![Distribution of old and new versions using the rolling update strategy](images/image-05.png)


#### Blue-Green Deployment

A blue-green deployment (sometimes referred to as a red/black) has both the new and old versions of the application deployed at the same time. During this period, only the old version receives production traffic. This setup allows developers to run tests against the new version before switching live traffic to it. Once the new version is ready and tested, traffic is switched (often at the load balancer level) from the old environment to the new one. The main advantage is a quick rollback in case of issues and minimal downtime during deployment.

> **Note:**  
> An important drawback of blue-green deployment is that twice the number of instances are created during the deployment. This can be a significant limitation for some environments.

![Distribution of old and new versions using the blue-green deployment strategy](images/image-06.png)


#### Canary Deployment

A small subset of users are directed to the new version of the application while the majority still use the old version. Based on the feedback and performance of the new version, the deployment is gradually rolled out to more users. This reduces risk by affecting a small user base initially, allows for A/B testing and real-world feedback.

![Distribution of old and new versions using the canary deployment strategy](images/image-07.png)


### Strategies for Smooth and Reliable Releases

In summary, deployment strategies are fundamental in modern software development and operations for ensuring smooth, safe, and efficient software releases. They cater to the need for balancing rapid deployment with the stability and reliability of production environments.

#### Benefits of Introducing Deployment Strategies

| **BENEFIT**          | **DESCRIPTION**                                                                                                                                                                                                 |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Risk mitigation       | They allow for safer deployments by reducing the risk of introducing bugs or performance issues into the production environment. Strategies like canary deployments enable gradual exposure to new changes.     |
| User experience       | Maintaining a consistent and high-quality user experience is essential. Strategies like blue–green deployments minimize downtime and potential disruptions to the user experience.                              |
| Feedback and testing  | They provide a framework for gathering real-world user feedback. Canary deployments, in particular, are valuable for understanding how changes perform in a live environment.                                   |
| Rollback capabilities | In case new versions have critical issues, strategies like blue–green deployments allow for quick rollbacks to the previous stable version.                                                                     |

#### Common Use Cases for Each Strategy

| **STRATEGY**          | **SUPPORTED BY**   | **COMMON USE CASES**                                                                                                                                                                                                                  |
|-----------------------|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Fixed deployment      | Kubernetes Native  | • The most basic way to deploy a workload is whenever downtime is acceptable.<br>• Often stateful workloads (e.g., Databases) require a “recreation” to avoid data corruption.                                                         |
| Rolling update        | Kubernetes Native  | • Commonly used for stateless, low-maintenance workloads like proxies, RESTful APIs, etc.                                                                                                      |
| Blue-green deployment | Argo Rollouts      | • Use when a) you can afford the extra cost of running twice the resources and b) need a quick and easy rollback option.<br>• B/G can also be helpful for experimentation scenarios.<br>• Can be advantageous to update services that depend on stateful connections, e.g., via WebSockets. |
| Canary deployment     | Argo Rollouts      | • Use it whenever a partial rollout is desirable (experimentation with a subset of users, desire a gradual rollout over hours or days, want to make rollout dependent on certain conditions).<br>• It can be a good alternative if the deployments are too large and the infra cost of running a full blue-green is too high. |

---




# Argo Events

Argo Events is a Kubernetes-native framework for building event-driven automation. It connects external event sources (like webhooks, S3, schedules, and message queues) to triggers that launch workflows or other actions.

**Highlights:**
- 20+ supported event sources
- Custom automation logic
- Ideal for CI/CD, workflow automation, and integrating with other Argo tools

Argo Events makes it easy to automate Kubernetes workflows based on real-world events.




