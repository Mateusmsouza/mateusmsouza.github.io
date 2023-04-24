---
layout: post
title: How to keep command running on background after logout on SSH server
categories: [Unix, Bash]
---

If you are working with a remote server, you might need to run a command that takes too long to complete. In such cases, you may want to close your local terminal, as it can be inconvenient and may even interrupt the command if connection is lost. A simple trick to handle this problem is nohup command. 

Nohup command allows you to run a command on a remote server and keep it running even after you disconnect from the server. Especially useful if you are running a long-running process, such as a data analysis script or a Machine Learning training.

To use nohup, you need to be connected to to server by SSH. Once you are logged in, run nohup:

```bash
nohup echo foo &
```

On this command "foo" will be printted. The "&" at the end of the command tells the server to run the command in the background, which allows you to disconnect from the server without interrupting the process.

For example, if you want to run a Python script called "some_script.py" on the remote server, you can use the following command:

```bash
nohup python some_script.py &
```

This will start the script in the background and keep it running even after you disconnect from the server.

You can also redirect the output of the command to a file using the ">" operator, like this:

```
nohup python some_script.py > output.log &
```

This will redirect the output of the command to a file called "output.log". This can be useful for debugging purposes, as you can check the output of the command at a later time.

To check the status of the process, you can use the "ps" command. This will show you a list of running processes on the server, including the process ID (PID) of the process you started with nohup. You can use this PID to terminate the process if necessary using the "kill" command.

In summary, the nohup command is a useful tool for running long-running processes on a remote server.

References

[https://man7.org/linux/man-pages/man1/nohup.1.html](https://man7.org/linux/man-pages/man1/nohup.1.html)