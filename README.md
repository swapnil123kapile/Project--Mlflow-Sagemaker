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

 * **AmazonSageMakerFullAccess**
    
 *  **AmazonEC2ContainerRegistryFullAccess**
    
 *  **AmazonLambdaFunctionBasicExecutionPolicy**
        
o	Click on Create group, then the current Policies window will be closed.

o	Click on Next: Tags.

o	Click on Next: Review
.
o	Click on Create user.

o	You will get a notification about sucessfully created new User on AWS IAM, see the screenshot below.

<img src="https://user-images.githubusercontent.com/117730243/225217942-fe956e14-014c-4575-bc1e-bc37934af1f5.png">


**Important.** Keep safe you credentials on your own notes. This step is only one occasion you see AWS Secret Access Key. Rewrite it carefully.

* **Setup AWS CLI configuration**

o	Be sure you have installed AWS CLI and type command in your terminal: aws configure.

o	Then you will have to enter your own credentials as follows:

	**AWS Access Key ID:** go to IAM, then Users, and click on your user just created. Select Security credentials tab and copy the value of AWS Access Key ID

	**AWS Secret Access Key:** paste this code from your own notes. You have seen this code originally from Security credentials of your user.

	**Default region name:** go to main AWS interface, click on your region, and check which region is activated for you (us-east-1, eu-west-1, and so on).

	**Default output format:** set it as json.

o	The filled AWS CLI configurations should looks like in the screenshot below.
<img src="https://user-images.githubusercontent.com/117730243/225221068-fbee2b8e-ffdc-4b3f-b0a8-c6d6e87aee8f.png">
	
### Step 4. Test if mlflow is working good

*	Before doing all following steps, we must be sure if our freshly installed mlflow service if working good on our local machine. To do it, type the following command in the terminal: mlflow ui.
	
•	Open the mlflow dashboard on you browser by entering following URL to your localhost: http://127.0.0.1:5000

Please keep in mind that this service uses port 5000 on your machine (open the second terminal window on the same working directory before run this command).
You should see mlflow dashboard interface it is shown in the screenshot below:
<img src="https://user-images.githubusercontent.com/117730243/225221593-3a9f00b8-4705-4c7e-a044-6237e90a8063.png"> 
If you see the same on your browser too, mlflow works fine and we are ready to go to the next steps.

### Prepare you Machine Learning model for mlflow

#### Step 1. Adapt your ML training code for mlflow

Usually we work with plain Python code in our local machine to train, debug and improve our ML models. In order to make out ML code be understandable for mlflow we must do quick changes in model training codes, described in the following steps.

*	Copy and paste the full code from train.py (available in this repo).

*	Adapt the Python code inside to track some model metrics in mlfflow with following changes in code:

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


### Step 2. Run the model and track metrics in mlflow
Once we finished the latest step in this tutorial, mlflow is able to track desired metrics (in our example, mse) in mlflow dashboard. To do it properly, do the following:


*	Open another terminal windown (be sure the first one with activated mlflow service in port :5000 is still running).
	
*	Activate the same virtual environment on the second terminal window as we have created in Step no. 1 with the following command: conda activate deploy_ml

*	Navigate your terminal to the project environment with cd (change directory) terminal command.
	
*      From the second terminal window, simply run the model training code train.py with the command in your terminal: python train.py.
      
*	Once the train script executed sucessfully, you will be notificated about creation of new experiment in mlflow, calculated MSE, absolute directory path to model artifacts, and runID, see the screenshot below.
	<img src="https://user-images.githubusercontent.com/117730243/225222680-9107e38c-e2e0-49ad-8da9-eb94b937c6fc.png>

•	Open the browser window again where mlflow service is running on. Refresh the browser window. Then, on the left menu Experiments you will see updated list of experiments ran so far, which includes that one you ran recently in this step (my_classification_model), see the screenshot below.
<html>	
<img src="https://user-images.githubusercontent.com/117730243/225222882-93fcd5d1-abc6-499c-8e9b-c8a89eca0989.png>

<a href="https://user-images.githubusercontent.com/117730243/225222882-93fcd5d1-abc6-499c-8e9b-c8a89eca0989.png> </a>
 </html>
*	Click on the new experiment, and freely navigate through available options to check metric values and other parameters from the experiment.
	
