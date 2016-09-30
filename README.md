
# Orchestrating GPU-Accelerated Workloads on Amazon ECS

## Background
AWS Solutions Architects are seeing an emerging type of application for ECS: GPU-accelerated workloads, or, more specifically, workloads that need to leverage large amounts of GPUs across many nodes. For example, at Amazon.com, the Amazon Personalization Team runs significant Machine Learning workloads that leverage many GPUs on Amazon ECS. Let’s take a look at how ECS enables GPU workloads. 

## Solution

In order to run GPU-enabled work on an ECS cluster, a Docker image configured with [Nvidia CUDA drivers][1], which allow the container to communicate with the GPU hardware, is built and stored in EC2 Container Registry. An [ECS Task Definition][2] is used to point to the container image in ECR and specify configuration for the container at runtime, like how much CPU and memory each container should use, the command to run inside the container, if a data volume should be mounted in the container, where the source dataset lives in S3, and so on. 

Once the ECS Tasks are run, the ECS [scheduler][3] finds a suitable place to run the containers by identifying an instance in the cluster with available resources. As shown in the below architecture diagram, ECS can place containers into the cluster of GPU instances (“GPU slaves” in the diagram)

![Build, tag, and push Docker image Console](https://s3.amazonaws.com/ecs-machine-learning/architecture.png)

## Deploying the architecture

### Prerequisistes


### Accepting terms

0. Accept AWS Marketplace terms for Amazon Linux AMI with NVIDIA GRID GPU Driver:
https://aws.amazon.com/marketplace/pp/B00FYCDDTE

1. Wait for email confirmation that marketplace subscription is active

### Launch the stack

2. Launch cloudformation stack from template:
https://code.amazon.com/packages/ECS-Machine-Learning/blobs/mainline/--/dsstne.template
(The template will build the dsstne container on the ECS cluster instance. Note this can take 30-45 minutes and the cloudformation stack will not report completion until the entire build process is done.)

### Run the model

3. Find name of DSSTNE ECS Task Definition in cloudformation stack outputs

4. Run task on DSSTNE ECS Cluster (defaults should work fine). By running this task, you are essentially running the DSSTNE sample modeling here:
https://github.com/amznlabs/amazon-dsstne/blob/master/docs/getting_started/examples.md

### Collect predictions

5. Find name of CloudWatch Logs Group in cloudformation stack outputs

6. Look at the task logs for details and output from the task run and location of results file in S3

7. Find name of S3 Bucket for the results in the cloudformation stack outputs

8. View the results file in the S3 Bucket

### Bonus steps
9. Bonus step 1: Repeat step 4, but change the config URL and training command by overriding the task definition environment variables to perform a benchmark:
https://github.com/amznlabs/amazon-dsstne/blob/master/benchmarks/Benchmark.md

10. Bonus step 2: Modify the cfntemplate (or launch a new stack) and use a g2.8xlarge instead of g2.2xlarge. Repeat step 4, but override the training command in the task definition environment variables to use MPI to take advantage of all 4 GPUs: mpirun -np 4 train -c ....

11. In both bonus steps, look at the CloudWatch Logs to view the task logs (different training commands, taking advantage of multiple GPUs, etc.)


![Build, tag, and push Docker image Console](https://s3.amazonaws.com/ecs-machine-learning/architecture.png)

[1]: http://www.nvidia.com/object/cuda_home_new.html
[2]: http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_defintions.html
[3]: http://docs.aws.amazon.com/AmazonECS/latest/developerguide/scheduling_tasks.html