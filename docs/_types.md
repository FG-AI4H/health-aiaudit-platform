# Our types of benchmarks
The main goal of our platform is to provide a flexible and reliable environment to test machine learning algorithms in health. We offer two types of benchmark pipelines, one solely testing the predictions of a model and the other testing the model itself. The results of both are displayed in a variety of performance metrics, yet the underlying pipelines and requirements differ. In the following section, the types of benchmarks will be explained and advice is given for when to use which. The explanations are from the perspective of a benchmark host. For explanations on how to participate, please view the respective section [Participate in Benchmarks](_participate.md).

## Prediction-based benchmarks (text-based)
The core idea behind prediction-based benchmarks is that the evaluation is based on the predictions of the model only. This means that he or she needs access to testing data for their model, manually test it and only upload the results to the platform. The predictions are then compared to the ground truths of the dataset, previously uploaded by the benchmark owner. These results will be analyzed and reviewed by the audit team, compared with the ground truths, and the general performance of the model evaluation. Additionally, the submission will be evaluated based on a qualitative questionnaire submitted in advance of the evaluation. Both the questionnaire and evaluation results will be viewed by our auditor to generate an extensive model report sheet.

### Benchmark Pipeline
![Prediction-based Pipeline](images/text.png)

### Requirements
  1. An evaluation script, defining the specific metrics for testing
  2. Test data you will provide. It needs to be publicly available for participants to test themselves.
  3. The annotated ground truths.

### When should it be used?
Prediction-based upload challenges are recommended if:
  - The test dataset can be made public.
  - The creation and built of the benchmark should be simple.
  - Desired evaluation metrics only rely on the comparison of ground truths and predictions.

## Code-upload-based benchmarks (Docker)
For code-upload-based benchmarks, a participant first needs to dockerize their trained model and then upload it through the EvalAI-CLI. From there the model is stored securely in our cloud environment. The testing environment is separate from the platform, to guarantee the isolation of the evaluation. The participant's docker image and the evaluation code are deployed in this separate environment and afterward, results are forwarded to the platform, where they can be viewed. Additionally, the submission will be evaluated based on a qualitative questionnaire submitted in advance of the evaluation. Both the questionnaire and evaluation results will be viewed by our auditor to generate an extensive model report sheet.

### Benchmark Pipeline
![Docker Pipeline](images/docker.png)

### Requirements
  1. An evaluation script, defining the specific metrics for testing
  2. Test data you will provide. It will not be available to the public.
  3. The annotated ground truths.
  4. An environment container, defining the communication between submitted model and test data. Requires knowledge of Docker and networking.

### When should it be used?
Prediction-based upload challenges are recommended if:
  - The test dataset cannot be made public.
  - The model's evaluation process can be isolated
