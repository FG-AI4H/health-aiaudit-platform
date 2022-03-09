# Developer Guide

## 1. Background and Goals

The evaluation platform is part of a larger project to develop an ML assessment platform. The platform is developed by FG-AI4H and consists of multiple work streams namely, data storage, data cataloguing, reporting, annotation and our core package.

The platform is based on the open-source project EvalAi by CloudCV. EvalAi offers ML benchmarking in the form of challenges, offered by organizations for everyone to participate. Our goal is to adapt EvalAi’s technology, and provide a model evaluation platform to be integrated into a larger project. A mockup of our project, including user stories and wireframes can be found here.

## 2. Project Overview

### Structure

```
├── apps                                                      #Django applications   
│   ├── challenges                                            #Handles creating, modifying and deleting challenges
│   ├── hosts                                                 #Functionalities for challenge hosts
│   ├── participants                                          #Functionalities for challenge participants
│   ├── jobs                                                  #Handles processing and evaluating submission
│   ├── web
│   ├── accounts
│   └── base
├── docker                                                    #Contains Dockerfiles, environment variables and settings
│   ├── dev
│   └── prod
├── frontend                                                 
│   └── src
│       ├── css
│       ├── fonts
│       ├── js                                                #Important for Angular Controllers and hosting settings
│       └── views
├── scripts                                                   #Scripts provided by EvalAi that make your life easier
│   ├── deployment
│       ├── deploy
│       └── push
│   ├── migration
│   ├── tools
│   └── workers                                               #Contains important code for the submission/code_upload workers
├── settings                                                  #Backend settings for the entire project, depending on environment
│   ├── common.py
│   ├── dev.py
│   ├── prod.py
│   └── test.py
├── docker-compose-production.yml                             #Necessary for starting production environment
├── manage.py                                                 #Very important Django tool; allows admin access, DB migration etc.
```
- **Apps** - contains all important Django applications. Referenced here are controllers, database structures as well as access points. When in doubt, familiarize yourself with Django’s setup.

- **Docker** - contains Docker and environment files with all important variables and configurations. These containers are called using the appropriate docker-compose.yml file with docker-compose. These are the project’s docker containers:
    - **Django**: Backend
    - **NodeJs**: Frontend
    - **Celery**: Queuing service for submissions
    - **Worker**: Submission worker which forwards them to the evaluation closer in ECS
    - **Code-Upload-Worker**: Worker specifically for Docker-based challenges.

- **Frontend** - This folder contains the frontend (HTML, CSS, fonts, images etc.) It is Angular-based, but also uses Django’s template language. It is important to highlight that the frontend also contains a lot of the business logic, so be mindful when you adapt it. Static and Media content is stored in S3 in AWS.

- **Scripts** -  Scripts provided by EvalAi to make your life easier (deploy to AWS, evaluation worker scripts)

- **Settings** - Django Settings for the entire project, also depends on what environment you are using (development, staging or production)

### Architecture
![Architecture](images/architecture.png)
In general, we have our frontend (nodeJS) and our backend (Django) hosted as scalable Docker instances on AWS. Django additionally communicates with our RDS database and an S3 bucket. When a client makes a request it gets processed by a loadbalancer and forwarded to our backend. The SQS queue registers a new task, such as executing an evaluation, and spawns a worker environment, in which the evaluation is performed.

The platform is fully hosted on AWS. Mainly the following services are used:
 - **EC2**: Our compute instances that host the platform.
 - **RDS**: Out databases.
 - **S3**: Our static storage. Mainly used for frontend resources such as images, fonts etc. Our platform secrets are also stored there.
 - **IAM**: Access and permission configurations.
 - **ECS**: This is where our evaluation cluster is managed from. Handles individual tasks for every individual challenge.
 - **CloudWatch**: Our error logs are processed here.

