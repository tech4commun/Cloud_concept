# Developer Tooling for AWS Step Functions

> Structured notes from the "Developer Tooling for AWS Step Functions" course. Preview in VS Code with `Ctrl/Cmd+Shift+V`.

---

## 1. Course Overview

AWS Step Functions is a **low-code visual workflow service** to orchestrate AWS services, automate business processes, and build serverless applications.

By the end of this course you should be able to:
- Use different Step Functions development tools.
- Integrate Step Functions with other AWS services.
- Decide **when** and **how** to use each development option.

---

## 2. Developer Tools for AWS Step Functions

Step Functions maintains **application state** and an **event log** across steps, so workflows can resume after failures.

| Tool | Purpose |
|---|---|
| **Step Functions Console** | Cloud-based authoring; includes Workflow Studio + data flow simulator + visual workflow. |
| **AWS SDKs** | Java, .NET, Ruby, PHP, Python (Boto3), JavaScript, Go, C++ — programmatic access to Step Functions HTTPS API. |
| **AWS Toolkit for VS Code** | Create/update/list/run/download state machines locally; templates, snippets, validation, visualization. |
| **AWS CLI** | Create/list state machines; start & manage executions; poll activities; record task heartbeats. |
| **AWS SAM** | Serverless IaC — integrate workflows with Lambda, API Gateway, EventBridge. |
| **AWS CDK** | Define infrastructure in TypeScript/JavaScript/Python/Java/.NET/Go; synthesize to CloudFormation. |
| **Data Science SDK** | Open-source Python library to build ML workflows with SageMaker + Step Functions. |

All listed in the Step Functions console under **Local Development**.

---

## 3. AWS Serverless Application Model (SAM)

Open-source IaC framework for **serverless apps**.

- Shorthand syntax in **JSON or YAML**; transformed into CloudFormation at deploy.
- **AWS SAM CLI** — locally build, test, debug, deploy; can set up CI/CD pipelines.

### Advantages with Step Functions
- Start from sample templates in minutes.
- Embed the state machine inside your serverless app.
- Substitute resource ARNs via variables during deployment.
- Simplify IAM roles using **SAM policy templates**.
- Trigger executions from **API Gateway**, **EventBridge**, or **schedule** — all inside the SAM template.

---

## 4. AWS Cloud Development Kit (CDK)

Software development framework to define cloud infra in code and provision via CloudFormation.

- Supported languages: **TypeScript, JavaScript, Python, Java, .NET, Go**.
- The `cdk` CLI is the primary interaction tool (synthesize + deploy).

### Advantages with Step Functions
- Define resources in a familiar language.
- Use logic, OOP, and modules.
- Build reusable **high-level abstractions**; publish as libraries.
- Test infra code with standard tooling.
- Fit into existing IDE + code review workflows.

### Sample — TypeScript
```ts
const stateMachine = new sfn.StateMachine(this, 'MyStateMachine', {
  definition: new tasks.LambdaInvoke(this, "MyLambdaTask", {
    lambdaFunction: helloFunction
  }).next(new sfn.Succeed(this, "GreetedWorld"))
});
```

### Sample — Java
```java
StateMachine stateMachine = StateMachine.Builder.create(this, "MyStateMachine")
    .definition(LambdaInvoke.Builder.create(this, "MyLambdaTask")
        .lambdaFunction(helloFunction)
        .build()
        .next(new Succeed(this, "GreetedWorld")))
    .build();
```

---

## 5. Knowledge Check — SAM vs CDK

