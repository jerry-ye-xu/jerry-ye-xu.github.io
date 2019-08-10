---
layout: post
title: Environment Variables for CRON Jobs
category: Computer Science
categories: ['cron', 'bash', 'nifty tricks']
tags: ['cron', 'bash', 'nifty tricks']
---

---
<h2 class="no_toc">Table of Contents</h2>

* TOC
{:toc}

<!-- Need this for table of contents above -->
---

## What is cron?

__cron__ is a Linux utility that allows users to specify a command to be executed at a specified time. 

You can use to the schedule file backups, webscrapping the latest stock prices, automatic retraining of your model given new data etc. 

## Scheduling Tasks using Cron Jobs

In one of our projects, we have to refresh our data twice a day to ensure our end users are accessing the latest information.

We automate this by scheduling a cron job that calls the script at a specified time. You usually put all your cron jobs in a file called `crontab` and inside it'll a single job will look something like 

```{bash}
* * * * * script.sh
```

The above 5 asterisks represent a time frame - minute, hour, day of month, month and day of week respectively. So if you wanted to run a command on the 15th minute at 3am in the morning every Sunday, it would look like

```{bash}
15 3 * * 0 script.sh
```
Note that Sunday is denoted by 0, Monday by 1 etc. up until Saturday which is 6.

If you want it to be run at 10pm everyday, it might look like 

```{bash}
0 20 * * * script.sh
```

Anyway you get the point. 

## Accessing Environment Variables

Now, this post is not about how to set up cron jobs and instead about how to deal with the fact cron sets up a minimal shell environment to run the script in, so when you run a script that requires a environment variable from your usual shell in the command line, it won't work.

This is because cron does not execute processes that are normally run when starting up the command line shell.

This is the case in both your local OS and inside containers. 

This was initially very annoying because for my project there were alot of exported variables required for connecting to databases/vault secrets etc.

In some cases the `PATH` variable was also different. 

It turns out that we can easily access our environment variables with

```{bash}
printenv
```

`printenv` will print out your environment variables which you can export line by line to ensure you have everything you need.  

Now, you can `cat` this to a file export it to the cron shell when executing the script so that you can access the them.

## Minimal Working Example

Let's give it a try with a simple example. 

Schedule your cron job inside your `crontab` file by firsting opening it with

```{bash}
env EDITOR=vim crontab -e 
``` 

and add
 
```{bash}
*/1 * * * * script.sh > /tmp/cron_logs.txt 2>&1
```

We schedule the job to run every minute so we can check our code and see if it worked.

Check your crontab has succcessfully added a new job with

```{bash}
crontab -l
```

You will need to save the environment variables you want inside a file.

Suppose you need the `TEST_VAR` variable to do something. If this is not sensitive, then you can `cat` it to a file. Let's create the variable and save it to a file:

```{bash}
export TEST_VAR=test_var
printenv | grep "TEST_VAR" > /tmp/env_var.txt
```

Your `script.sh` would look something like this:

```{bash}
#!/bin/bash 

set -e 

export $(</tmp/env_var.txt)

# Now it exists in our cron shell environment 

if [[ ${#TEST_VAR} -lt 1 ]] 
then 
    echo "TEST_VAR does not exist! get_env.sh script failed."
    exit 1
else
    echo ${TEST_VAR}
fi

echo "Cron job successful."
```

If everything goes correctly, you should see a 'test_var' and 'Cron job successful' in your `cron_logs.txt` file appear every minute.

__Note:__ You should only use `$(<file)` if your environment variables are without any special characters. 

__Note2:__ You should never export all your environment variables. Cron likes to keep things minimal so make sure you only export what you need. 


When you `unset TEST_VAR` in the command line, the next cronjob should fail with an error message. 

```{bash}
unset TEST_VAR=test_var
printenv | grep "TEST_VAR" > /tmp/env_var.txt
```

That's it! Go wild with your cron jobs =) 

