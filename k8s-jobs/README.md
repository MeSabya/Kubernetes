# Working with Kubernetes Jobs
- A Job is a Kubernetes concept that creates one (or more) Pods, and ensures that a specified number of them successfully terminates
- Kubernetes Jobs are generally used for large computation and batch tasks.
- It can be used for sending emails, rendering frames, transcoding files, taking back of some of files and directory etc. 
- Jobs can be used to support parallel execution as well.

## K8s Job Policies

### Restart Policy
While executing jobs in temporary created pods, container can fail due to any number of reasons like Memory limit exceeded and Process exited with non-zero code
In this case we need to restart pod so that it can start executing it’s task again without failure. so we will be using below option:
> .spec.template.spec.restartPolicy = “OnFailure”

if We do not want container to be restarted then we can use below option in our job configuration:
> .spec.template.spec.restartPolicy = “Never”.

### Pod Backoff failure policy:
In some situations we want to have such policy where our job should be attempted to run for specific number of times and then also if it’s failing it should declare that job as failed.
> .spec.backoffLimit = 2

### ActiveDeadlineSecondsPolicy:
if .spec.activeDeadlineSeconds is set to some value in seconds it means job will be terminated after that particular time duration, 
no matters job completed or not, Pod has been created or not.
Activedead line seconds Policy takes precedence over backoffLimit policy as well.In this case once Job exceeds activeDeadlineSeconds, 
it will mark the job failed with below keywords:
type: failed
reason: DeadlineExceeded

## Types of Task
There are mainly three types of task we are suitable to run as Job:
- **Non-parallel jobs**
- **Parallel Jobs (with fix completion count)**
- ![image](https://user-images.githubusercontent.com/33947539/137770619-3b3b0222-ba4a-4b68-93f9-988c61e4c1ed.png)

  Specify **.spec.completions** to some positive value assume 4.This means job will create 4 pods and and 
  execute activities mentioned in that Job in all pods and once all pods will be completing successfully it’s task. Job will be finished.
  Eg. see below file where i mentioned .spec.completions to 4 and it has created 4 pods to execute task. once it has executed it has been marked as resolved status

- **Parallel Jobs (with work queue)**
  We have to specify **.spec.parallelism** in job file.In this case pod must co-ordinate among themselves to determine what each should work.
  Each pod is independently capable of determining whether it’s all peers are done or not When any Pod from the Job terminates with success, no new Pods are created
  Once at least one Pod has terminated with success and all Pods are terminated, then the Job is completed with success.  
  
  ![image](https://user-images.githubusercontent.com/33947539/137771248-01641041-4a6b-45b4-a964-19daf91807f8.png)
  
## CronJob Creation
It’s used for creating periodic and recurring tasks like running backups and sending email.

![image](https://user-images.githubusercontent.com/33947539/137771468-a10af3ec-06ff-4e18-a710-66be17f394f6.png)


# References:
- https://betterprogramming.pub/tutorial-how-to-use-kubernetes-job-and-cronjob-1ef4ffbc8e84
