---
layout: post
title:  "Accessing GPUs from a Docker Swarm service"
description:  "GPUs are great for neural network training, but accessing the GPU from a Docker container can be tricky. This article shows how to get the advantages of Docker Swarm orchestration by advertising GPUs as swarm resources."
---

This article shows how to access GPUs from [Docker Swarm][swarm] services. In essence, we need to do two things:

1. Set the nodes in the cluster to advertise their GPUs as Docker generic resources;
1. Have the service specify the constraint that it needs GPU resources.

[swarm]:            https://docs.docker.com/engine/swarm/swarm-tutorial/

Once these are both in place, the swarm orchestrator can automatically allocate services that need GPUs to nodes that have GPUs, without us needing to manually place tasks on specific nodes. Yay!

However, please note that only one Docker service replica can be assigned to a given GPU; there is no time-sharing between services on a single node. Practically this means you need at least as many nodes with GPUs as tasks that require them. If you have 5 nodes with GPUs and start 6 replicas of your service, one replica will stay pending due to lack of resources.

This article assumes you are already familiar with a number of concepts. Here are some resources for more background information:

- [A nice introduction to Docker images and containers][tutorial];
- [A tutorial on Docker Swarm and service creation][swarm];
- [An introduction to specifying constraints on Docker services][constraints].

[tutorial]:         https://docker-curriculum.com/
[constraints]:      https://docs.docker.com/engine/swarm/services/#control-service-placement

## Why GPUs and Docker Swarm?

Why might you want to access GPUs from Docker Swarm services? For this article I'll assume that you want to rapidly train a lot of neural networks using [Apache Spark][spark]. We can use Docker Swarm to manage our Spark cluster, deploying the Spark master on one node and replicating the Spark workers across the remaining nodes. With this architecture, we can direct each worker to train a single network, and use the GPU on a given worker node to speed up the training time.

[spark]:            https://spark.apache.org/

## Accessing the GPU from your own software

Before we get to Spark workers and Docker services, we need to ensure that our neural network training code can access the GPU in the first place. The nodes in the cluster should have an nVidia GPU (e.g. [AWS EC2 instances starting with p][aws-instances]), and the [nVidia CUDA toolkit][cuda] installed.

[aws-instances]:    https://aws.amazon.com/ec2/instance-types/
[cuda]:             https://developer.nvidia.com/cuda-zone

You also need a framework for designing and training neural networks such as Tensorflow or Theano, or the higher-level wrapper Keras. If installing these Python packages yourself make sure to install the GPU-enabled versions, e.g.:

{% highlight bash %}
pip install tensorflow-gpu keras-gpu
{% endhighlight %}

If running on EC2, Amazon provides an [AMI for their GPU-enabled nodes][aws-ami] that comes with CUDA, Tensorflow, and Python already installed.

[aws-ami]:          https://docs.aws.amazon.com/mxnet/latest/dg/CUDA9_AML1.html

Now your Keras or Tensorflow neural network program should run on the GPU!
 

## Accessing the GPU from a Docker container

Containers are great for abstracting away the details of the native system that we're running on, but the GPU is one of the details that gets abstracted away! In order for a Docker container to access the GPU, [we need to use `nvidia-docker`][nvd] instead of `docker` to run containers.

[nvd]:              https://github.com/NVIDIA/nvidia-docker

On Linux we install `nvidia-docker` through the package manager in the usual way (e.g. `apt-get`). Then launching a container becomes:

{% highlight bash %}
nvidia-docker run <image_name>
{% endhighlight %}

If we launch our Keras program in this container, it will run on the GPU!

However, originally `nvidia-docker` didn't support Docker Swarm. This meant that Spark workers couldn't be replicated across nodes in a cluster. The work-around was to manually allocate a Spark worker to a specific node by issuing an `nvidia-docker run` command on that node, instead of issuing a `service create --replicas` request to the swarm manager. It gets the job done, but it misses all the nice benefits of orchestration.

In December 2017 `nvidia-docker2` was released which supports Docker Swarm. Yay! The rest of this article draws from [a GitHub comment explaining how to use `nvidia-docker` with Docker Swarm][comment] from January this year. If you previously had `nvidia-docker` installed, you need to uninstall it and change to `nvidia-docker2` for swarm support. For example:

[comment]:    https://github.com/NVIDIA/nvidia-docker/issues/141#issuecomment-356458450

