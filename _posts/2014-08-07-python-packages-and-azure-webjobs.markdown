---
layout: post
title:  "Python Packages and Azure WebJobs"
date:   2014-08-07 12:12:32
categories: azure python
comments: true
---
### Introduction
I had a simple to task to solve of merging two json files so being constantly curious I decided I would hamper any chance of performing this simple task by choosing to review two new technologies, *Python* and *Azure WebJobs*.

In this post I'm not going to discuss the finer details of *Python* but I hope to pass as quick tip regarding using *Python* packages with *Azure WebJobs*

### Using Python Packages with Azure WebJobs
As a *Python* programmer you will most likely be familiar with *pip* as a noob my lack of familiarity kept me from my bed until 1am.  As it turns out all pip does (most likely a vast understatement) is to download the python package and store it in a location referenced by your *PYTHONPATH*.  Since most Python packages are only collections of organised .py files these could be located anywhere as long as your *PYTHONPATH* has reference to them.

This turned out to be the key in working out how to use *Python* packages with *Azure WebJobs*, you can't use pip so the package must be uploaded as part of the WebJob package.

#### Step 1.
If you are using OSX and the default Python 2.7 install your packages installed with pip will be in  
`/usr/local/lib/python2.7/site-packages`, create a folder called `site-packages` in the root of your python job and copy any packages you need for your job into it.

#### Step 2
Next you need to modify your run.py or any other file which requires access to the package files.  At the top of the file add....  
{% highlight python %}
import sys  
sys.path.append("site-packages")
{% endhighlight %}

#### Step 3
Zip all this up and upload as a WebJob or alternatively if you have previously created a job upload the files over FTP.

#### Step 4
Sit back watch it all explode in a ball of fire, call me all maner of names for mixing a step, revert to Stack Overflow.
