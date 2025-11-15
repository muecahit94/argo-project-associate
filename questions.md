# **Certified Argo Project Associate (CAPA) Practice Exam**

This practice exam covers the key areas of the CAPA curriculum, with questions weighted according to the topic distribution:
- Argo Workflows (36%)
- Argo CD (34%)
- Argo Rollouts (18%)
- Argo Events (12%)

---

## **Section 1: Argo Workflows (14 Questions)**

1. **Which of the following best describes an Argo Workflow?**
   - [ ] **A)** A continuous delivery tool for Kubernetes
   - [ ] **B)** A container-native workflow engine for orchestrating parallel jobs
   - [ ] **C)** A progressive delivery controller
   - [ ] **D)** An event-driven automation framework

2. **In an Argo Workflow template, what is the purpose of the `outputs` section?**
   - [ ] **A)** To define the workflow's exit conditions
   - [ ] **B)** To specify the artifacts produced by the template
   - [ ] **C)** To configure output logging
   - [ ] **D)** To set environment variables

3. **Which of the following YAML defines a valid DAG task dependency?**
   ```yaml
   dag:
     tasks:
     - name: A
       template: process
     - name: B
       dependencies: [A]
       template: analyze
   ```
   - [ ] **A)** The syntax is incorrect
   - [ ] **B)** The syntax is correct and B will run after A
   - [ ] **C)** The syntax is correct but missing required fields
   - [ ] **D)** Dependencies should use "depends" instead of "dependencies"

4. **How do you specify that a workflow step should only execute if a previous step failed?**
   - [ ] **A)** Using `when: "{{workflow.status}} == Failed"`
   - [ ] **B)** Using `when: "{{steps.STEPNAME.status}} == Failed"`
   - [ ] **C)** Using `if: failed`
   - [ ] **D)** Using `condition: failed`

5. **Which Argo Workflows feature allows you to share common workflow steps across multiple workflows?**
   - [ ] **A)** Templates
   - [ ] **B)** WorkflowTemplates
   - [ ] **C)** SharedSteps
   - [ ] **D)** CommonWorkflows

6. **What is the correct way to reference an input parameter in an Argo Workflow template?**
   - [ ] **A)** `$(inputs.parameters.PARAM_NAME)`
   - [ ] **B)** `{{inputs.parameters.PARAM_NAME}}`
   - [ ] **C)** `${inputs.parameters.PARAM_NAME}`
   - [ ] **D)** `%inputs.parameters.PARAM_NAME%`

7. **Which of the following correctly defines an artifact input in a workflow template?**
   ```yaml
   inputs:
     artifacts:
     - name: data
       path: /tmp/data
       s3:
         bucket: my-bucket
         key: data.csv
   ```
   - [ ] **A)** The configuration is correct
   - [ ] **B)** The path should be relative
   - [ ] **C)** The s3 configuration needs credentials
   - [ ] **D)** The name field is unnecessary

8. **How do you specify a workflow should retry failed steps?**
   - [ ] **A)** Using `retryStrategy` in the workflow spec
   - [ ] **B)** Using `retry` in each step
   - [ ] **C)** Using `onFailure: retry`
   - [ ] **D)** Using `failurePolicy: Retry`

9. **Which command shows the logs of a specific workflow step?**
   - [ ] **A)** `argo logs <workflow-name>`
   - [ ] **B)** `argo logs <workflow-name> -f`
   - [ ] **C)** `argo logs <workflow-name> <pod-name>`
   - [ ] **D)** `argo logs <workflow-name> --step=<step-name>`

10. **What is the purpose of the `archiveLogs` field in a workflow spec?**
    - [ ] **A)** To save logs to a persistent volume
    - [ ] **B)** To enable log aggregation
    - [ ] **C)** To save logs as an artifact
    - [ ] **D)** To compress log files

11. **Which of the following artifact types is NOT supported by Argo Workflows?**
    - [ ] **A)** S3
    - [ ] **B)** Git
    - [ ] **C)** Artifactory
    - [ ] **D)** MongoDB

12. **What is the correct way to specify a workflow should run with a specific service account?**
    - [ ] **A)** Using `spec.serviceAccount`
    - [ ] **B)** Using `spec.serviceAccountName`
    - [ ] **C)** Using `metadata.serviceAccount`
    - [ ] **D)** Using `metadata.serviceAccountName`

13. **Which feature allows you to suspend a workflow at a specific step?**
    - [ ] **A)** Using `suspend: true` in the step
    - [ ] **B)** Using a `suspend` template
    - [ ] **C)** Using `wait: true`
    - [ ] **D)** Using `pause: true`

