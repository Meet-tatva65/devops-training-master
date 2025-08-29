# DevOps Deployment Documentation

Here in this document I will cover each and every step I have taken from zero knowledge to successful deployment.

First of all, we have to understand the flow. For that, I have built a flow diagram to make it clear.

---

## 1. AWS CodePipeline

**Definition:**  
CodePipeline is a continuous integration and continuous delivery (CI/CD) service in AWS.  
It automates the build, test, and deployment phases every time there is a code change.

**Why it is useful:**
- Removes manual deployment steps  
- Provides automation and faster releases  

---

## 2. AWS CodeBuild

**Definition:**  
CodeBuild is a fully managed build service in AWS.  
It compiles source code, runs tests, and produces build artifacts (e.g., `.zip`, `.jar`).  
It uses a `buildspec.yml` file for instructions like `install → build → test → package`.

**Why it is useful:**
- No need to manage build servers manually  
- Scales automatically  
- Works seamlessly with CodePipeline  

---

## 3. IAM Roles

**Definition:**  
IAM (Identity and Access Management) manages permissions for AWS services.  
Services like CodePipeline, CodeBuild, and CodeDeploy require roles to interact with each other.

**Why it is useful:**
- Provides secure access  
- Ensures least privilege principle  

---

## Steps I Have Performed in AWS Project

### 1) Push Your Repo in GitHub
- Push your code into GitHub containing:
  - CloudFormation files  
  - Frontend  
  - Lambda (backend)  
  - `buildspec.yml` file  

### 2) Created an EC2 Instance
- Go to **AWS Console → EC2 → Key Pairs → Create Key**  
- Configure the key pair with the key name in EC2 instance **KeyName**

### 3) Created S3 Bucket
- Go to AWS Console → search for **S3**  
- Click **Create Bucket**  
- Give it a unique name (e.g., `my-devops-artifacts-bucket`)  
- Select region (same as where Lambda will run)  
- Leave other options default, then click **Create bucket**  

### 4) Create IAM Role for CodePipeline & CodeBuild
- Go to **IAM (Identity & Access Management)**  
- Click **Roles → Create Role**  
- Select AWS Service → **CodePipeline** (for pipeline role)  

**Attach the following managed policies:**
- `AmazonS3FullAccess` (to access artifacts in S3)  
- `AWSCodeBuildAdminAccess` (to trigger builds)  
- `AWSLambda_FullAccess` (to update Lambda function)  

- Name the role: **CodePipelineServiceRole** (or any as per your choice)  
- Repeat the process for **CodeBuild**, naming it **CodeBuildServiceRole** with similar S3 and Lambda access  

### 5) Create CodeBuild Project
- Go to **CodeBuild service → Create build project**  
- Name: `MyLambdaBuild` (or any appropriate name)  
- Source provider: **CodePipeline** (not S3)  
- Buildspec: Select "Use a `buildspec.yml` file"  
- Service Role: Choose **CodeBuildServiceRole**  
- Click **Create project**  

### 6) Create CodePipeline
- Go to **CodePipeline → Create pipeline**  
- Pipeline name: `CI/CDPipeline` (or any appropriate name)  
- Execution Mode: **Queued**  
- Service role: Select **CodePipelineServiceRole**  

**Add Source Stage:**
- Source provider → CodeCommit  
- Repository → `my-lambda-repo`  
- Branch → `main`  

**Add Build Stage:**
- Build provider → CodeBuild  
- Project name → `<Project name you created>`  

**Add Deploy Stage:**
- Deploy provider → AWS Lambda  
- Function name → `my-devops-lambda`  
- Input artifact → `lambda.zip` from CodeBuild  

- Click **Create pipeline**  

---

## 7) Test the Deployment
- Make a code change in `lambda/app.py`  
- Push the code to GitHub  
- Check **CodePipeline → Pipeline executed automatically:**  
  - Source → Build → Deploy  
- Verify in Lambda console → new code deployed successfully  

---

## Pipeline Flow Diagram
![Pipeline Diagram](https://i.postimg.cc/mkJvtVJr/workflow-img.jpg)

## HereWith i am attaching my documents
## Image One:-
![Pipeline Diagram](https://i.postimg.cc/NFsMdGjX/img-1.png)

## Image Two:-
![Pipeline Diagram](https://i.postimg.cc/NfPMxcqy/img-2.png)

## Image Three:-
![Pipeline Diagram](https://i.postimg.cc/pVsX2Hd0/img-3.png)

## Image Four:-
![Pipeline Diagram](https://i.postimg.cc/1z1yWRWz/img-4.png)

## Final Pipeline
![Pipeline Diagram](https://i.postimg.cc/fRJvHDV1/Pipeline.png)



# Issue Documentation

| Sr. No. | Issue                                   | Description                                                   | Root Cause                                     | Resolution                                                   | Status   |
|---------|-----------------------------------------|---------------------------------------------------------------|------------------------------------------------|--------------------------------------------------------------|----------|
| 1       | EC2 Instance failed                     | The key-pair is mismatched (`devops-test-keypair`) doesn’t exist. | Key pair not found in the selected region       | Created new key-pair in the region                           | Resolved |
| 2       | CloudFormation nested stack failed      | IAMRolesStack `CREATE_FAILED`                                 | Template URL used is of local path instead of S3 URL | Uploaded template to S3 and updated the `TemplateUrl` in `main.yml` | Resolved |
| 3       | EC2 Instance Creation Failed – Free Tier Error | The specified instance type is not eligible for Free Tier.    | `t2.micro` is free but `t3.micro` is preferable | Changed `t2.micro` to `t3.micro` in `main.yml` and committed the changes | Resolved |
