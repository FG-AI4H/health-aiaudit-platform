# Hosting Benchmarks
In addition to this documentation, our onboarding team is continually creating [training resources](https://drive.google.com/drive/folders/1VtZqqDt5YbtC9ODFdTVsJV9It3_8LKH2) that help new benchmarks host to understand the process better.

In general the process of uploading a benachmark to our platform consists of configuring and uploading [this starter package](https://github.com/FG-AI4H/health-aiaudit-benchmark-starters) to our platform. The repository also contains a demo configuration in the folder **retino-public-example**.
>[!ATTENTION]
 >Please note that after configuration you need to delete the demo folder. Otherwise it can cause errors when initializing the challenge on the platform.

## Overview of the starters package
```
├── annotations                                                                 # Contains the annotations (ground truths)
│   ├── test_annotations_devsplit.json                                       
│   └── test_annotations_testsplit.json  
├── challenge_data                                                              # Contains the challenge test data
│   ├──  challenge_1                                                            # Evaluation script for the challenge
│        ├──__init__.py                                                         # Main.py file for evaluation
│        └──main.py                                                             # Challenge evaluation script
│   └── __init__.py                                                             
├── code_upload_challenge_evaluation                                            # Docker-based configurations
    ├── agent    
    ├── docker    
    ├── environment                   
    ├── requirements               
    ├── utils           
    └── docker-compose.yml
├── evaluation_script                                                           # Contains the evaluation script
│   ├── __init__.py                                                             # Imports the external libraries
│   └── main.py                                                                 # Main `evaluate()` method
├── logo.jpg                              
├── submission.json                   
├── github                                                 
├── remote_challenge_evaluation                                                 # NOT needed
└── templates                                                                   # HTML templates for benchmark descriptions
    ├── challenge_phase_1_description.html      
    ├── challenge_phase_2_description.html     
    ├── description.html                        
    ├── evaluation_details.html                 
    ├── submission_guidelines.html              
    └── terms_and_conditions.html                     
├── worker                                      
│   ├── __init__.py                             
│   └── run.py                                                                  # Run the evaluation locally
├── challenge_config.yaml                                                       # Define challenge setup
├── run.sh                                                                      # Bash file to create configuration zip
```

## Challenge Configuration
The first step when creating a new benchmark is to configure the **challenge_config.yaml** file. It contains all the necessary settings for your benchmark from title, start date to all the dataset splits. For an extensive overview of all fields and settings please view the EvalAI documentation [here](https://evalai.readthedocs.io/en/latest/configuration.html).

>[!ATTENTION]
 >We have extended the challenge_config.yaml with additional fields that are not included in the official documentation. Please view the section QUESTIONNAIRE for more details.


## Text-Based Evaluation

Only the folders annotations, evaluation_script and templates are relevant for this type of challenge, in addition to the challenge_config.yml. The following steps are necessary to host a text-based evaluation:
1. Configure the challenge_config.yml. Set the flags **remote_evaluation** and **is_docker_based** to FALSE.
2. Put your annotations into the annotation folder.
3. Define your evaluation script.
4. Edit the templates files with descriptions of your benchmark.
5. Execute **run.sh**. It generates a ZIP file that contains all elements of your benchmark.
6. Upload the ZIP [here](https://health.aiaudit.org/web/challenge-create).

## Docker-Based Evaluation
In the docker-based evaluation all files except remote_challenge_evaluation are necessary to be configured. Before you can begin with the configuration you need to know how your worker container will communicate with the participants submission container. All communication is defined in the **code_upload_challenge_evaluation** folder. This is not trivial and we will soon update the documentation with an extensive explanation on how to do this.

The following steps are necessary to host a docker-based evaluation:
1. Configure the challenge_config.yml. Set the flags **remote_evaluation** to FALSE. Set  **is_docker_based** and **is_static_dataset_docker_based_challenge** to TRUE.
2. Put your annotations into the annotation folder.
3. Put the test data in the challenge_data folder. This data will be pulled into the participants containers and his/her model willl be evaluated against it.
3. Define your evaluation script. It will receive the participant's containers results instead of a result upload.
4. Edit the templates files with descriptions of your benchmark.
5. Execute **run.sh**. It generates a ZIP file that contains all elements of your benchmark.
6. Upload the ZIP [here](https://health.aiaudit.org/web/challenge-create).

## Evaluation Script
The evaluation script is the core component of the benchmark, wherein the model is evaluated against a variety of metrics. The evaluation script is fully customizable and up to the host to define. While this makes the process very flexible, it comes with the challenge of setting up quite a lot by yourself.

What metrics you want to use should also be defined in the previous step in  the challenge configuration. This is necessary so that the results can be displayed properly on the leaderboard. Additionally, you can define different phases, such as test and development. For each phase, you can define different evaluation scripts.
```json
leaderboard:
  - id: 1
    schema:
      {
        "labels": ["Accuracy", "F1", "Precision", "Recall"],
        "default_order_by": "Total",
        "metadata": {
          "Metric1": {
            "sort_ascending": True,
            "description": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
          },
          "Metric2": {
            "sort_ascending": True,
            "description": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
          }
        }
      }
```
Generally, the evaluation script works the same for both modes of evaluation. The main function required in the evaluation script is the evaluate() function, used by workers to evaluate the submission message.

The function evaluate() receives both the test_annotations, previously uploaded in the configuration file, and the user_submission through the platform, as well as the phase_codename.
```python
def evaluate(test_annotation_file, user_annotation_file, phase_codename, **kwargs):
    pass
```
It receives three arguments, namely:

1. **test_annotation_file**: The file previously uploaded by the benchmark host that contains the ground truths.
2. **user_annotation_file**: The file submitted by the participant for evaluation.
3. **phase_codename**: It is the codename of the challenge phase from the challenge_configuration.yaml. This is passed as an argument so that the script can take actions according to the challenge phase.

The core steps are the following:

  1. Install needed libraries using the provided init.py file
  2. Implement evaluation metrics such as accuracy using the annotations and submissions. If you offer multiple phases, implement an evaluation for all of them.
  3. Return the results according to the phase and defined metrics.

Here a snippet from our public example:
```python
def evaluate(test_annotation_file, user_submission_file, phase_codename, **kwargs):
    gt = pd.read_json(test_annotation_file)    
    pred = pd.read_json(user_submission_file)

    acc = accuracy_score(gt,p)
    f1 = f1_score(gt,p)
    prec = precision_score(gt,p)
    recall = recall_score(gt,p)

    output = {}
    if phase_codename == "dev":
        print("Evaluating for Dev Phase")
        output["result"] = [
            {
                "train_split": {
                    "Accuracy": acc,
                    "F1": f1,
                    "Precision": prec,
                    "Recall": recall,
                }
            }
        ]
        # To display the results in the result file
        output["submission_result"] = output["result"][0]["train_split"]

    elif phase_codename == "test":
        ...
    return output
```
The output should contain a key named result, which is a list containing entries per dataset split that is available for the challenge phase in consideration (in the function definition of evaluate() shown above, the argument: phase_codename will receive the codename for the challenge phase against which the submission was made).

## Questionnaire Config
In addition to the quantitative evaluation based on metrics, our goal is to provide qualitative assessment in form of a questionnaire. This questionnaire consists of 70 questions that also need to be configured in the **challenge_config.yaml** file.

Previously under *challenge_phases* both for DEV and TEST phase, there were only four default meta submission attributes under
*default_submission_meta_attributes*. Now every team should add approximately 70 attributes in both DEV and TEST phases with the proper indentation under *default_submission_meta_attributes*.
```yml
challenge_phases:
  - id: 1
    name: Dev Phase
    default_submission_meta_attributes:
        - name: pl_reg_int_use_prd_name
          is_visible: True
        - name: pl_reg_int_use_cl_use
          is_visible: True
        - name: pl_reg_int_use_prd_tsk
          is_visible: True
        ...

```
A full example that can be use are a starting point is found in our [starters repository.](https://github.com/FG-AI4H/health-aiaudit-benchmark-starters)