14. **How do you specify a global timeout for an entire workflow?**
    - [ ] **A)** Using `spec.timeout`
    - [ ] **B)** Using `metadata.timeout`
    - [ ] **C)** Using `spec.activeDeadlineSeconds`
    - [ ] **D)** Using `spec.deadline`

---

## **Section 2: Argo CD (14 Questions)**

15. **Which component in Argo CD is responsible for maintaining the Git repository cache?**
    - [ ] **A)** API Server
    - [ ] **B)** Repository Server
    - [ ] **C)** Application Controller
    - [ ] **D)** Dex Server

16. **What is the correct way to configure automated sync in an Argo CD Application?**
    ```yaml
    spec:
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
    ```
    - [ ] **A)** The configuration is correct
    - [ ] **B)** `automated` should be `auto`
    - [ ] **C)** `selfHeal` should be `autoHeal`
    - [ ] **D)** `prune` should be under `spec`

17. **Which command shows the sync status of all applications?**
    - [ ] **A)** `argocd app list`
    - [ ] **B)** `argocd status`
    - [ ] **C)** `argocd get applications`
    - [ ] **D)** `argocd app status`

18. **What is the purpose of the `argocd-cm` ConfigMap?**
    - [ ] **A)** To store user credentials
    - [ ] **B)** To configure Argo CD settings
    - [ ] **C)** To store application definitions
    - [ ] **D)** To manage RBAC policies

19. **Which Argo CD feature allows you to manage applications across multiple clusters?**
    - [ ] **A)** Cluster management
    - [ ] **B)** Multi-cluster sync
    - [ ] **C)** Cluster registry
    - [ ] **D)** Application Sets

20. **What is the correct way to ignore differences in a specific field using ignoreDifferences?**
    ```yaml
    spec:
      ignoreDifferences:
        - group: apps
          kind: Deployment
          jsonPointers:
            - /spec/replicas
    ```
    - [ ] **A)** The configuration is correct
    - [ ] **B)** `jsonPointers` should be `paths`
    - [ ] **C)** `group` should be `apiGroup`
    - [ ] **D)** The syntax is completely wrong

21. **Which of the following is NOT a valid sync option in Argo CD?**
    - [ ] **A)** `CreateNamespace`
    - [ ] **B)** `PruneLast`
    - [ ] **C)** `ForceSync`
    - [ ] **D)** `AutoHeal`

22. **How do you enable SSO with GitHub in Argo CD?**
    - [ ] **A)** Configure GitHub connector in Dex
    - [ ] **B)** Use GitHub OAuth directly
    - [ ] **C)** Enable GitHub integration
    - [ ] **D)** Configure GitHub OIDC

23. **What is the purpose of the Application Health status in Argo CD?**
    - [ ] **A)** To show Git repository status
    - [ ] **B)** To indicate sync status
    - [ ] **C)** To show resource operational status
    - [ ] **D)** To display cluster health

24. **Which command syncs an application and skips hooks?**
    - [ ] **A)** `argocd app sync --no-hooks`
    - [ ] **B)** `argocd app sync --skip-hooks`
    - [ ] **C)** `argocd app sync --force`
    - [ ] **D)** `argocd app sync --ignore-hooks`

25. **What is the default port for the Argo CD API server?**
    - [ ] **A)** 8080
    - [ ] **B)** 443
    - [ ] **C)** 8443
    - [ ] **D)** 80

26. **How do you configure Helm value overrides in an Argo CD Application?**
    - [ ] **A)** Using `spec.source.helm.values`
    - [ ] **B)** Using `spec.helm.parameters`
    - [ ] **C)** Using `spec.source.helm.parameters`
    - [ ] **D)** Using `spec.helmValues`

27. **Which feature allows Argo CD to automatically detect and deploy applications?**
    - [ ] **A)** Auto-discovery
    - [ ] **B)** App of Apps pattern
    - [ ] **C)** ApplicationSet
    - [ ] **D)** Auto-sync

28. **What is the purpose of the 'argocd-server' service account?**
    - [ ] **A)** To run the API server
    - [ ] **B)** To manage cluster access
    - [ ] **C)** To execute sync operations
    - [ ] **D)** To handle authentication

---

## **Section 3: Argo Rollouts (7 Questions)**