*	Once you entered to your new experiment space, click on the most recent run tagged with Random Forest Experiment Run time tag. Be sure, that on the Artifacts section you are seeing this kind of structured experiment files with filled content for each file. See screenshot below.
<img src="https://user-images.githubusercontent.com/117730243/225223148-fc975551-f802-47f7-ac30-e455695596ee.png">
Among the artifact files, there is a conda.yaml file. This is the environment your model needs to run, and it can be heavily customized based on your needs.

*	Also, check the file structure in you Finder (Mac OS), or in Windows explorer (Windows OS). You should see new folder mlruns created in your project directory with run numbers folder(0, 1, and so on), runID folder, and full set of artifacts inside. See the screenshot below.
<img src="https://user-images.githubusercontent.com/117730243/225223346-a053f87e-ff0d-4b69-852c-291fcf4625ef.png">
When we are sure that our model can be sucessfully tracked with local mlflow user interface on your local machine, we can go forward to bring our model to the clouds.

### Deploy the model to AWS

#### Step 1. Build a Docker Image and push it to AWS ECR

* On the second terminal windows, got to the artifact directory of selected model run in mlruns folder (in my example I set this directory as: /mlruns/1/ccc73fe5e7784df69b8518e3b9daa0c6/artifacts/random-forest-model), then type and run a command:
 
mlflow sagemaker build-and-push-container.

You will see all processes running in real time in your terminal, as shown in a screenshot below.
 <img src="https://user-images.githubusercontent.com/117730243/225224418-c7323ce9-1d5d-4da0-8eeb-03bf398757f6.png">

The finished processes on building and pushing Docker container images looks as it is in a screenshot below:
 <img src="https://user-images.githubusercontent.com/117730243/225224603-092c35ef-c9e8-49fd-8bf9-03435e843c5b.png">

If you have set up AWS correctly with the proper permissions, this will build an image locally and push it to your image (name is _mlflow-pyfunc_) registry on AWS.

* Once you finished the previous action without any issues, you should check local **Docker Desktop** to be sure that everything is well at this moment. Open your Docker Desktop application and go to Images section. You should see two images created. The first one indicates your **AWS ECR** container image, and the other one is mlflow-pyfunc. See the screenshot below.
<img src="https://user-images.githubusercontent.com/117730243/225225052-592f2fa2-aa1e-4906-90fd-f3b829941213.png">

*	Check AWS ECR repos list. You should see the same image name as it was listed first in Docker Desktop application. See the screenshot below.
 <img src="https://user-images.githubusercontent.com/117730243/225225289-51787717-81f5-4231-a4d9-1efb8b33122d.png">

 
*	Check image parameters in **AWS ECR** by clicking on mlflow-pyfunc image repository. On a image properties window you will see the main parameters of your image such as Image URI, Size, and other ones, see the screenshot below.

<img src="https://user-images.githubusercontent.com/117730243/225225548-570d12d3-6e0c-44a4-b1d9-a1f964170899.png">

 
If you see the same set of image parameters with meaningful values, move forward with next steps.

### Step 2. Deploy image to Sagemaker

Now all what we have to do is provide mlflow our image URL and desired model and then we can deploy these models to SageMaker.
**Important:** Before doing all following steps, you should create a special CLI token between your terminal and AWS. For this boto3 package is responsible, and you can do it in very simple way, just by setting your environment variables in your terminal like following:

* Type AWS_ACCESS_KEY=XXXXXXXXX, where XXXXXXXXX is your AWS Access Key for your User in AWS.
 
* Type AWS_SECRET_ACCESS_KEY=XXXXXXXXX, where XXXXXXXXX is your AWS Secret Access Key for your User in AWS.
 
Once you set your virtual environment, the terminal with active CLI is able to communicate with AWS services and make required operations.

*	Create a new Python script in your root project directory by terminal command touch deploy.py. This command will create a new file deploy.py.

*	Write the following script logic in the deploy.py you have just created:
	
*	import mlflow.sagemaker as mfs
	
*	experiment_id = '1'
	
*	run_id = 'XXXXXXXXXXXXXX'
	
*	region = 'us-east-2'
	
*	aws_id = 'XXXXXXXXXXXXXX'
	
*	arn = 'arn:aws:iam::XXXXXXXXXXXXXX:role/sagemaker-full-access-role'

*	app_name = 'model-application'

*	model_uri = 'mlruns/%s/%s/artifacts/random-forest-model' % (experiment_id, run_id)
	
*	tag_id = '1.18.0'
	
*	image_url = aws_id + '.dkr.ecr.' + region + '.amazonaws.com/mlflow-pyfunc:' + tag_id
	
