# Setting Up GitHub Actions for Deploying a Docker Python Project to AWS Elastic Beanstalk

This markdown file will guide you through the process of setting up GitHub Actions to automate the deployment of a Docker Python project to AWS Elastic Beanstalk. The deployment will be done by zipping the project and sending it to Elastic Beanstalk for deployment.

## Prerequisites

Before you begin, make sure you have the following:

- A GitHub repository containing your Docker Python project.
- An AWS account with access to Elastic Beanstalk.

## Step 1: Configure AWS Elastic Beanstalk

1. Log in to your AWS account and navigate to the Elastic Beanstalk service.
2. Create an Elastic Beanstalk application and environment for your project.
3. Ensure that the environment has the necessary resources (such as EC2 instances, load balancers, etc.) to run your Docker Python project.

## Step 2: Set Up AWS Credentials

To enable GitHub Actions to interact with your AWS resources, you need to set up AWS credentials.

1. In the AWS Management Console, navigate to the IAM service.
2. Create a new IAM user or use an existing one (user named 'Github_Actions' is already created for this).
3. Attach the necessary permissions to the IAM user. At a minimum, the user should have permissions to deploy to Elastic Beanstalk.
4. Generate an access key for the IAM user and make note of the access key ID and secret access key.

## Step 3: Configure GitHub Secrets

To securely store your AWS credentials in GitHub, you'll use GitHub Secrets.

1. In your GitHub repository, go to "Settings" and click on "Secrets" in the left sidebar.
2. Click on "New repository secret" to create a new secret.
3. Name the secret AWS_ACCESS_KEY_ID and enter the corresponding value.
4. Click on "Add secret" to save the AWS access key ID.
5. Repeat the previous steps to create another secret named AWS_SECRET_ACCESS_KEY with the corresponding value.

## Step 4: Create the GitHub Actions Workflow

1. In your GitHub repository, navigate to the .github/workflows directory.
2. Create a new file with a name like deploy.yml to define the deployment workflow.
3. Open the file in your preferred text editor and add the following code:

```yaml
name: Docker Image CI

on:
  push:
    branches: [ "main" ]

jobs:

  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Build the Docker image
      run: docker build . --file Dockerfile --tag optimus-micro:$(date +%s)
    - name: Generate Deployment Package
      run: zip -r deploy.zip .* -x "../*"
    - name: Get timestamp
      uses: gerred/actions/current-time@master
      id: current-time 
    - name: Run string replace
      uses: frabert/replace-string-action@master
      id: format-time
      with:
        pattern: '[:\.]+'
        string: "${{ steps.current-time.outputs.time }}"
        replace-with: '-'
        flags: 'g'
        
    - name: Deploy to EB
      uses: einaregilsson/beanstalk-deploy@v14
      with:
        aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        application_name: optimus-ds
        environment_name: Optimus-ds-env
        version_label: "optimus-ds-${{ steps.format-time.outputs.replaced }}"
        region: ap-south-1
        deployment_package: deploy.zip
   ```

4. Customize the workflow by replacing your-application-name, your-environment-name, and your-aws-region with your actual values.

## Step 5: Commit and Push the Workflow

1. Save the changes to the deploy.yml file.
2. Commit the file to your repository and push it to the main branch.

## Step 6: Test the Deployment

1. Make a change to your Docker Python project and push the changes to the main branch.
2. Go to the "Actions" tab in your GitHub repository to monitor the workflow execution.
3. The workflow will trigger, and you can view the progress and logs in the GitHub Actions interface.
4. Once the workflow completes successfully, check your AWS Elastic Beanstalk environment to verify that the new version of your Docker Python project has been deployed.

After completing all the steps, your application should be deployed on Elastic beanstalk via Github Actions.
