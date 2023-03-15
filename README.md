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

*	Create a new conda virtual environment in you working directory with the following command in your terminal:

* conda create --name deploy_ml python=3.6

With this command you will create a new conda based virtual environment on your local machine.

* Once your virtual environment is sucessfully create, you can easily to activate it with the following command:
conda activate deploy_ml
At this moment we are having an almost empty virtual environment which is ready to be filled with new dependencies.

### Step 2. Install dependencies in you virtual environment

*	Install mlflow package into our virtual environment with the following command:
pip install -q mlflow==1.18.0.

At the moment of preparing this repo, the version of mlfflow is mlflow==1.18.0.
*	To run properly the ML model itself we have to install following modules and packages to our virtual environment as well:
*	Pandas: pip install pandas.
*	Scikit-learn: pip install -U scikit-learn.
*	AWS Command Line Interface (AWS CLI): pip install awscli --upgrade --user.
*	Boto3: pip install boto3

### Step 3. Setup AWS IAM User and AWS CLI configuration

*	**Create a new AWS AIM User**
	
o	Open **Identity and Access management (IAM) dashboard.**

o	Click on __Users.__

o	Click __Add users__ on the right side of the screenshot.

o	Set the User name and mark Programmatic access tick below.

o	Click on **Create group** as the part of Add user to group option.

o	Type a group name you want to assign to your IAM User.

o	From the list below select following policies:

        *	__AmazonSageMakerFullAccess.__
        *	__AmazonEC2ContainerRegistryFullAccess.__
        *	__AmazonLambdaFunctionBasicExecutionPolicy__
        
o	Click on Create group, then the current Policies window will be closed.

o	Click on Next: Tags.

o	Click on Next: Review
.
o	Click on Create user.

o	You will get a notification about sucessfully created new User on AWS IAM, see the screenshot below.

<img src="https://user-images.githubusercontent.com/117730243/225217942-fe956e14-014c-4575-bc1e-bc37934af1f5.png">


**Important.** Keep safe you credentials on your own notes. This step is only one occasion you see AWS Secret Access Key. Rewrite it carefully.

**•	Setup AWS CLI configuration
o	Be sure you have installed AWS CLI and type command in your terminal: aws configure.
o	Then you will have to enter your own credentials as follows:
	AWS Access Key ID: go to IAM, then Users, and click on your user just created. Select Security credentials tab and copy the value of AWS Access Key ID
	AWS Secret Access Key: paste this code from your own notes. You have seen this code originally from Security credentials of your user.
	Default region name: go to main AWS interface, click on your region, and check which region is activated for you (us-east-1, eu-west-1, and so on).
	Default output format: set it as json.
o	The filled AWS CLI configurations should looks like in the screenshot below.
 
Step 4. Test if mlflow is working good
•	Before doing all following steps, we must be sure if our freshly installed mlflow service if working good on our local machine. To do it, type the following command in the terminal: mlflow ui.
•	Open the mlflow dashboard on you browser by entering following URL to your localhost: http://127.0.0.1:5000
Please keep in mind that this service uses port 5000 on your machine (open the second terminal window on the same working directory before run this command).
You should see mlflow dashboard interface it is shown in the screenshot below:
 
If you see the same on your browser too, mlflow works fine and we are ready to go to the next steps.
Prepare you Machine Learning model for mlflow
Step 1. Adapt your ML training code for mlflow
Usually we work with plain Python code in our local machine to train, debug and improve our ML models. In order to make out ML code be understandable for mlflow we must do quick changes in model training codes, described in the following steps.
•	Copy and paste the full code from train.py (available in this repo).
•	Adapt the Python code inside to track some model metrics in mlfflow with following changes in code:
import mlflow
import mlflow.sklearn
mlflow.set_experiment("my_classification_model")
with mlflow.start_run(run_name="My model experiment") as run:
    mlflow.log_param("num_estimators", num_estimators)
    mlflow.sklearn.log_model(rf, "random-forest-model")
    # Log model metrics (mse - mean squared error)
    mlflow.log_metric("mse", mse)
    # close mlflow connection
    run_id = run.info.run_uuid
    experiment_id = run.info.experiment_id
    mlflow.end_run()
    print(mlflow.get_artifact_uri())
    print("runID: %s" % run_id)
With the code changes above, we are ready to track mse (Mean Squared Error) metric for the model and generate some run artifacts in mlflow dashboard.
Step 2. Run the model and track metrics in mlflow
Once we finished the latest step in this tutorial, mlflow is able to track desired metrics (in our example, mse) in mlflow dashboard. To do it properly, do the following:
•	Open another terminal windown (be sure the first one with activated mlflow service in port :5000 is still running).
•	Activate the same virtual environment on the second terminal window as we have created in Step no. 1 with the following command: conda activate deploy_ml
•	Navigate your terminal to the project environment with cd (change directory) terminal command.
•	From the second terminal window, simply run the model training code train.py with the command in your terminal: python train.py.
•	Once the train script executed sucessfully, you will be notificated about creation of new experiment in mlflow, calculated MSE, absolute directory path to model artifacts, and runID, see the screenshot below.
 
•	Open the browser window again where mlflow service is running on. Refresh the browser window. Then, on the left menu Experiments you will see updated list of experiments ran so far, which includes that one you ran recently in this step (my_classification_model), see the screenshot below.
 
•	Click on the new experiment, and freely navigate through available options to check metric values and other parameters from the experiment.
•	Once you entered to your new experiment space, click on the most recent run tagged with Random Forest Experiment Run time tag. Be sure, that on the Artifacts section you are seeing this kind of structured experiment files with filled content for each file. See screenshot below.
 
Among the artifact files, there is a conda.yaml file. This is the environment your model needs to run, and it can be heavily customized based on your needs.
•	Also, check the file structure in you Finder (Mac OS), or in Windows explorer (Windows OS). You should see new folder mlruns created in your project directory with run numbers folder(0, 1, and so on), runID folder, and full set of artifacts inside. See the screenshot below.
 
When we are sure that our model can be sucessfully tracked with local mlflow user interface on your local machine, we can go forward to bring our model to the clouds.