{% highlight bash %}
sudo apt-get -y install nvidia-docker2
{% endhighlight %}


## Accessing the GPU from a Docker service

So how do we get Docker services to use the GPU? Well, in addition to the requirements above (CUDA, `keras-gpu`, `nvidia-docker2`) we need to do three more things:

- Configure the Docker daemon on each node to advertises its GPU
- Make the Docker daemon on each node default to using `nvidia-docker`
- Add a constraint to our Docker service specifying that it needs a GPU

Once we take these steps, the orchestrator will be able to see which nodes have GPUs and which services require them, and deploy our services accordingly!

### Configuring the Docker daemon

The first step is to find the identifier of the GPU on a specific node, so we can pass it to the daemon later. We find it and store it in an environment variable with this command:

{% highlight bash %}
export GPU_ID=`nvidia-smi -a | grep UUID | awk '{print substr($4,0,12)}'`
{% endhighlight %}

What this is doing is running `nvidia-smi -a`, finding the line containing 'UUID', then extracting the first 12 characters of the 4th column of this line. You can see [an example of the output of `nvidia-smi -a` in the comment here][smi]. Line 19 contains the UUID; columns 1, 2, and 3 are 'GPU', 'UUID', and ':' respectively. The first 12 characters of column 4 should be enough to uniquely identify this GPU.

[smi]:        https://devtalk.nvidia.com/default/topic/883054/cuda-programming-and-performance/multi-gpu-peer-to-peer-access-failing-on-tesla-k80-/post/4690740/#4690740

If we `echo $GPU_ID`, we can see it looks something like `GPU-c143e771` or `GPU-c5c84263`. 

Docker is launched and managed as a service through systemd. We can change its default behaviour by adding an override file, called `/etc/systemd/system/docker.service.d/override.conf`.

This file should contain the following lines:

{% highlight code %}
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fdd:// --default-runtime=nvidia --node-generic-resource gpu=${GPU_ID}
{% endhighlight %}

Note: the second line is essential, because it clears any previously set `ExecStart` commands. You'll get an error if this is missing.

What is the third line doing? Three things: it's saying that when we start the Docker daemon we want the default runtime to be `nvidia-docker` (instead of `docker`), and that this node provides a generic resource of type `gpu`. (The name `gpu` could be anything, but it should be the same thing across all the nodes in our cluster so that the orchestrator sees which nodes offer the same resource type.) Finally, it's saying that on this specific node, the generic `gpu` resource has the identifier we previously stored in `$GPU_ID`.

Next, we modify the file `/etc/nvidia-container-runtime/config.toml` to allow the GPU to be advertised as a swarm resource. Uncomment or add the following line to this file:

{% highlight code %}
swarm-resource = "DOCKER_RESOURCE_GPU"
{% endhighlight %}

After taking these three steps, we need to reload the Docker daemon (to pick up the new configuration override file), and start it:

{% highlight bash %}
sudo systemctl daemon-reload
sudo systemctl start docker
{% endhighlight %}

#### Scripting these steps

It's a bit tedious to manually take these steps on every node in our cluster. They can be scripted as follows:

{% highlight bash %}
export GPU_ID=`nvidia-smi -a | grep UUID | awk '{print substr($4,0,12)}'`
sudo mkdir -p /etc/systemd/system/docker.service.d
cat <<EOF | sudo tee --append /etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fdd:// --default-runtime=nvidia --node-generic-resource gpu=${GPU_ID}
EOF
sudo sed -i '1iswarm-resource = "DOCKER_RESOURCE_GPU"' /etc/nvidia-container-runtime/config.toml
sudo systemctl daemon-reload
sudo systemctl start docker
{% endhighlight %}

### Adding a service constraint

Now our cluster nodes are advertising to the swarm that they offer access to a GPU. The final step is to ensure that the service requests a GPU. We do this by adding to the Docker service create command `--generic-resource "gpu=1"`. The full command looks something like this:

{% highlight code %}
docker service create --generic-resource "gpu=1" --replicas 10 \
--name sparkWorker <image_name> \"service ssh start && \
/opt/spark/bin/spark-class org.apache.spark.deploy.worker.Worker spark://<spark_master_ip>:7077\"
{% endhighlight %}

The name of the generic resource being requested (`gpu` here) should match the name of the resource being advertised by the nodes.

Congratulations! The Docker swarm orchestrator will now distribute your Spark workers onto nodes with GPU capability.


