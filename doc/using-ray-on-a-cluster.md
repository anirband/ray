## Using Ray on a cluster

Running Ray on a cluster is still experimental.

Ray can be used in several ways. In addition to running on a single machine, Ray
is designed to run on a cluster of machines. This document is about how to use
Ray on a cluster.

### Getting started with Ray on a cluster

These instructions work on EC2, but they may require some modifications to run
on your own cluster. In particular, on EC2, running `sudo` does not require a
password, and we currently don't handle the case where a password is needed.

1. Create a file `nodes.txt` of the IP addresses of the nodes in the cluster.
For example

        12.34.56.789
        12.34.567.89
The first node in the file is the "head" node. The scheduler will be started on
the head node, and the driver should run on the head node as well.

2. Make sure that the nodes can all communicate with one another. On EC2, this
can be done by creating a new security group and adding the inbound rule "all
traffic" and adding the outbound rule "all traffic". Then add all of the nodes
in your cluster to that security group.

3. Run something like

    ```
    python scripts/cluster.py --nodes nodes.txt \
                              --key-file key.pem \
                              --username ubuntu \
                              --installation-directory /home/ubuntu/
    ```
where you replace `nodes.txt`, `key.pem`, `ubuntu`, and `/home/ubuntu/` by the
appropriate values. This assumes that you can connect to each IP address
`<ip-address>` in `nodes.txt` with the command
    ```
    ssh -i <key-file> <username>@<ip-address>
    ```
4. The previous command should open a Python interpreter. To install Ray on the
cluster, run `install_ray()` in the interpreter. The interpreter should block
until the installation has completed. The standard output from the nodes will
be redirected to your terminal.
5. To check that the installation succeeded, you can ssh to each node, cd into
the directory `ray/test/`, and run the tests (e.g., `python runtest.py`).
6. Now that Ray has been installed, you can start the cluster (the scheduler,
object stores, and workers) with the command
`start_ray("/home/ubuntu/ray/scripts/default_worker.py")`, where the second
argument is the path on each node in the cluster to the worker code that you
would like to use. After completing successfully, this command will print out a
command that can be run on the head node to attach a shell (the driver) to the
cluster. For example,

    ```
    source "$RAY_HOME/setup-env.sh";
    python "$RAY_HOME/scripts/shell.py" --scheduler-address=52.50.28.103:10001 --objstore-address=52.50.28.103:20001 --worker-address=52.50.28.103:30001 --attach
    ```

7. Note that there are several more commands that can be run from within
`cluster.py`.

    - `install_ray()` - This pulls the Ray source code on each node, builds all
      of the third party libraries, and builds the project itself.
    - `start_ray(worker_path, num_workers_per_node=10)` - This starts a
      scheduler process on the head node, and it starts an object store and some
      workers on each node.
    - `stop_ray()` - This shuts down the cluster (killing all of the processes).
    - `restart_workers(worker_path, num_workers_per_node=10)` - This kills the
      current workers and starts new workers using the worker code from the
      given file. Currently, this can only run when there are no tasks currently
      executing on any of the workers.
    - `update_ray()` - This pulls the latest Ray source code and builds it.

### Running Ray on a cluster

Once you've started a Ray cluster using the above instructions, to actually use
Ray, ssh to the head node (the first node listed in the `nodes.txt` file) and
attach a shell to the already running cluster.