29. **Which feature in Argo Rollouts allows gradual traffic shifting to a new version?**
    - [ ] **A)** Progressive delivery
    - [ ] **B)** Canary deployment
    - [ ] **C)** Blue-Green deployment
    - [ ] **D)** Rolling update

30. **What is the correct way to specify a blue-green deployment strategy?**
    ```yaml
    spec:
      strategy:
        blueGreen:
          activeService: active-svc
          previewService: preview-svc
    ```
    - [ ] **A)** The configuration is correct
    - [ ] **B)** `activeService` should be `currentService`
    - [ ] **C)** Strategy should be under `metadata`
    - [ ] **D)** `blueGreen` should be `blue-green`

31. **Which Argo Rollouts feature allows automated promotion based on metrics?**
    - [ ] **A)** Metric analysis
    - [ ] **B)** Analysis templates
    - [ ] **C)** Automated analysis
    - [ ] **D)** Promotion rules

32. **What is the purpose of the 'autoPromotionEnabled' field in a blue-green deployment?**
    - [ ] **A)** To automatically scale the deployment
    - [ ] **B)** To automatically promote the preview service to active
    - [ ] **C)** To enable automatic rollbacks
    - [ ] **D)** To enable progressive delivery

33. **Which command shows the status of a rollout?**
    - [ ] **A)** `kubectl argo rollouts get rollout`
    - [ ] **B)** `kubectl-argo-rollouts get rollout`
    - [ ] **C)** `argo rollouts status`
    - [ ] **D)** `kubectl get rollout`

34. **What is the purpose of an AnalysisTemplate in Argo Rollouts?**
    - [ ] **A)** To analyze deployment logs
    - [ ] **B)** To define metric queries and thresholds
    - [ ] **C)** To analyze resource usage
    - [ ] **D)** To monitor service health

35. **Which field specifies the maximum time for an analysis run?**
    - [ ] **A)** `maxDuration`
    - [ ] **B)** `timeout`
    - [ ] **C)** `analysisTimeout`
    - [ ] **D)** `duration`

---

## **Section 4: Argo Events (5 Questions)**

36. **What is the primary purpose of Argo Events?**
    - [ ] **A)** To manage Kubernetes events
    - [ ] **B)** To provide event-driven automation
    - [ ] **C)** To monitor cluster events
    - [ ] **D)** To log application events

37. **Which component in Argo Events receives events from event sources?**
    - [ ] **A)** Event Bus
    - [ ] **B)** Sensor
    - [ ] **C)** Gateway
    - [ ] **D)** Controller

38. **What is the correct way to define a webhook event source?**
    ```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: EventSource
    metadata:
      name: webhook
    spec:
      webhook:
        example:
          port: "12000"
          endpoint: /webhook
    ```
    - [ ] **A)** The configuration is correct
    - [ ] **B)** `webhook` should be `webhooks`
    - [ ] **C)** Port should be an integer
    - [ ] **D)** Endpoint must start with /api

39. **Which of the following is NOT a supported event source in Argo Events?**
    - [ ] **A)** Webhook
    - [ ] **B)** Calendar
    - [ ] **C)** Kafka
    - [ ] **D)** GraphQL

40. **What is the purpose of a Sensor in Argo Events?**
    - [ ] **A)** To generate events
    - [ ] **B)** To process and trigger actions based on events
    - [ ] **C)** To store event data
    - [ ] **D)** To monitor event sources

---

## Answer Key and Explanations

1. **B) A container-native workflow engine for orchestrating parallel jobs**  
   *Explanation:* Argo Workflows is specifically designed as a container-native workflow engine that orchestrates parallel jobs on Kubernetes.

2. **B) To specify the artifacts produced by the template**  
   *Explanation:* The outputs section in a template defines what artifacts the template produces, which can then be used by subsequent steps in the workflow.

3. **B) The syntax is correct and B will run after A**  
   *Explanation:* The DAG syntax correctly shows task B depending on task A using the dependencies field.

4. **B) Using `when: "{{steps.STEPNAME.status}} == Failed"`**  
   *Explanation:* This is the correct syntax to conditionally execute a step based on a previous step's failure status.

5. **B) WorkflowTemplates**  
   *Explanation:* WorkflowTemplates are reusable workflow definitions that can be referenced across multiple workflows.

6. **B) {{inputs.parameters.PARAM_NAME}}**  
   *Explanation:* Input parameters in Argo Workflows are referenced using double curly braces syntax.

7. **A) The configuration is correct**  
   *Explanation:* The YAML correctly defines an S3 artifact input with all required fields including name, path, and S3 configuration.

