# Motivation

Saturn Cloud is a set of centralized tools for data scientists to do their work. It is infrastructure for data scientists to run code, train models, and deploy APIs, and replaces a data scientist having to manually set up their programming environment themselves.

## Motivation

The work of a data scientist involves tasks like cleaning data, training models, and creating analyses. In many companies, those tasks would primarily be done by the data scientist on their local workstations. The data scientist would install a programming language like R or Python and the particular packages they need on the machine. They’d do the work, and then they would share the results with rest of the organization (maybe by email). The code might be stored in a shared location like GitHub, otherwise it would only live on their machine.

Using a local machine however has a number of limitations:

- The hardware is fixed. If it turns out that the data scientist needs more memory, more processing power, or a GPU, then they would have no method to get that hardware.
- Dependencies are difficult to manage. While a data scientist may share the code they wrote in a Git repository, that often doesn’t capture exactly what the environment was that the code was running on. The analysis may only work with a particular combination of operating system, operating system dependencies, hardware, and programming language libraries. While language-specific tools like conda and renv can be used to capture some of the environment, it’s difficult to capture everything exactly. This can create situations where past work can’t be reproduced and has to be redone–a huge waste of data scientists' time.
- They aren’t easily backed up and secured. If a data scientist quits their job, it’s nearly impossible to transfer their work off their machine and to someone else. This is because not only are there highly specific dependencies installed, but files and folders are saved within the machine in ways that only make sense to the user. That means when someone leaves their work will often have to be duplicated. There are also security concerns about a laptop being stolen with vulnerable information on it, and it can be dropped in an especially large puddle.

Given these problems, a natural improvement that data science teams have been making has been to shift to cloud resources. Cloud virtual machines solve a number of the problems that local machines have. First, it is very easy to adjust the hardware on the virtual machines. They also are potentially more secure. You no longer have to worry about someone losing a laptop on a subway ride, and you can easily take snapshots to back up the machines at points in time.

However, they have the exact same issues when it comes to dependencies–it’s hard to keep track of exactly what’s installed on a virtual machine to ensure work can be shared with colleagues. As each data scientist creates more virtual machines for their work, it becomes more and more difficult to keep track of what each one is for and what’s installed on it. Cloud virtual machines are more expensive than local machines, and if they’re left running you can accidentally spend tens of thousands of dollars.

For a sufficiently large data science team using virtual machines can become total chaos. The number of virtual machines can become vast, and won’t be clear which ones are supporting vitally important projects and which ones are minor experiments that should be deleted. Often a person in the organization becomes the defacto “cloud parent” who is tasked with wrangling the chaos and helping data scientists get their virtual machines working the way they should. This further adds time and cost.

### The Solution

Saturn Cloud provides the power of cloud infrastructure for a data science team without adding chaos or technical complexity.

Saturn Cloud:
- lets a data scientist do a project with precisely the hardware and software they need,
- keep track of exactly the environment that the data scientist is using for reproducibility,
- allow data scientists to share work with each other,
- deploy code as scheduled pipelines to run or host APIs and dashboards, and
- gives the team leader admin privileges to manage the platform and its users.

### Other Solutions

We believe data science platforms should work with your existing patterns. Other products have attempted to solve the same problems. We have found that they usually cause more problems than they solve.

Often they:

- Force Python code to be packaged in highly specific ways before it can run on the platform.
- Force data scientists to only use a single branch of a single git repo.
- Force data scientists to keep all of their work in a single notebook.
- Limit which programming languages can be used on the platform (sorry R and Julia).
- Limit which IDEs the data scientists can use (sorry Visual Studio Code).
- Force code to be written in platform specific ways with proprietary APIs. This locks you in.
- Do not meet IT Security requirements.
