---
layout: post
title: Environment Variables for Cron Jobs
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

__cron__ is a Linux utility that allows users to enlist scripts to be routinely executed at a specified time. 

You can use it to schedule file backups, webscrape the latest stock prices, and even automatically retrain your model given new data!

## Scheduling Tasks using Cron Jobs

In one of our projects, we have to refresh our data twice a day to ensure end users are accessing the latest information.

We automate this by scheduling a cron job that calls the script at a specified time. You usually put all your cron jobs in a file called `crontab`. A single job will look something like

```bash
* * * * * script.sh
```

The above 5 asterisks represent a time frame - minute, hour, day of month, month and day of week respectively. So if you wanted to run a command at 3:15am every Sunday, it would look like

```bash
15 3 * * 0 script.sh
```
Sunday is denoted by 0, Monday by 1 up until Saturday which is 6.

If you want it to be run at 10pm everyday, you would have 

```bash
0 22 * * * script.sh
```

## Accessing environment variables

This post is about how to deal with the fact cron sets up a minimal shell environment to run the jobs, so when you execute a script that requires a environment variable from your usual shell in the command line, it won't work.

This is the case in both your local OS and containers. 

This was initially very annoying because for my project there were alot of exported variables required for connecting to databases and secrets.

In some cases the `PATH` variable was also different. 

It turns out that we can easily access our environment variables with

```bash
printenv
```

`printenv` will output the variables, allowing you to grab what you need and export it inside your cron environment.  

You can `cat` the output of `printenv` to a file and export it to the cron shell when executing the script.

## Minimal working example

Let's give it a try with a simple example. 

Schedule your cron job inside your `crontab` file by first opening it with

```bash
env EDITOR=vim crontab -e 
``` 

and add the cron job like below
 
```bash
*/1 * * * * script.sh > /tmp/cron_logs.txt 2>&1
```

We schedule the job to run every minute so we can check our code and see if it worked.

Check if your crontab has succcessfully added a new job with

```bash
crontab -l
```

Suppose you need the `TEST_VAR` variable for your cron job. If this is not sensitive then you can `cat` it to a file. Let's create the variable and save it to a file:

```bash
export TEST_VAR=test_var
printenv | grep "TEST_VAR" > /tmp/env_var.txt
```

Now let's create a cron job that requires `TEST_VAR`.

Your `script.sh` would look something like this:

```bash
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

If everything goes correctly, you should see a 'test_var' and 'Cron job successful' appear in your `cron_logs.txt` file every minute.

__Note:__ You should only use `$(<file)` if your environment variables are without any special characters. 

__Note2:__ You should never export all your environment variables. Cron likes to keep things minimal so make sure you only export what you need. 

When you `unset TEST_VAR` in the command line, the next cronjob should fail with an error message. 

```bash
unset TEST_VAR=test_var
printenv | grep "TEST_VAR" > /tmp/env_var.txt
```

That's it! Go wild with your cron jobs =) 
