
# Step-by-Step Guide to Install Dask Python

To set up Dask across multiple servers, it’s crucial to ensure that Python and all related packages are the same version on each server. This guide outlines how to prepare the system, install Python 3.10.13 from source, create a virtual environment, and install Dask on three servers (127.0.0.1, 127.0.0.2, and 127.0.0.3).

## Preparation for Installation

First, update your package manager and install the necessary dependencies for compiling Python from source:

```bash
sudo apt update
sudo apt-get install openssl make gcc
sudo apt-get install tk-dev
sudo apt-get install build-essential libbz2-dev libssl-dev libffi-dev libsqlite3-dev libgdbm-dev zlib1g-dev liblzma-dev libreadline-dev libncurses5-dev libxml2-dev libxmlsec1-dev libjpeg-dev libpng-dev
```

These packages are required to ensure Python is built correctly with all necessary components.

## Install Python from Source

Next, download and install Python 3.10.13 from source to ensure all servers are using the same version.

```bash
cd /opt
sudo wget https://www.python.org/ftp/python/3.10.13/Python-3.10.13.tgz
sudo tar xzvf Python-3.10.13.tgz
```

Change to the Python directory and build it:

```bash
cd Python-3.10.13
sudo ./configure
sudo make
sudo make install
```

After installation, create a symbolic link to ensure the system uses this version of Python:

```bash
sudo ln -fs /opt/Python-3.10.13/python /usr/bin/python3.10
```

Verify the installation:

```bash
python3.10 --version
```

You should see `Python 3.10.13` as the output.

## Set Up Dask in a Virtual Environment

Create a virtual environment for your Dask installation. First, install the `virtualenv` package if it’s not already installed:

```bash
sudo apt install python3-virtualenv
```

Create a directory for Dask and set up a virtual environment:

```bash
mkdir -p /shared/folder/dask
cd /shared/folder/dask
virtualenv dask_env
```

Activate the virtual environment:

```bash
source dask_env/bin/activate
```

## Create `requirements.txt` and Install Packages

Create a `requirements.txt` file to specify the necessary packages for Dask:

```bash
nano requirements.txt
```

Add the following lines to the file:

```
numpy==1.26.4
pandas==2.0.0
dask[complete]==2024.9.0
```

Install the packages using pip:

```bash
pip install -r requirements.txt
```

Check the Dask version to verify the installation:

```bash
dask --version
```

## Repeat on Other Servers

Repeat the above steps for all servers (127.0.0.1, 127.0.0.2, and 127.0.0.3) to ensure consistency across the distributed environment.

---

## Setting Up Dask Distributed System Using Systemd

This guide explains how to configure a distributed Dask setup across multiple servers, using systemd to manage the Dask scheduler and workers. By using systemd, we can ensure that the services run in the background and automatically restart if they fail.

### Set Up Dask Scheduler

#### Run the Dask Scheduler

Start by manually running the Dask scheduler:

```bash
dask scheduler --host 127.0.0.1
```

This will start the scheduler on the specified IP address (127.0.0.1 in this case).

**Expected Output:**

Once the scheduler starts, you should see output confirming that the Dask scheduler is running, with information about the dashboard and the scheduler’s address. If no errors appear, you can proceed to the next step.

To stop the running process and set it up as a systemd service, press:

```bash
CTRL + C
```

#### Create a Dask Scheduler Systemd Service

We will now automate the scheduler startup by creating a systemd service.

```bash
sudo nano /etc/systemd/system/dask-scheduler.service
```

Add the following content to the file:

```ini
[Unit]
Description=Dask Scheduler
After=network.target

[Service]
Type=simple
User=your_username
WorkingDirectory=/shared/folder/dask

ExecStart=/bin/bash -c 'source /shared/folder/dask/dask_env/bin/activate && dask scheduler --host 127.0.0.1'

Restart=on-failure

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Replace `your_username` with the username you use on your server.

#### Enable and Start the Scheduler Service

Reload systemd to recognize the new service, enable it to start at boot, and then start the scheduler:

```bash
sudo systemctl daemon-reload
sudo systemctl enable dask-scheduler
sudo systemctl start dask-scheduler
```

Check the status of the Dask scheduler:

```bash
sudo systemctl status dask-scheduler
```

### Set Up Dask Worker

#### Run the Dask Worker

Start by manually running the Dask worker:

```bash
dask worker 127.0.0.1:8786
```

This command connects the worker to the scheduler running at 127.0.0.1 on port 8786.

**Expected Output:**

The output should show that the worker successfully connected to the scheduler. If no errors are encountered, proceed to the next step.

To stop the running process and set it up as a systemd service, press:

```bash
CTRL + C
```

#### Create a Dask Worker Systemd Service

Now, we will automate the worker startup by creating a systemd service for the Dask worker:

```bash
sudo nano /etc/systemd/system/dask-worker.service
```

Add the following content to the file:

```ini
[Unit]
Description=Dask Worker
After=network.target

[Service]
Type=simple
User=your_username
WorkingDirectory=/shared/folder/dask

ExecStart=/bin/bash -c 'source /shared/folder/dask/dask_env/bin/activate && dask worker 127.0.0.1:8786 --nthreads 96 --memory-limit 500GB'

Restart=on-failure

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Replace `your_username` with your server’s username.

Reload systemd to recognize the new service, enable it to start at boot, and then start the worker:

```bash
sudo systemctl daemon-reload
sudo systemctl enable dask-worker
sudo systemctl start dask-worker
```

Check the status of the Dask worker:

```bash
sudo systemctl status dask-worker
```

Repeat the above steps for servers (127.0.0.2, and 127.0.0.3) to ensure consistency across the distributed environment.

---

## Testing the Dask Distributed Setup

To test your distributed Dask setup, we will create a simple Python script that connects to the Dask scheduler, performs some basic computations, and returns the result.

### Create a Python File for Testing

Create a file named `testing.py` in your project directory:

```bash
nano testing.py
```

### Add the Following Code to `testing.py`

```python
from dask.distributed import Client
import dask.array as da

# Connect to the Dask scheduler
client = Client('10.70.200.2:8786')

# Print cluster information
print(client)

# Example computation: Create a large Dask array and compute the sum
array = da.random.random((10000, 10000), chunks=(1000, 1000))
result = array.sum()

# Submit computation to the cluster
print("Submitting task...")
result_computed = result.compute()
print("Computation result:", result_computed)
```

In this script:

- We first connect to the Dask scheduler using `Client('10.70.200.2:8786')`. Replace `10.70.200.2` with your scheduler’s actual IP address.
- We then print information about the connected Dask cluster.
- Next, we create a large random Dask array and calculate the sum of its elements.
- The computation is submitted to the cluster using `result.compute()`, and the result is printed.

### Run the Script

Activate your Dask environment and run the `testing.py` script:

```bash
source /shared/folder/dask/dask_env/bin/activate
python testing.py
```

### Verify Output

If everything is set up correctly, the script will:

- Print information about the Dask cluster.
- Submit the computation to the workers.
- Return the computed result of the sum.

If you see the result without any errors, congratulations! You have successfully set up a distributed processing system using Dask Python.
