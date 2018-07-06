---
status: in progress
---

Tuesday Exercise 1.3: Running jobs in the OSG
=============================================

The goal of this exercise is to have your jobs running on the OSG and map their geographical locations.

Where in the world are my jobs? (Part 2)
----------------------------------------

In this version of the geolocating exercise, you will submit jobs to the OSG from `osg-learn.chtc.wisc.edu` and
hopefully getting back much more interesting results!
Due to some differences between the machines in the OSG and our local cluster here at UW-Madison, you will be using a
slightly different payload and then performing the geolocation on the results from the submit host.

### Hostname fetching code

The following Python script finds the ClassAd of the machine it's running on and finds a network identity that can be
used to perform lookups:

```
#!/bin/env python

import re
import os
import socket

machine_ad_file_name = os.getenv('_CONDOR_MACHINE_AD')
try:
    machine_ad_file = open(machine_ad_file_name, 'r')
    machine_ad = machine_ad_file.read()
    machine_ad_file.close()
except TypeError:
    print socket.getfqdn()
    exit(1)

try:
    print re.search(r'GLIDEIN_Gatekeeper = "(.*):\d*/jobmanager-\w*"',
                    machine_ad,
                    re.MULTILINE).group(1)
except AttributeError:
    try:
        print re.search(r'GLIDEIN_Gatekeeper = "(\S+) \S+:9619"',
                        machine_ad,
                        re.MULTILINE).group(1)
    except AttributeError:
        exit(1)
```

### Gathering network information from the OSG

Now to create submit files that will run in the OSG!

1. If not already logged in, `ssh` into `osg-learn.chtc.wisc.edu`
1. Make a new directory for this exercise, `tuesday-1.3` and change into it
1. Save the above [hostname fetching code](#hostname-fetching-code) script to a file and call it `ce_hostname.py`
1. Create a submit file that runs `ce_hostname.py` 100 times and uses the `$(Process)` macro to write different `output`
   and `error` files
1. Submit your file and wait for the results

### Geolocating machines in the OSG

!!! warning "Submit host etiquette"
    In this section, we are bending a rule about running jobs locally on the submit host because this host is a closed
    environment only utilized by the class and this exercise is designed for low load.
    Normally you should **NOT** run jobs on your submit host.

You will be re-using the Python script from the the last exercise to perform the geolocation except instead of
submitting it as a job, you will run it manually.
Copy it over from `learn.chtc.wisc.edu` using `scp` with the following command:

1.  Log in to `learn.chtc.wisc.edu`
1.  Copy the file over to `osg-learn.chtc.wisc.edu`:

        :::console
        user@learn $ scp tuesday-1.1/location.py osg-learn.chtc.wisc.edu:tuesday-1.3/

`location.py` can take a text file that contains a list of locations as an argument, which can be done by collating your
output files into a single file.
The easiest way to do this is to use the `cat` command from today's exercise 1.2, the `*` wildcard from last exercise
and a new operator `>`, which can write command output to a file.
If your output files are named `ce_hostname-0.out...ce_hostname-99.out`, your commands would look like this:

```console
user@osg-learn $ cat ce_hostname-*.out > hostnames.txt
user@osg-learn $ ./location.py hostnames.txt
```

Mapping your jobs
-----------------

As before, you will be using <http://www.mapcustomizer.com/> from `osg-learn.chtc.wisc.edu` to visualize where your jobs
have landed in the OSG.
Copy and paste the results from the Python script into the bulk creation text box at the bottom of the screen. Where did
your jobs end up?

Extra Challenge: Running it all as a DAG
----------------------------------------

Yesterday, you learned about DAGs and you can take advantage of them to avoid manually running `location.py` by hand.
You could make this exercise into DAG with the hostname fetching submit file followed by a submit file that calls the
geolocation code.
You'll need to leverage `PRE` and/or `POST` scripts to collate the hostname results, which can be done with a little bit
of shell scripting!
To create a shell script:

1.  Write a file with a `.sh` extension
1.  Add a bash "shebang" line to the top of the script and lines for each shell command that you'd like to run (imagine
    it as a terminal in file form).
    For example, the following shell script would create a directory `foo` then list its contents:

        :::bash
        #/bin/bash
        mkdir foo
        ls foo

1.  Mark the script as executable
1.  Run it by hand to verify that it works

What did your results look this time around via DAG?