### Our Deployment
We are currently working with two instances of our platform, one [staging](https://staging.health.aiaudit.org/) and one [production](https://health.aiaudit.org/). To work on both you need access to AWS, including the admin permission set. Once granted, it is best to use the AWS developer environment called Cloud9, which provides easy access to the instances.

As mentioned previously, the project is deployed as a docker cluster on the AWS instances. This is not optimal, as there is no fault-tolerance or resource management. As a long-term goal we plan to migrate to using a Kubernetes cluster on AWS instead which will then manage the platform.


## 3. How does X work?
### Starting the project
Sometimes the docker containers running on our EC2 instances receive a timeout error. Unfortunately, this causes the entire website to go down. If this is the case, go to the respective instance and first of check the status of the containers.
```bash
docker ps
```
If no containers appear, they need to be re-started with the following command. Type is either production or staging.
```bash
sudo docker-compose -f docker-compose-TYPE.yml up
```
### Performing changes
Simply changing the code on the instance will not actually do anything. The reason for this is that our platform is spawned from the docker containers, which are isolated from the instance itself. Instead you need to re-build the respective docker containers, such that they pull the new code from the instance.
If you want to re-build the entire project, use:
```bash
sudo docker-compose -f docker-compose-TYPE.yml build
```
If you just want to re-build a single or a subset of services use:
```bash
sudo docker-compose -f docker-compose-TYPE.yml <SERVICENAME>
```
Occassionally re-building the project won't work because Docker uses up a lot of disk space. In this case you should delete old images and containers with the following command:
```bash
sudo docker system prune -a
```
### Migrating from staging to production
The staging instance is used to develop novel features, but once finished, we want to migrate them to our production instance. To do this we need to merge our features from our staging to our master branch on GitHub. Due to security issues this repository is still private, but all developers will be given access. You should perform the following steps when mergeing:

1. **Testing**: Ensure that the new features are well tested before attempting to merge.
2. **Commit to Staging**: Commit your local changes on the staging instance to the **remote** staging branch. You can view the remote staging branch using this command:
```bash
remote -v
```
Push using this command:
```bash
git push origin staging:staging
```
It's also very helpful to have a look at the files that differ between the local and remote staging branch
```bash
git diff --name-only origin/YourBranch YourBranch
```
>[!ATTENTION]
 >Lastly it is VERY important to NEVER commit the .env files in the Docker folder or the ssl folder.

3. **Merge Pull Request**:
This is the tricky part of the migration process that frequently causes errors. Firstly, it can be very helpful to merge using the GitHub user interface, here [a detailed guide](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/merging-a-pull-request).
You should merge staging into master, but **keep** the staging branch after the merge is completed. You will likely run into merge errors where you need to manually decide which code to keep. Generally this fully depends on which file you're working on. If it's a frontend file or overwriting an old tool, you should go with the new version. There are a few exceptions where it's very important NOT to overwrite the old code. These include:
  - Any configuraiton files. Configs are different for staging and production. Examples are the settings, docker files, nodejs config files.
  - Instance specific configurations hardcoded. Examples are queue name, aws-region etc.
  >[!ATTENTION]
   >Throughout the entire project the aws-region needs to be set to eu-central-1. EvalAI's source code uses us-east-1 but this will cause errors

4. **Pull changes**:
After step 3 the changes are now merged into the master branch on GitHub, but not actually changed on the production instance. So you need to go to production and pull the changes from the remote repository doing:
```bash
git pull origin master
```
5. **Rebuild**:
Now the changes are on the production instance (and hopefully don't cause havoc). The last step is to re-build the docker containers.
```bash
sudo docker-compose -f docker-compose-TYPE.yml build
```
### Submission
While the code for submission processing is complete, it can be very helpful to understand how in general submissions are processed. As mentioned, there are two types of submissions: text-based and docker-based. For text-based submissions the pipeline is the following:
### Extending Database Model
### Managing Challenges
- Approve Admin Panel
### Changing SSL Certificates
### Re-set AWS passwords
## 3. Debug Help
The first thing to **always** do when something seems to be broken is to check the CloudWatch error logs. They are separated by staging/production and by services, i.e. backend, frontend, worker. They will indicate what went wrong. Be aware that while the errors are usually straightforward, they are often caused by other components, so simply heading to StackOverflow rarely works. A very common issue is that some configurations such as queue name, aws region or deployment type got overwritten and need to be changed back. For this reason it's helpful to keep track of recent changes by regularly making commits to our GitHub repository.

### Django Migrations Error
### Issues Migrating to Production
