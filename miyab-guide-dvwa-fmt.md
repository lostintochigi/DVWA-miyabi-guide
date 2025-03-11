# Introduction

If you try to run the container as is, you might run into several issues such as being unable to determine the ServerName, port binding issues, etc.

I have written this guide with the Miyabi system in mind. However, it might also be helpful for running WebApplication containers in servers where you don't have root access.

Now back to business, there are a few things that you need to keep in mind when trying to run a DVWA container on Miyabi:

1. The container environment in use is Singularity NOT Docker.
2. Network Args - The default Apache server config inside the container is set to PORT 80.
3. Running the MariaDB container.

There are two solutions to this issue:

## Solution 1

Run the DVWA container locally on your machine and use SSH reverse tunneling to connect to Miyabi.

When you do this, follow instructions on the digininja/DVWA GitHub page for running the container using docker compose.

Once the container is up and running on your local machine, use the following command to do reverse tunneling:

```bash
ssh -R 4280:localhost:4280 username miyabi-c.jcahpc.jp
```

> **Note:** Replace `username` here with your Miyabi username. This is the most hassle-free solution with minimum amount of setup required.

## Solution 2 - Modifying Apache configuration and running the MariaDB container in parallel

The Apache server inside the container is configured to host on port 80, which unfortunately isn't possible on a system where you don't have administrative privileges. To fix this, we'll need to convert the `.sif` to sandbox in order to be able to edit the config files.

First pull the image using the following command:
```bash
singularity pull docker://ghcr.io/digininja/dvwa:3587fb5
```

This will pull a DVWA image and convert it into a .sif file with the following name, 'dvwa_3587fb5.sif' 

Use the following command to convert it into a writable sandbox. Don't forget the writable flag:

```bash
singularity build --sandbox apache_modify_dvwa dvwa_3587fb5.sif
```

Now use Nano to edit the ports config file inside the Apache directory.
```bash
sed -i 's/Listen 80/Listen 8080/' ports.conf
```bas

You'll also need to edit the Apache config to define the ServerName. Set the ServerName to 127.0.0.1 using the following command:

```bash
# Command for setting ServerName
```

If you try to run the container, you will run into the issue of no runscript defined. Use the following command to copy the runscript from the `.sif` file to the sandbox container. It is best to copy the whole `.singularity` directory from `.sif` to the sandbox.

If you try to run the container now, you'll still run into the error that Apache is unable to write the log files.

Use `mkdir -p ~/apache-data/` to create a folder for binding the container.

Now you can run the container with the following command:

```bash
# Command for running the container
```

However, before you do that, you'll need to pull the MariaDB container from DockerHub for DB Server. Create a `.def` file in Nano with the following content:

```
# MariaDB .def file content
```

Now use this `.def` file to create a MariaDB `.sif` container:

```bash
# Command to create MariaDB container
```

Now use the following script to run both the MariaDB and DVWA container in the background:

```bash
# Script content for running containers
```

Don't forget to use `chmod +x` to make it executable.

Run it:

```bash
# Command to run the script
```

You can check the running containers using:

```bash
singularity instance list
```

To stop the containers, use:

```bash
singularity instance stop -a
```

You can check if it is running correctly by using the curl command:

```bash
curl localhost:8080/login.php
```

You can also use reverse SSH to connect to Miyabi and then use a GUI web browser to confirm it's running correctly.

Assuming you've set the Apache server to port 8080 and are running the container on miyabi-c node, use the following command:

```bash
ssh -R 8080:localhost:8080 username miyabi-c.jcahpc.jp
```

Be careful here to connect to the correct Miyabi node where you're actually running the container. There are multiple Miyabi nodes: miyabi-c, miyabi-c1, miyabi-c2, etc.