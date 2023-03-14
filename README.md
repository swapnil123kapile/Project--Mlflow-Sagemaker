# Project-13

## Deploying Models to Production with MLflow and Amazon Sagemaker


This repo introduces how to prepare you Machine Learning (ML) model for production in AWS Sagemaker with the help of MLflow and AWS Command Line Inteface (AWS CLI). Before start, we should know what is what:

*	__Mlflow__ - an open source platform for the ML lifecycle.
*	__AWS CLI__ - is an unified tool to manage your Amazon Web Services (AWS) services.

With this repo you can follow the provided steps on your local machine and AWS account. All steps will be explained clearly with screens and real example.

### Setup the environment

For this tutorial you will need:

*	AWS Account.
*	Docker installed on your local machine.
*	Python **3.6** (or higher) with mlflow>=1.0.0 installed.
*	Anaconda software to create conda virtual environment.

Next, I will demonstrate how to install mlflow to your dedicated virtual environment. For this I will use Mac OS, but you can do the same steps on both Windows and Mac OS.

## Step 1. Prepare you Python Virtual environment
With this step we will create dedicated virtual environment to perform the whole example in this repo. Do the following steps.
•	Create a new conda virtual environment in you working directory with the following command in your terminal:
conda create --name deploy_ml python=3.6
With this command you will create a new conda based virtual environment on your local machine.
•	Once your virtual environment is sucessfully create, you can easily to activate it with the following command:
conda activate deploy_ml
At this moment we are having an almost empty virtual environment which is ready to be filled with new dependencies.
Step 2. Install dependencies in you virtual environment
•	Install mlflow package into our virtual environment with the following command:
pip install -q mlflow==1.18.0.
At the moment of preparing this repo, the version of mlfflow is mlflow==1.18.0.
•	To run properly the ML model itself we have to install following modules and packages to our virtual environment as well:
o	Pandas: pip install pandas.
o	Scikit-learn: pip install -U scikit-learn.
o	AWS Command Line Interface (AWS CLI): pip install awscli --upgrade --user.
o	Boto3: pip install boto3