*	mfs.deploy(app_name=app_name, 
	
*	           model_uri=model_uri, 
	           
*	           region_name=region,
	            
*	           mode="create",
	           
*	           execution_role_arn=arn,
           image_url=image_url)
	   
As you can see from the deploy.py skeleton code, we will need to get run_id, region, aws_id, ARN for AmazonSageMakerFullAccess (arn), model_uri, and image_url. Let's extract these values one by one.

o	experiment_id


	Open the Mlflow user interface in http://127.0.0.1:5000.

	Click on your experiment (for example **my_classification_model**).

	You will see an Experiment ID in the upper side of the screen next to Experiment ID text. Copy the value (in my case it is 1).

o	run_id:

	Open the Mlflow user interface in http://127.0.0.1:5000.

	Click on your experiment (for example **my_classification_model**).

	Click on run which has an image to an AWS ECR.

	On Artifacts section expand the list of artifacts by clicking on an arrow, and select MLModel.
	On the data window on the right, you can see main data about the model. One of these is run_id. Copy it.
o	aws_id:
Get your AWS ID from the terminal by running this command: aws sts get-caller-identity --query Account --output text.
o	arn:
Create the Role for the SageMakerFullAccess and grab it's ARN.
	Open IAM Dashboard.
	Click on Create role.
	Select SageMaker service from the given list and click Next: Permissions.
	Click Next: Tags.
	Once you completed to create this role, copy Role ARN for further usage (Role ARN).
o	region:
Go to main AWS interface, click on your region, and check which region is activated for you (us-east-1, eu-west-1, and so on).
o	app_name:
Set any name you want to recognize your application.
o	tag_id
This the version of mlflow. Open AWS ECR, then open your repository, and copy the value of Image tag.


•	Open again terminal with activated virtual environment deploy_ml and enter run the deploy.py Python script with the command: python deploy.py.
The output should be notify you about sucessfuly complete deploying the image to SageMaker, see the screen below.
 

You should see the inService message in you terminal resuting into sucessfully deployed model to SageMaker.
Use the model with the new data
Once you are sure that your model is running in AWS, you can make a new prediction for a new given data with the help of boto3 library. Follow the following steps to do it.
•	Open the terminal with activated virtual environment deploy_ml.
•	Create a new file predict.py with command touch predict.py.
•	For predict.py use this Python code:
•	import pandas as pd
•	from sklearn.model_selection import train_test_split
•	from sklearn import datasets
•	import json
•	import boto3
•	
•	global app_name
•	global region
•	app_name = 'model-application'
•	region = 'us-east-2'
•	
•	def check_status(app_name):
•	    sage_client = boto3.client('sagemaker', region_name=region)
•	    endpoint_description = sage_client.describe_endpoint(EndpointName=app_name)
•	    endpoint_status = endpoint_description["EndpointStatus"]
•	    return endpoint_status
•	
•	def query_endpoint(app_name, input_json):
•	    client = boto3.session.Session().client("sagemaker-runtime", region)
•	
•	    response = client.invoke_endpoint(
•	        EndpointName=app_name,
•	        Body=input_json,
•	        ContentType='application/json; format=pandas-split',
•	    )
•	    preds = response['Body'].read().decode("ascii")
•	    preds = json.loads(preds)
•	    print("Received response: {}".format(preds))
•	    return preds
•	
•	## check endpoint status
•	print("Application status is: {}".format(check_status(app_name)))
•	
•	# Prepare data to give for predictions
•	iris = datasets.load_iris()
•	x = iris.data[:, 2:]
•	y = iris.target
•	X_train, X_test, y_train, y_test = train_test_split(x, y, test_size=0.3, random_state=7)
•	
•	## create test data and make inference from enpoint
•	query_input = pd.DataFrame(X_train).iloc[[3]].to_json(orient="split")
prediction = query_endpoint(app_name=app_name, input_json=query_input)  
•	Set variables app_name and region based on your preferences.
•	Save the predict.py file.
•	Open again the terminal with activated virtual environment deploy_ml and run the file with command python predict.py.
•	As a result, you should see an inService message in terminal informing about sucessful new prediction made in the cloud, see the screenshot below.
 
Once you sucessfully did all the steps above and achieved the same results, in order to avoid extra billing, do not forget to remove SageMaker endpoint and AWS ECR repository with Docker image.
Youtube Tutorial is live now - all explained without missing parts
Full Hands-on tutorial of this repo is available now on Youtube!

