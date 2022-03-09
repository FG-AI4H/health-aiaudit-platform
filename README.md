# AI/ML model assessment - benchmarking platform

![health-bow-01](https://user-images.githubusercontent.com/53757856/134309621-0fa65f0a-2310-4f22-aa03-4b0298f12cd6.png)

The [ITU/WHO Focus Group on Artificial Intelligence for Health](https://itu.int/go/fgai4h/), founded in 2018 by ITU-T in partnership with the World Health Organization (WHO), is establishing a standardized framework for the assessment of AI/ML models for health. This repository contains the source code of an experimental [benchmarking platform](https://health.aiaudit.org/) (which is based on, but not affiliated to the open-source tool EvalAI).

## Create a benchmark

To host a benchmark (a.k.a. challenge) on our [platform](https://health.aiaudit.org/), please create a configuration file using [our starter repository](https://github.com/FG-AI4H/AI4H-Starters). Within this configuration, you can create your evaluation script and upload your testing data. 


## Participate in a benchmark by submitting an AI/ML model to our platform

Benchmarking is offered in two versions: The text-based version allows the participant to submit the predictions/output of the AI model via command-line interface (CLI) or user interface (UI) to the platform, where they are compared with the ground truth in order to assess the model performance. The docker-based version enables the participant to submit the AI/ML model via CLI in a docker image to the platform, where the model performance is evaluated in a protected environment with the test dataset. An additional questionnaire can be submitted with the UI of the platform, and reviewed by an audit team.

To submit to the platform, you need to log in on [our platform](https://health.aiaudit.org/) as a verified user. For testing purposes this is the user **participant**. Please use the command-line tool [EvalAI-CLI](https://github.com/Cloud-CV/evalai-cli) for submission.
1. The CLI is by default configured to submit to the original EvalAI platform and not to [our platform](https://health.aiaudit.org/). Thus, the submission host needs to be change before sumission:
```
evalai host -sh <HOST_IP:8000>
```
2. Sign up for Challenge and Participant Team on the platform
3. Submission.
```
evalai login
evalai set_token <usr_token>
evalai push <image>:<tag> --phase <phase_name> 
```


## Project structure

```
├── apps                          # Django applications   
│   ├── challenges                # Handles creating, modifying and deleting challenges
│   ├── hosts                     # Functionalities for challenge hosts
│   ├── participants              # Functionalities for challenge participants
│   ├── jobs                      # Handles processing and evaluating submission
│   ├── web
│   ├── accounts
│   └── base
├── docker                        # Contains docker files, environment variables and settings
│   ├── dev
│   └── prod
├── frontend                                                 
│   └── src
│       ├── css
│       ├── fonts
│       ├── js                    # Important for Angular Controllers and hosting settings
│       └── views
├── scripts                       # Important scripts for deployment, submission and evaluation
│   ├── deployment
│       ├── deploy
│       └── push
│   ├── migration
│   ├── tools
│   └── workers                   # Contains important code for the submission/code_upload workers
├── settings                      # Backend settings for the entire project, depending on environment
│   ├── common.py
│   ├── dev.py
│   ├── prod.py
│   └── test.py
├── docker-compose-production.yml # Necessary for starting production environment
├── manage.py                     # Important Django tool; allows admin access, DB migration etc.
└── README.md
```

## Project overview

We are currently running two instances on AWS (EC2), hosting a staging and a production environment. To access these instances, you need an AWS account and access permission granted by us. Cloning this repository will not reproduce an executable project, as key configurations are not pushed here. 

1. **Production**- https://health.aiaudit.org
2. **Production-Staging** - https://staging.health.aiaudit.org 

The production environment corresponds with the master branch and the staging instance with the staging branch. The staging instance is currently at EvalAI version of July 2021 and will be updated again soon.


## Task overview

Task | Description | Stage | Assigned To
--- | --- | --- | --- 
Deploy Docker pipeline | Fix VPC issues and execute evaluation | Doing | Elora
Integrate Reporting Package | Integrate finished reporting tool into platform | Doing | Golam
Credential Management | Take security credentials out of project and store securely | Doing | Kaushik
Audit Development | Host first draft of additional audits | Doing | Jeremiah, Adullah
Integrate Data Package | Make datasets stored accessible for evaluation in AWS. Configure EvalAI to use them. | TODO | ----
Change EvalAI Auth | Change the Auth system to AWS Cognito for universal user management over all packages. | TODO | ----
Integrate EvalAI-CLI in UI | Currently the image upload is made through a command-line tool. Integration into UI would be good. | TODO | ----
