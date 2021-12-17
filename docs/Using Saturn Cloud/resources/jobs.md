# Jobs

A job is a set of code that can be set to run in one of three ways:

* By pressing the "start button" within Saturn Cloud
* By running on a preset schedule
* Via an HTTP POST request to Saturn Cloud for programmatic running

## Creating a job

To create a job, go to the **Resources** tab of Saturn Cloud and press **New Job**

![New job button](/images/docs/new-job-button.jpg "doc-image")

This will allow you to edit the job in detail.

![New job options](/images/docs/new-job-options.jpg "doc-image")

The options to set up the job are as follows:

* __Name__ - the name of the job (must be unique).
* __Command__ - the command to execute the job within the resource. Typically something like `python run.py` where `run.py` is a script in the resource.
* __Hardware__ - the type of hardware to use for the job, either a CPU or GPU.
* __Size__ - the size of the instance to use for the job.
* __Image__ - the image to use for the job.
* __Run this job on a schedule__ - should the job be run once, or on a repeating basis. If on a schedule, more options become available:
  * __Cron Schedule__ - the frequency at which to run the job, specified as a <a href="https://pkg.go.dev/gopkg.in/robfig/cron.v2" target='_blank' rel='noopener'>cron expression</a>. Note that scheduling can be at the minute level at most (not second)
  * __Concurrency Policy__ - should an instance of a job be able to run if the previous one hasn't stopped? Select _Allow_ for concurrent jobs, _Forbid_ if only one job can run at a time (cancelling the new job), and _Replace_ if only one job can run at a time (cancelling the original job).
  * __Backoff limit__ - attempts at starting the task before it should be considered it to have failed.
* __Environment Variables__ (Advanced setting) - any specific environment variable values for the job.
* __Start Script (Bash)__ (Advanced setting) - the script to run at startup of the image before the job is executed.
* __Working Directory__ (Advanced setting) - where within the file directory the job command should be executed.

Pressing **Create** will add the job to the resource list and you will be taken to the resource page for the new job. 

## Running Saturn Cloud jobs

* **On a schedule** - add the schedule to the settings of the job. After the schedule has been set the job will run automatically.
* **On demand** - press the **Start** button on the Job resource page to cause the job to immediately run.
* **Via an API request** - see the section below

### Running a job as an API

First, to ensure that the request is authenticated, you'll need your **Saturn Cloud user token**. You can get your token from the **Settings** page of Saturn Cloud. You will also need the **Job ID** of your job, which is the hexadecimal value in the URL of the job's resource page: `https://app.community.saturnenterprise.io/dash/resources/job/{job_id}`.

From Python, you can then call the API and trigger the job using the code below. Note that any HTTP request sending system, like curl or Postman, would work fine too:

```python
import requests

user_token = "youusertoken" # (don't save this directly in a file!)
job_id = "yourjobid"

url = f'https://app.community.saturnenterprise.io/api/jobs/{job_id}/start'
headers={"Authorization": f"token {user_token}"}
r = requests.post(url, headers=headers)
```

## Troubleshooting Jobs

For general troubleshooting, the logs for a  job can be viewed by clicking the **Status** button in the resource details.

![Job status link](/images/docs/job-status.jpg "doc-image")