8. **A) Using retryStrategy in the workflow spec**  
   *Explanation:* retryStrategy is the correct field to configure retry behavior for failed steps in a workflow.

9. **D) argo logs <workflow-name> --step=<step-name>**  
   *Explanation:* The --step flag is used to view logs for a specific step in an Argo Workflow.

10. **C) To save logs as an artifact**  
    *Explanation:* The archiveLogs field enables saving workflow logs as artifacts that can be accessed after workflow completion.

11. **D) MongoDB**  
    *Explanation:* MongoDB is not a supported artifact repository type in Argo Workflows. Supported types include S3, Git, and Artifactory.

12. **B) Using spec.serviceAccountName**  
    *Explanation:* The serviceAccountName field in the workflow spec is used to specify which service account the workflow should run as.

13. **B) Using a suspend template**  
    *Explanation:* A suspend template is used to pause workflow execution at a specific point until manual intervention.

14. **C) Using spec.activeDeadlineSeconds**  
    *Explanation:* activeDeadlineSeconds in the workflow spec defines the maximum duration for the entire workflow.

15. **B) Repository Server**  
    *Explanation:* The Repository Server component in Argo CD is responsible for maintaining the Git repository cache and generating Kubernetes manifests.

16. **A) The configuration is correct**  
    *Explanation:* The YAML correctly configures automated sync with pruning and self-healing enabled.

17. **A) argocd app list**  
    *Explanation:* The argocd app list command shows the sync status of all applications.

18. **B) To configure Argo CD settings**  
    *Explanation:* The argocd-cm ConfigMap stores various Argo CD configuration settings including SSO, repositories, and general settings.

19. **D) Application Sets**  
    *Explanation:* ApplicationSet allows managing applications across multiple clusters through templating and generators.

20. **A) The configuration is correct**  
    *Explanation:* The ignoreDifferences configuration correctly specifies how to ignore differences in the replicas field of Deployments.

21. **D) AutoHeal**  
    *Explanation:* AutoHeal is not a valid sync option. Valid options include CreateNamespace, PruneLast, and others.

22. **A) Configure GitHub connector in Dex**  
    *Explanation:* SSO with GitHub is configured through the Dex identity service in Argo CD.

23. **C) To show resource operational status**  
    *Explanation:* The Health status indicates the operational status of deployed resources, distinct from sync status.

24. **A) argocd app sync --no-hooks**  
    *Explanation:* The --no-hooks flag skips running hooks during application sync.

25. **C) 8443**  
    *Explanation:* The default HTTPS port for the Argo CD API server is 8443.

26. **A) Using spec.source.helm.values**  
    *Explanation:* Helm value overrides are configured under spec.source.helm.values in the Application spec.

27. **C) ApplicationSet**  
    *Explanation:* ApplicationSet provides automation for discovering and generating Argo CD Applications.

28. **A) To run the API server**  
    *Explanation:* The argocd-server service account is used to run the Argo CD API server component.

29. **B) Canary deployment**  
    *Explanation:* Canary deployment in Argo Rollouts enables gradual traffic shifting to a new version.

30. **A) The configuration is correct**  
    *Explanation:* The YAML correctly configures a blue-green deployment strategy with active and preview services.

31. **B) Analysis templates**  
    *Explanation:* Analysis templates define metric queries and success conditions for automated promotion.

32. **B) To automatically promote the preview service to active**  
    *Explanation:* autoPromotionEnabled determines whether the preview service is automatically promoted to active.

33. **B) kubectl-argo-rollouts get rollout**  
    *Explanation:* This is the correct command to view rollout status using the kubectl plugin.

34. **B) To define metric queries and thresholds**  
    *Explanation:* AnalysisTemplates define the metrics to query and success/failure conditions for automated analysis.

35. **A) maxDuration**  
    *Explanation:* The maxDuration field specifies the maximum time an analysis run can take.

36. **B) To provide event-driven automation**  
    *Explanation:* Argo Events is designed to provide event-driven automation in Kubernetes environments.

37. **B) Sensor**  
    *Explanation:* Sensors in Argo Events receive events and trigger defined actions.

38. **A) The configuration is correct**  
    *Explanation:* The YAML correctly defines a webhook event source with port and endpoint specifications.

39. **D) GraphQL**  
    *Explanation:* GraphQL is not a supported event source type in Argo Events.

40. **B) To process and trigger actions based on events**  
    *Explanation:* Sensors process incoming events and trigger defined actions based on event data.