| # | Use case | Best choice | Reason |
|---|---|---|---|
| 1 | Serverless app, resources defined in JSON/YAML | **SAM** | SAM is serverless-focused, declarative JSON/YAML templates. |
| 2 | Serverless app, resources defined in a programming language | **CDK** | CDK defines infra in real languages (TS/Python/Java/C#). |
| 3 | Standardize with high-level abstractions shared across teams | **CDK** | CDK supports reusable, publishable abstractions; SAM does not. |

---

## 6. Workflow Studio

Low-code **visual designer** in the Step Functions console.

- Drag-and-drop AWS service states.
- Configure input/output filtering & transformation per state.
- Configure error handling (**retriers** on action + Parallel states).
- Auto-generates & validates **Amazon States Language (ASL)** as you build.
- Export code for local dev or CloudFormation.

### Creating a workflow
1. Sign in to the Step Functions console.
2. Choose **Design your workflow visually**.
3. Pick type **Standard**.
4. Design on canvas → configure via Form panel.
5. Review generated ASL on **Review generated code**.
6. Under Permissions → **Create a new IAM role**.
7. **Create state machine**.

### Known limitations
- No dynamic resource IDs in a Task's `Resource` — use dynamic ARN in `Parameters` instead.
- **Wait for callback** does not auto-add a task token — add manually in General code snippet.
- No support for Internet Explorer 11.

---

## 7. AWS Toolkit for Visual Studio Code

Open-source extension for local dev/debug/deploy of serverless apps.

### Prerequisites
- AWS account.
- Linux / Windows / macOS.
- **VS Code ≥ 1.42**.
- SDKs for target languages (.NET, Node.js, Python, Java, Go).
- **AWS SAM CLI** for local serverless dev.

### Setup
1. Install **AWS Toolkit** from VS Code Extensions.
2. Configure AWS credentials.

### Step Functions features in the toolkit
- Real-time **state machine visualization**.
- **Code snippets** for each state.
- **ASL language server** validates definitions and flags common errors.
- Create/update/run state machines against your AWS account.

### Creating a state machine from template
- `View → Command Palette → AWS: Create a new Step Functions state machine`.

---

## 8. Connecting VS Code to AWS

Three supported methods:

### AWS Access Keys & Credentials
- Retrieve AWS access keys from the console.
- Add to your environment / credentials file.

### AWS SSO (IAM Identity Center)
- Central SSO across AWS accounts & apps.
- Define a named profile pointing to the SSO portal + AWS account + IAM role.

### External Credentials
- For credentials not natively supported.
- Add a `credential_process` entry in the shared AWS config file (same as AWS CLI).

### AWS Explorer binding
- Bound to a **single profile + Region** at a time.
- Switch profile/Region inside AWS Explorer.
- Multiple VS Code windows can each bind to a different profile/Region.
- Remembers the **last used** profile & Region.

**Knowledge check answer:** Valid connection methods = **AWS SSO** and **AWS access keys and credentials**.

---

## 9. Validating the State Definition — Statelint

**Statelint** = AWS-provided Ruby gem that validates Amazon States Language JSON.

### Install
```bash
sudo install statelint
```

### Detects issues such as
- Missing `Type` in a **Parallel** state.
- Missing `Next` in a **Choice** state.
- Unreachable states.
- Missing terminal states.

On success, Statelint prints nothing.

---

## 10. Demo — Writing Step Functions in VS Code

Sample project: **serverless-account-signup-service** (verifies identity + address for auto-insurance applicants).

```bash
git clone https://github.com/aws-samples/serverless-account-signup-service
```

Structure:
- `functions/` — two Lambda functions.
- `statemachine/application_service.asl.json` — ASL definition.

### Workflow logic
1. **StartAt: Verification** — Parallel state runs **Check Identity** + **Check Address** Lambdas.
2. **Choice** state on outputs:
   - If **either denied** → publish denial via **SNS** → end.
   - If **both approved** →
     - **Add Account** → write applicant record to **DynamoDB**.
     - **Home Insurance Interests** → post message to **SQS**.
     - **Approved Message** → send approval email via **SNS**.

Visualize: `View → Command Palette → "Render state machine graph"`.

---

## 11. AWS Step Functions Data Science SDK

Open-source Python SDK to build ML workflows with **SageMaker** + Step Functions.

Use cases include **scheduled model retraining and deployment**. `workflow.render_graph()` renders the workflow like the console does.

### Parameterized execution input
```python
execution_input = ExecutionInput(schema={
    'TrainingJobName': str,
    'GlueJobName': str,
    'ModelName': str,
    'EndpointName': str,
    'LambdaFunctionName': str
})
```

### Glue ETL step
```python
etl_step = steps.GlueStartJobRunStep(
    'Extract, Transform, Load',
    parameters={"JobName": execution_input['GlueJobName']}
)
```

### Training step
```python
training_step = steps.TrainingStep(
    'Model Training',
    estimator=xgb,
    data={
      'train': sagemaker.s3_input(train_data, content_type='csv'),
      'validation': sagemaker.s3_input(validation_data, content_type='csv')},
    job_name=execution_input['TrainingJobName']
)
```

### Query training results via Lambda
```python
lambda_step = steps.compute.LambdaStep(
    'Query Training Results',
    parameters={"FunctionName": execution_input['LambdaFunctionName'],
        'Payload':{"TrainingJobName.$": "$.TrainingJobName"}
    }
)
```

### Model + endpoint config + endpoint
```python
model_step = steps.ModelStep(
    'Save Model',
    model=training_step.get_expected_model(),
    model_name=execution_input['ModelName'],
    result_path='$.ModelStepResults'
)

endpoint_config_step = steps.EndpointConfigStep(
    "Create Model Endpoint Config",
    endpoint_config_name=execution_input['ModelName'],
    model_name=execution_input['ModelName'],
    initial_instance_count=1,
    instance_type='ml.m4.xlarge'
)

endpoint_step = steps.EndpointStep(
    'Update Model Endpoint',
    endpoint_name=execution_input['EndpointName'],
    endpoint_config_name=execution_input['ModelName'],
    update=True   # update existing endpoint instead of creating new
)
```

### Branching + linking
```python
check_accuracy_step = steps.states.Choice('Accuracy > 90%')

# link individually
endpoint_config_step.next(endpoint_step)

# or chain
workflow_definition = steps.Chain([
    etl_step,
    training_step,
    model_step,
    lambda_step,
    check_accuracy_step
])
```

### Create the workflow
```python
workflow = Workflow(
    name='MyInferenceRoutine_{}'.format(id),
    definition=workflow_definition,
    role=workflow_execution_role,
    execution_input=execution_input
)
workflow.create()
```

---

## 12. Quick Cheat Sheet

| Need | Reach for |
|---|---|
| Declarative serverless template (JSON/YAML) | **AWS SAM** |
| Infra in a real programming language | **AWS CDK** |
| Visual drag-and-drop authoring | **Workflow Studio** |
| Local dev + visualization + validation | **AWS Toolkit for VS Code** |
| Validate ASL from CLI | **Statelint** |
| ML workflows with SageMaker | **Data Science SDK** |
| Scripted automation | **AWS CLI** / **SDKs** |
