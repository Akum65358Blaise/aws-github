#Deploy React Application to AWS Using Github Actions

A lab on how to deploy a React Application to AWS using Github Actions.

Create React Project
First let’s create react project locally. We are going to use CRA for this. Use below command to create a react project.

npx create-react-app aws-github

##Create GitHub Project

Now that we have our project already created locally, it is time to create a new project on our GitHub. Login to your GitHub account and create a new project.

Click on New repository

At Repository name give name aws-github and scroll down all the way to bottom and click on Create repository button.

As we already created a new project locally. Simply grab below command

git remote add origin git@github.com:Akum65358Blaise/aws-github.git

and follow below steps.


> cd aws-github
> rm -rf .git
> git init
> git remote add origin git@github.com:zafar-saleem/aws-github.git

In the first line, we are cd into the root folder of our project locally. Then we get rid of the .git folder and initialise the parent folder as a new and fresh repository. Finally, the command we grabbed in previous step paste it here and run it which will add our GitHub repo as origin to our local repository.

##Get AWS Keys

Time for get credentials from our AWS console. Login to your AWS console and click on your username at the top right corner of the screen as shown below. Then click on Security Credentials.

##Adding Credentials to GitHub Repository

Since we already have the credentials from AWS let’s finish this step off by adding them to our GitHub repository. Go to security tab on your GitHub aws-github repository.

Then click on Secrets and then Actions. Then click on New Repository Secret.

Add your Access Key which you downloaded from AWS in previous step here.

Click on New Repository secret again and this time add Secret Access Key.

Finally add key for production environment by following the same procedure i.e. clicking on New Repository secret.

We have to create above aws-github-prod S3 bucket next in our AWS console.

##Creating and Setting Up AWS S3 Bucket

Now that we have GitHub setup, we need to have an S3 bucket where we will deploy our react application. The name of this bucket must match the Secret we added to GitHub above which is aws-github-prod.

Open your AWS console and click on S3.

Click on Create bucket.

Once you land on below page enter the name of the bucket as aws-github-prod.

Scroll down and uncheck Block all public access and check your acknowledgement.

Now scroll all the way down and click on Create bucket button.

You should be able to view the newly created bucket now.

##Create Bucket Policy

Click on your bucket name from the list of S3 home page i.e. aws-github-prod. On the next page click on permissions.

Scroll down to bucket policy section and click on Edit.

Paste below contents and click on Save changes.

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::aws-github-prod/*"
        }
    ]
}

Next click on Properties on your aws-github-prod S3 bucket page.

Scroll down to the bottom to the static website hosting. Click on Edit.

Under static website hosting check Enable.

Scroll down and enter enter index.html value to Index Document and to Error Document. Then click on Save changes button at the bottom of below page where you have to scroll to the bottom.

GitHub Actions Workflows
Time for the real part of this article where we will write YAML file. Create .github folder at the root of your project. Inside that create workflows folder then create production.yml file as below.

mkdir .github
cd .github
mkdir workflows
cd workflows
touch production.yml

Open production.yml file and paste below code.

name: Production Build
on:
  push:
    branches:
      - master
jobs:
  build:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [17.x]
        
    steps:
    - uses: actions/checkout@v1
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ matrix.node-version }}
    - name: Yarn Install
      run: |
        yarn install
    - name: Development Build
      run: |
        yarn build
    - name: Deploy to S3
      uses: jakejarvis/s3-sync-action@master
      env:
        AWS_S3_BUCKET: ${{ secrets.AWS_PROD_BUCKET_NAME }}
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        SOURCE_DIR: "build"


The above job is going to run when we make a git push to master branch. Node 17 is the version to use. Finally it will follow the steps under the steps area where it will install dependencies, build for production and finally deploy it to aws-github-prod S3 bucket.

Let’s give it a try if it is working. Stage, commit and push your changes to master branch.

-git add -A
-git commit -m "initial commit"
-git push origin master

Now that our react application is successfully deployed it is time to run. Go to your S3 Bucket home on AWS and click Properties and scroll to the bottom where you should be able to view a link, click it and you should be able to see your react application live.





