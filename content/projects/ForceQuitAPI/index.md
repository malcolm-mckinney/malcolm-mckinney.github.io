---
title: "Force Quit API" # Title of your project
date: 2023-04-21T19:24:34+09:00
weight: 2 # Order in which to show this project on the home page
external_link: "" # Optional external link instead of modal
resources:
    - src: "force-quit.png"
      params:
          weight: 1 # Optional weighting for a specific image in this project folder
---
Continuing for about a decade at LINE, after a developer initiated a deployment (build, restart, rsync operation) in error or if said deployment was hanging or stuck (running for several hours with no progress), there was no automated way to force quit the process.
In other words, in order for a developer to terminate his or her ongoing build, he or she must contact the Delivery Infrastructure team on Slack and make a request for them to SSH into the machine responsible for deployment (PMC) and kill the process.

One major problem of the manually killing the deployment in the console is that the Java process responsible for deployment spawns child processes which in turn also spawn child processes.

If only the grandfather process is killed, the child and grandchildren processes will be orphaned.
If only the grandchild process is killed, then the parent and grandparent processes may continue to run.

So, it’s necessary to kill the whole generation of processes to avoid high CPU usage or memory leaks.
Also, we want to avoid the manual operation of killing the process on the he server; the kill operation is quite dangerous and can disrupt unrelated service if the incorrect process id is inputted into the console.

We use [Apache Executor](https://commons.apache.org/proper/commons-exec/apidocs/org/apache/commons/exec/Executor.html) to run the shell scripts responsible for build and restart.

Even though the Apache Executor contains an ExecuteWatchdog which can kill the process, but it does not have the ability to kill the process tree.
Also, due to constraints of the version of GWT that PMC is using, we are limited to Java 8.
The very useful Process API which allows for the killing of the descendants, is only available from Java 9 and above.
So, we had no choice but to subclass the Apache Executor, since the Process was only available through a protected method: `protected Process launch(CommandLine command, Map<String, String> env, File dir)`

Using this Process, we needed to find the process id, however in Java 8, for java.lang.UnixProcess, the pid field is private.
(In Java 9, you can use Process::pid to get the process id). So the only way to get the pid here, unfortunately, is to use Java Reflection to get the process id.
With this process id, then we can run this script to kill the process tree using the Java Process Builder.


```
#!/bin/bash

self=""

while getopts "p:" opt; do
  case ${opt} in
    p ) self="${OPTARG}" ;;
  esac
done

function kill_process_tree() {

  local generation=$(pgrep -P "${1}");

  for child in "${generation}" do
    if [[ "${child}" ]]; then
      kill_process_tree "${child}";
    fi
  done

  kill "${1}";
}

kill_process_tree "${self}"
```

This solved the problem of not being able to force quit builds or restarts running locally on the PMC machine, but there one other other edge case that were unsolved: newly created “remote build servers”, which are Kubernetes pods that have been allocated to run the builds remotely in a distributed fashion.
These remote build servers help alleviate the CPU load of deployments on the PMC machine and distribute the work to multiple machines. However, we need to introduce the same kill logic to the remote build servers.

By using the formula: `hash(projectId) % (n of remote build servers)`, we are able to find the exact machine that is executing the deployment.
I introduced a simple process service in the remote build service repository, which is basically contains a mapping of projectId to processId.
With the processId, we can use the above script to finally kill the process.

In conclusion, I decreased workload of manual operations on administrators as well as gave more flexibility and control to developers' deployment environment by providing a force quit feature.

