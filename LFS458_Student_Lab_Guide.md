# LFS458 — Kubernetes Administration
## Student Lab Guide
### 1 Control Plane + 2 Worker Nodes | Ubuntu 24.04
### Ubuntu 24.04 | kubeadm | containerd | Cilium CNI

---

## How to Use This Guide

- Each exercise begins with a `cd` command — **run it before anything else**.
- All commands are ready to run as-is — no need to become root separately.
- Commands that require elevated privileges already include `sudo`.
- YAML files referenced in the exercises are pre-staged in your lab folder.
- When you see `<output_omitted>` some output has been hidden for brevity — yours may look similar.
- If a command references a node name like `cp` or `worker`, use the name shown by `kubectl get nodes` on your cluster.
- **Control plane commands** run on your CP node (the one where you run `kubectl`).
- **Worker node commands** are indicated in the exercise text — open a second terminal for those.

---


---

# Chapter 2: Basics of Kubernetes
**Working directory: `~/lfs458/ch02-basics/`**

## Exercise 2.1: View Online Resources

Visit kubernetes.io
With such a fast changing project, it is important to keep track of updates. The main place to find documentation of the
current version is https://kubernetes.io/.
1. Open a browser and visit the https://kubernetes.io/ website.
2. In the upper right hand corner, use the drop down to view the versions available. It will say something like v1.30.
3. Select the top level link for Documentation. The links on the left of the page can be helpful in navigation.
4. As time permits navigate around other sub-pages such as SETUP, CONCEPTS, and TASKS to become familiar with the
layout.

Track Kubernetes Issues
There are hundreds, perhaps thousands, working on Kubernetes every day. With that many people working in parallel
there are good resources to see if others are experiencing a similar outage. Both the source code as well as feature
and issue tracking are currently on github.com.
1. To view the main page use your browser to visit https://github.com/kubernetes/kubernetes/
2. Click on various sub-directories and view the basic information available.
3. Update your URL to point to https://github.com/kubernetes/kubernetes/issues. You should see a series of issues, feature
requests, and support communication.
4. In the search box you probably see some existing text like isissue is:open: which allows you to filter on the kind of
information you would like to see. Append the search string to read: isissue is:open label:kind/bug: then press enter.
5. You should now see bugs in descending date order. Across the top of the issues a menu area allows you to view entries
by author, labels, projects, milestones, and assignee as well. Take a moment to view the various other selection criteria.
6. Some times you may want to exclude a kind of output. Update the URL again, but precede the label with a minus sign,
like: is:issue is:open -label:kind/bug. Now you see everything except bug reports.


3.1

Getting Started With Kubernetes

3.2

Minikube . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3.3

kubeadm . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3.4

More Installation Tools . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3.5

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

31
35

3.1


Getting Started With Kubernetes

Installation Tools
• Kubernetes Installation Tools
– kubeadm community tool
– Minikube
– MicroK8s
– Kubernetes Operations (kops)
• Managed Kubernetes Services
– Google Kubernetes Engine (GKE)
– Amazon Elastic Kubernetes Service (EKS)
– Microsoft Azure Kubernetes Service (AKS)
– DigitalOcean Kubernetes
• Use may need kubectl or proprietary command

This chapter is about Kubernetes cluster installation and configuration. We are going to review a few installation mechanisms
that you can use to create your own Kubernetes cluster. While the use of cloud providers is easy and common there are still
many reasons to deploy on-prem such as security, data sovereignty, and cost.
To get started without having to dive right away into installing and configuring your own cluster, there are a few choices. One
way is to use Google Kubernetes Engine (GKE), a Cloud service from the Google Cloud Platform, that lets you request
a Kubernetes cluster with the latest stable version. Amazon has a service Elastic Kubernetes Service (EKS), which allows
more control of the cp nodes. Azure and DigitalOcean are other options to investigate.
An easy way to get started locally is to use Minikube. It is a single binary which deploys into Oracle VirtualBox software.
While Minikube is only a single node, it will give you a learning, testing, and development platform. Another tool from Canonical,
MicroK8s, is aimed at easy feature-rich installation. More can be found here https://microk8s.io/.
A command line tool to use a Kubernetes cluster, kubectl, can be installed locally and configured to connect to one or more
clusters. It is a powerful tool that we will use throughout the rest of this course. So, you should become familiar with it.
You can also use a wrapper command such as gcloud. This runs locally on your machine and targets the Google API server
endpoint. It allows you to create, manage, and delete all Kubernetes resources and more. Each cloud provider offers a similar
wrapper.
To create our cluster will use kubeadm, the community-suggested tool from the Kubernetes project. It makes installing
Kubernetes easy and avoids vendor-specific installers. Getting a cluster running basically involves two commands: kubeadm
init, that you run on a cp node, and then, kubeadm join, that you run on your worker or redundant cp nodes, and your cluster
bootstraps itself. The flexibility of these tools allows Kubernetes to be deployed in a number of places and configuration
options. The course lab exercises use this method.
Several other installation mechanisms exist, such as kubespray and kops, with diffing levels of use and maturity. Each has
been built for a particular need. Some may become popular, others may become defunct within months.


3.1. GETTING STARTED WITH KUBERNETES

Installing kubectl

• Install or compile kubectl
• Main binary for working with objects
• Available for common distributions via dedicated repos
• Configuration file: $HOME/.kube/config
– apiserver endpoints
– User SSL keys
– contexts

To configure and manage your cluster, you can use the kubectl command. You can use RESTful calls or the Go language as
well, which is handy for integrating Kubernetes with your existing environment.
Enterprise Linux distributors have the various Kubernetes utilities and other files available in their repositories. For example,
on RHEL/CentOS, you would find kubectl in the kubernetes-client package. On OpenShift they use a command very similar
to kubectl called oc.
You can (if needed) download the code from github.com/kubernetes/kubernetes/tree/master/pkg/kubectl and go through the
typical steps to compile and install.
This command line tool uses $HOME/.kube/config as a configuration file. This contains the Kubernetes endpoints that you
might use, even for multiple clusters. If you examine it, you will see cluster definitions (i.e. IP endpoints), credentials, and
contexts.
A context is a combination of a cluster and user credentials. You can pass these parameters on the command line or switch
the shell settings between contexts with a command as in:
$ kubectl config use-context foobar

This is handy when going from a local environment to a cluster in the Cloud, or from one cluster to another, such as from
development to production.


Using Google Kubernetes Engine (GKE)

• Create account on GKE
• Add method of payment
• Install and use gcloud
– Vendor-specific command to manage GKE
• More details:
https://console.cloud.google.com/getting-started

Google takes every Kubernetes release through rigorous testing and makes it available via its GKE service. To be able to
use GKE, you will need an account on Google Cloud, a method of payment for the services you will use, and the gcloud
command line client.
There is an extensive documentation to get it installed. Pick your favorite method of installation and set it up.
See https://cloud.google.com/sdk/downloads#linux .
You will then be able to follow the GKE quickstart guide and you will be ready to create your first Kubernetes cluster:
$ gcloud container clusters create linuxfoundation
$ gcloud container clusters list
$ kubectl get nodes

By installing gcloud, you will have automatically installed kubectl. In the commands above, we created the cluster, listed it,
and then, listed the nodes of the cluster with kubectl.
Once you are done, do not forget to delete your cluster otherwise you will keep on getting charged for it:
$ gcloud container clusters delete linuxfoundation


3.2. MINIKUBE

3.2

Minikube

Using Minikube

• Open source project within GitHub Kubernetes
• Download from Google
• Assumes VirtualBox already installed
• Useful for developers
• Uses Go binary localkube
• Also uses Docker

While you can download a release from GitHub and compile following listed directions, it may be easier to download a precompiled binary. Make sure to verify and get the latest version.
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
$ chmod +x minikube
$ sudo mv minikube /usr/local/bin

With Minikube now installed, starting Kubernetes on your local machine is very easy:
$ minikube start
$ kubectl get nodes

This will start a VirtualBox virtual machine that will contain a single node Kubernetes deployment and the Docker engine.
Internally, minikube runs a single Go binary called localkube. This binary runs all the components of Kubernetes together.
This makes Minikube simpler than a full Kubernetes deployment. In addition, the Minikube VM also runs Docker, in order to
be able to run containers.


3.3


kubeadm

Install With kubeadm

• Easy to use with strong community support
• Works with current versions of Ubuntu and CentOS
• Main steps
– Run kubeadm init on the control plane node
– Create a network for IP-per-Pod criteria
– Run kubeadm join on workers or secondary cp nodes

Once you become familiar with Kubernetes using Minikube, you may want to start building a more production capable cluster.
Currently, the most straightforward method is to use kubeadm that be used to bootstrap a cluster quickly. As the community
has focused on kubeadm it has continued to mature with many features.
Documentation on how to do this is available on the Kubernetes website: https://kubernetes.io/docs/setup/independent/
create-cluster-kubeadm/.
Package repositories are available for current versions of Ubuntu and CentOS, among others. We will work with Ubuntu in
our lab exercises.
To join other nodes to the cluster you will need at least one token and a SHA256 hash from the control plane(cp). This
information is returned by the command, kubeadm init. Once the cp has initialized you would install and configure a network
plugin and be ready to start using the cluster. You can configure the network with kubectl, by using a resource manifest of the
network plugin to be used. Each plugin may have a different method of installation.


3.3. KUBEADM

kubeadm upgrade

• Allows validation prior to and eventual upgrade
• Upgrade to alpha, beta, release candidate, or stable release
• Sub-commands for various functions
– plan
– apply
– diff
– node

If you build your cluster with kubeadm, you also have the option to upgrade the cluster using the kubeadm upgrade command.
While most choose to remain with a version for as long as possible, and will often skip several releases, this does offer a useful
path to regular upgrades for security reasons.
plan - This will check the installed version against the newest found in the repository, and verify the cluster can be upgraded.
apply - Upgrades the first control plane node of the cluster to the specified version.
diff - Similar to an apply –dry-run this command will show the differences applied during an upgrade.
node - This allows for updating the local kubelet configuration on worker nodes, or the control planes of other cp nodes if there
is more than one Also will accept a phase command to step through the upgrade process.
General upgrade process:
• Update the software
• Check the software version
• Drain the control plane
• View the planned upgrade
• Apply the upgrade
• Uncordon the control plane to allow pods to be scheduled
Detailed steps can be found here: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/


Install A Pod Network

• Only one pod network per cluster
• Several to choose from
– Calico
– Canal
– Flannel
– Kube-router
– Cilium
• Can become complicated to manage
• Several add-ons available

Prior to initializing the Kubernetes cluster, the network must be considered and IP conflicts avoided. There are several Pod
networking choices, in varying levels of development and feature set.
Calico (projectcalico.org)
A flat Layer 3 network which communicates without IP encapsulation used in production with software such as Kubernetes,
OpenShift, Docker, Mesos and OpenStack. Viewed as a simple and flexible it scales well for large environments. Another
network option Canal, also part of this project, allows for integration with Flannel. Allows for implementation of network
policies.
Flannel (github.com/coreos/flannel)
A Layer 3 IPv4 network between nodes of a cluster. Developed by CoreOS it has a long history with Kubernetes. Focused on
traffic between hosts, not how containers configure local networking, it can use one of several backend mechanisms such as
VXLAN. A flanneld agent on each node allocates subnet leases for the host. While it can be configured after deployment it is
much easier prior to any Pods being added.
Kube-router (github.com/cloudnativelabs/kube-router)
Feature-filled single binary which claims to ”do it all”. The project is alpha stage but promises to offer a distributed load
balancer, firewall, and router purpose built for Kubernetes.
Cilium (https://cilium.io/) A newer but incredibly powerful network plugin which is used by major cloud providers. Via the use
of eBPF and other features this network plugin has become so powerful it is considered a service mesh, which we will discuss
later in the course.
Many of the projects will mention Container Network Interface (CNI) which is a CNCF project. Several container runtimes
currently use CNI. As a standard to handle deployment management and cleanup of network resources, it will become more
popular.


3.4. MORE INSTALLATION TOOLS

3.4

More Installation Tools

More Installation Tools

• kubespray
• kops
• kind

Since Kubernetes is, after all, like any other application installed on a server (whether physical or virtual), all of the configuration
management systems (e.g. Chef, Puppet, Ansible, OpenTofu) can be used. Various recipes are available on the Internet.
Here are just a few examples of other installation tools that can be used:
• kubespray
It is an advanced Ansible playbook which allows you to setup a Kubernetes cluster on various operating systems and
use different network providers. More information can be found here https://github.com/kubernetes-sigs/kubespray.
• kops
Lets you create a Kubernetes cluster on AWS and other providers via a single command line. It offers lots of features.
More information can be found here https://kops.sigs.k8s.io/.
• kind One of a few methods to run Kubernetes locally. Currently written to work with Docker. More information can be
found here https://kind.sigs.k8s.io/.

Please Note
A good way to learn how to install Kubernetes using step-by-step manual commands is to examine Kelsey Hightower’s
walk through: https://github.com/kelseyhightower/kubernetes-the-hard-way.


Installation Considerations

• Which provider should I use?
– Public or private cloud?
• Which operating system should I use?
• Which networking solution should I use?
– Do I need an overlay?
• Where should I run my etcd cluster?
• Should I configure Highly-Available Control Plane nodes?

To begin the installation process, you should start experimenting with a single-node deployment. This single-node will run all
the Kubernetes components (e.g. API server, controller, scheduler, kubelet, and kube-proxy). You can do this with Minikube,
for example.
Once you want to deploy on a cluster of servers (physical or virtual), you will have many choices to make, just like with any
other distributed system.
To learn more about how to choose the best options, you can see Picking the Right Solution at http://kubernetes.io/docs/
getting-started-guides/.
With systemd now the dominant init system on Linux, your Kubernetes pods will run kubelet service running on the cp
node(s).
The lab exercises were written using Google Compute Engine (GCE) nodes. Each node with 2 vCPUs and 7.5G of memory,
running Ubuntu 24.04. Smaller nodes should work, but you should expect slow response. Other operating system images are
also possible, but there may slight difference in some command output. Use of GCE requires setting up an account and will
incur expense if using nodes of the size suggested. You can view Getting Started pages here: https://cloud.google.com/
compute/docs/quickstart-linux
Amazon Web Service (AWS) is another provider of cloud-based nodes, and requires an account and will incur expense
for nodes of the suggested size. You can find videos and getting-started information here: https://aws.amazon.com/
getting-started/tutorials/launch-a-virtual-machine
Virtual machines such as KVM, VirtualBox, or VMWare can also be used for the lab systems. Putting the VMs on a private
network can make troubleshooting easier.
Finally using bare-metal nodes, with access to the internet, will also work for the lab exercises. Air gapped systems are
possible with proper planning and resources.


3.4. MORE INSTALLATION TOOLS

Main Deployment Configurations

• Single-node
• Single head node, multiple workers
• Multiple head nodes with HA, multiple workers
• HA etcd, HA head nodes, multiple workers
• Multi-cluster Federation also provides higher availability

At a high level, you have four main deployment configurations. Which of the four you will use will depend on how advanced
you are in your Kubernetes journey, but also on what your goals are.
With a single-node deployment all the components run on the same server. This is great for testing, learning, and developing
around Kubernetes.
Adding more workers, a single head node and multiple workers typically will consist of a single node etcd instance running on
the head node with the API, the scheduler, and the controller-manager.
Multiple head nodes in an HA configuration and multiple workers add more durability to the cluster. The API server will be
fronted by a load balancer, the scheduler and the controller-manager will elect a leader (which is configured via flags). The
etcd setup can still be single node.
The most advanced and resilient setup would be an HA etcd cluster, with HA head nodes and multiple workers. Also etcd
would run as a true cluster, which would provide HA and would run on nodes separate from the Kubernetes head nodes.
The use of Kubernetes Federation also offers higher availability. Multiple clusters are joined together with a common control
plane allowing movement of resources from one cluster to another administratively or after failure. While Federation has had
some issues there is hope v2 will be a stronger product. https://github.com/kubernetes-sigs/Kubefed


Compiling from Source

• Configure Golang environment
• Clone source code
• May need to install other compiler and libraries

The list of binary releases is available on GitHub: https://github.com/kubernetes/kubernetes/releases. Together with gcloud,
minikube, and kubeadm, these cover several scenarios to get started with Kubernetes.
Kubernetes can also be compiled from source relatively quickly. You can clone the repository from GitHub, and then use the
Makefile to build the binaries. You can build them natively on your platform if you have a Golang environment properly setup,
or via containers or virtual machines.
To build natively with Golang, first install Golang. Download and directions can be found here: https://golang.org/doc/install
Once Golang is working, you can clone the kubernetes repository, around 500 MB in size. Change into the directory and use
make:
$ cd $GOPATH
$ git clone https://github.com/kubernetes/kubernetes
$ cd kubernetes
$ make

There may be other software and settings you need in order for the make to work properly. Review the output until it completes
properly.
The _output/bin directory will contain the newly built binaries.
You may find some materials via https://gitlab.com/, but most resources are on https://github.com/ at the moment.


3.5

Labs


---


---

# Chapter 3: Installation and Configuration
**Working directory: `~/lfs458/ch03-install/`**

## Exercise 3.1: Install Kubernetes

```bash
cd ~/lfs458/ch03-install/
```


Overview
There are several Kubernetes installation tools provided by various vendors. In this lab we will learn to use kubeadm. As a
community-supported independent tool, it is planned to become the primary manner to build a Kubernetes cluster.

Platforms: Digital Ocean, GCP, AWS, VirtualBox, etc
The labs were written using Ubuntu 24.04 instances running on Google Cloud Platform (GCP). They have been written
to be vendor-agnostic so could run on AWS, local hardware, or inside of virtualization to give you the most flexibility
and options. Each platform will have different access methods and considerations. As of v1.21.0 the minimum (as in
barely works) size for VirtualBox is 3vCPU/4G memory/5G minimal OS for cp and 1vCPU/2G memory/5G minimal OS
for worker node. Most other providers work with 2CPU/7.5G.
If using your own equipment you will have to disable swap on every node, and ensure there is only one network interface.
Multiple interfaces are supported but require extra configuration. There may be other requirements which will be shown as
warnings or errors when using the kubeadm command. While most commands are run as a regular user, there are some
which require root privilege. You If you are accessing the nodes remotely, such as with GCP or AWS, you will need to use an
SSH client such as a local terminal or PuTTY if not using Linux or a Mac. You can download PuTTY from www.putty.org. You
would also require a .pem or .ppk file to access the nodes. Each cloud provider will have a process to download or create this
file. If attending in-person instructor led training the file will be made available during class.

Very Important
Please disable any firewalls while learning Kubernetes. While there is a list of required ports for communication between
components, the list may not be as complete as necessary. If using GCP you can add a rule to the project which allows
all traffic to all ports. Should you be using VirtualBox be aware that inter-VM networking will need to be set
to promiscuous mode.
In the following exercise we will install Kubernetes on a single node then grow the cluster, adding more compute resources.
Both nodes used are the same size, providing 2 vCPUs and 7.5G of memory. Smaller nodes could be used, but would run
slower, and may have strange errors.

YAML files and White Space
Various exercises will use YAML files, which are included in the text. You are encouraged to write some of the files as
time permits, as the syntax of YAML has white space indentation requirements that are important to learn. An important
note, do not use tabs in your YAML files, white space only. Indentation matters.
If using a PDF the use of copy and paste often does not paste the single quote correctly. It pastes as a back-quote instead.
You will need to modify it by hand. The files mentioned in labs have also been made available as a compressed tar file. You
can view the resources by navigating to this URL:


(Note: depending on your PDF viewer, if you are cutting and pasting the above instructions, the underscores may disappear
and be replaced by spaces, so you may have to edit the command line by hand!)

Install Kubernetes
Log into your control plane (cp) and worker nodes. If attending in-person instructor led training the node IP addresses
will be provided by the instructor. You will need to use a .pem or .ppk key for access, depending on if you are using
ssh from a terminal or PuTTY. The instructor will provide this to you.
1. Open a terminal session on your first node. For example, connect via PuTTY or SSH session to the first GCP node. The
user name may be different than the one shown, student. Create a non-root user if one is not present. The IP used in
the example will be different than the one you will use. You may need to adjust the access mode of your pem or ppk key.
The example shows how a Mac or Linux system would change mode. Windows may have a similar process.
# Connect to your CP node via SSH (your instructor will provide the IP)
ssh student@<CP_IP>
The authenticity of host '54.214.214.156 (35.226.100.87)' can't be established.
ECDSA key fingerprint is SHA256:IPvznbkx93/Wc+ACwXrCcDDgvBwmvEXC9vmYhk2Wo1E.
ECDSA key fingerprint is MD5:d8:c9:4b:b0:b0:82:d3:95:08:08:4a:74:1b:f6:e1:9f.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '35.226.100.87' (ECDSA) to the list of known hosts.
<output_omitted>

2. Use the wget command above to download and extract the course tarball to your node. Again copy and paste won’t
always paste the underscore characters.
3. You are encouraged to type out commands, if using a PDF or eLearning, instead of copy and paste. By typing the
commands you have a better chance to remember both the command and the concept. There are a few exceptions,
such as when a long hash or output is much easier to copy over, and does not offer a learning opportunity.
4. Become root and update and upgrade the system. You may be asked a few questions. If so, allow restarts and keep
the local version currently installed. Which would be a yes then a 2.
sudo -i
sudo apt-get update && apt-get upgrade -y
<output_omitted>

5. The main choices for a container environment are containerd, cri-o, and Docker on older clusters. We suggest containerd for class, as it is easy to deploy and commonly used by cloud providers.
Please note, install one engine only. If more than one are installed the kubeadm init process search pattern will
use Docker at the moment. Also be aware that engines other than containerd may show different output on some
commands.
6. There are several packages we should install to ensure we have all dependencies take care of. Please note the backslash is not necessary and can be removed if typing on a single line.


sudo apt install apt-transport-https tree \
software-properties-common ca-certificates socat -y
<output-omitted>

7. Disable swap if not already done. Cloud providers disable swap on their images.


swapoff -a

8. Load modules to ensure they are available for following steps.
sudo modprobe overlay
sudo modprobe br_netfilter

9. Update kernel networking to allow necessary traffic. Be aware the shell will add a greater than sign (>) to indicate the
command continues after a carriage return.
sudo cat << EOF | tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

10. Ensure the changes are used by the current kernel as well
sudo sysctl --system
* Applying /etc/sysctl.d/10-console-messages.conf ...
kernel.printk = 4 4 1 7
* Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
* Applying /etc/sysctl.d/10-kernel-hardening.conf ...
kernel.kptr_restrict = 1
<output_omitted>

11. Install the necessary key for the software to install
sudo mkdir -p /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

12. Install the containerd software.
sudo apt-get update && apt-get install containerd.io -y
sudo containerd config default | tee /etc/containerd/config.toml
sudo sed -e 's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml
sudo systemctl restart containerd


Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
<output_omitted>

13. Download the public signing key for the Kubernetes package repositories
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

14. Add the appropriate Kubernetes apt repository. Please note that this repository have packages only for Kubernetes
1.30; for other Kubernetes minor versions, you need to change the Kubernetes minor version in the URL to match your
desired minor version
sudo echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg]
https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /

15. Update with the new repo declared, which will download updated repo information.
sudo apt-get update
<output-omitted>

16. Install the Kubernetes software. There are regular releases, the newest of which can be used by omitting the equal sign
and version information on the command line. Historically new versions have lots of changes and a good chance of a
bug or five. As a result we will hold the software at the recent but stable version we install. In a later lab we will update
the cluster to a newer version.
sudo apt-get install -y kubeadm=1.33.1-1.1 kubelet=1.33.1-1.1 kubectl=1.33.1-1.1
<output-omitted>

sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.

17. Find the IP address of the primary interface of the cp server. The example below would be the ens4 interface and an IP
of 10.128.0.3, yours may be different. There are two ways of looking at your IP addresses.
sudo hostname -i
10.128.0.3

sudo ip addr show


....
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
link/ether 42:01:0a:80:00:18 brd ff:ff:ff:ff:ff:ff
inet 10.128.0.3/32 brd 10.128.0.3 scope global ens4
valid_lft forever preferred_lft forever
inet6 fe80::4001:aff:fe80:18/64 scope link
valid_lft forever preferred_lft forever
....

18. Add an local DNS alias for our cp server. Edit the /etc/hosts file and add the above IP address and assign a name
k8scp.
sudo vim /etc/hosts
10.128.0.3 k8scp
#<-- Add this line
10.128.0.3 cp
#<-- Add this line
127.0.0.1 localhost
....

19. Create a configuration file for the cluster. There are many options we could include, and they differ for containerd,
Docker, and cri-o. Use the file included in the course tarball. After our cluster is initialized we will view other default
values used. Be sure to use the node alias we added to /etc/hosts, not the IP so the network certificates will continue
to work when we deploy a load balancer in a future lab. The file is already included in the course tarball and doesn’t
require any changes.
sudo cp /home/student/LFS458/SOLUTIONS/s_03/kubeadm-config.yaml /root/
sudo vim kubeadm-config.yaml

kubeadm-config.yaml
2
4
6

apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
kubernetesVersion: 1.33.1
controlPlaneEndpoint: "k8scp:6443"
networking:
podSubnet: 192.168.0.0/16

#<-- Use the word stable for newest version
#<-- Use the alias we put in /etc/hosts not the IP

20. Initialize the cp. Scan through the output. Expect the output to change as the software matures. At the end are
configuration directions to run as a non-root user. The token is mentioned as well. This information can be found later
with the kubeadm token list command. The output also directs you to create a pod network to the cluster, which will
be our next step. Pass the network settings Cilium has in its configuration file. Please note: the output lists several
commands which following exercise steps will complete.
sudo kubeadm init --config=kubeadm-config.yaml --upload-certs --node-name=cp \
| tee kubeadm-init.out
#<-- Save output for future review
[init] Using Kubernetes version: v1.33.1
[preflight] Running pre-flight checks
<output_omitted>
You can now join any number of the control-plane node
running the following command on each as root:
kubeadm join k8scp:6443 --token vapzqi.et2p9zbkzk29wwth \


--discovery-token-ca-cert-hash
sha256:f62bf97d4fba6876e4c3ff645df3fca969c06169dee3865aab9d0bca8ec9f8cd \
--control-plane --certificate-key
911d41fcada89a18210489afaa036cd8e192b1f122ebb1b79cce1818f642fab8
Please note that the certificate-key gives access to cluster sensitive
data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If
necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.
Then you can join any number of worker nodes by running the following
on each as root:
kubeadm join k8scp:6443 --token vapzqi.et2p9zbkzk29wwth \
--discovery-token-ca-cert-hash
sha256:f62bf97d4fba6876e4c3ff645df3fca969c06169dee3865aab9d0bca8ec9f8cd

21. As suggested in the directions at the end of the previous output we will allow a non-root user admin level access to the
cluster. Take a quick look at the configuration file once it has been copied and the permissions fixed.
sudo exit
logout

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
less .kube/config
apiVersion: v1
clusters:
- cluster:
<output_omitted>

22. Deciding which pod network to use for Container Networking Interface (CNI) should take into account the expected
demands on the cluster. There can be only one pod network per cluster, although the CNI-Genie project is trying to
change this.
The network must allow container-to-container, pod-to-pod, pod-to-service, and external-to-service communications.
We will use Cilium as a network plugin which will allow us to use Network Policies later in the course. Currently
Cilium does not deploy using CNI by default.
Cilium is genereally installed using ”cilium install” or using ”helm install” commands. We have generated the
cilium-cni.yaml file using the below commands for your convenience. Note: You dont need to execute the commands in this box, they are just for reference.
$ helm repo add cilium https://helm.cilium.io/
$ helm repo update
$ helm template cilium cilium/cilium --version 1.16.1 \
--namespace kube-system > cilium.yaml

find $HOME -name cilium-cni.yaml


kubectl apply -f /home/student/LFS458/SOLUTIONS/s_03/cilium-cni.yaml
serviceaccount/cilium created
serviceaccount/cilium-operator created
secret/cilium-ca created
configmap/cilium-config created
<output_omitted>

23. While many objects have short names, a kubectl command can be a lot to type. We will enable bash auto-completion.
Begin by adding the settings to the current shell. Then update the $HOME/.bashrc file to make it persistent. Ensure the
bash-completion package is installed. If it was not installed, log out then back in for the shell completion to work.
sudo apt-get install bash-completion -y
<exit and log back in>
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> $HOME/.bashrc

24. Test by describing the node again. Type the first three letters of the sub-command then type the Tab key. Auto-completion
assumes the default namespace. Pass the namespace first to use auto-completion with a different namespace. By
pressing Tab multiple times you will see a list of possible values. Continue typing until a unique name is used. First
look at the current node (your node name may not start with cp), then look at pods in the kube-system namespace. If
you see an error instead such as -bash: _get_comp_words_by_ref: command not found revisit the previous step,
install the software, log out and back in.
kubectl des<Tab> n<Tab><Tab> cp<Tab>
kubectl -n kube-s<Tab> g<Tab> po<Tab>

25. Explore the kubectl help command. The output has been omitted from commands. Take a moment to review help
topics.
kubectl help
kubectl help create

26. View other values we could have included in the kubeadm-config.yaml file when creating the cluster.
sudo kubeadm config print init-defaults
apiVersion: kubeadm.k8s.io/v1beta4
bootstrapTokens:
- groups:
- system:bootstrappers:kubeadm:default-node-token
token: abcdef.0123456789abcdef
ttl: 24h0m0s
usages:
- signing
- authentication
kind: InitConfiguration
<output_omitted>


---

## Exercise 3.2: Grow the Cluster

```bash
cd ~/lfs458/ch03-install/
```

sudo kubeadm token create --print-join-command


Open another terminal and connect into a your second node. Install containerd and Kubernetes software. These are
the many, but not all, of the steps we did on the cp node.
This book will use the worker prompt for the node being added to help keep track of the proper node for each command.
Note that the prompt indicates both the user and system upon which run the command. It can be helpful to change the
colors and fonts of your terminal session to keep track of the correct node.
1. Using the same process as before connect to a second node. If attending an instructor-led class session, use the same
.pem key and a new IP provided by the instructor to access the new node. Giving a different title or color to the new
terminal window is probably a good idea to keep track of the two systems. The prompts can look very similar.
2. sudo -i
3. sudo apt-get update && apt-get upgrade -y
<If asked allow services to restart and keep the local version of software>

4. Install the containerd engine, starting with dependent software.
sudo apt install apt-transport-https
\
software-properties-common ca-certificates tree socat -y
sudo swapoff -a
sudo modprobe overlay
sudo modprobe br_netfilter
sudo cat << EOF | tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
sudo mkdir -p /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update && apt-get install containerd.io -y
sudo containerd config default | tee /etc/containerd/config.toml
sudo sed -e 's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml
sudo systemctl restart containerd

5. Get the GPG key for the software
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

6. Add Kubernetes repo


sudo echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list

7. Update repos then install the Kubernetes software. Be sure to match the version on the cp.
sudo apt-get update

8. sudo apt-get install -y kubeadm=1.33.1-1.1 kubelet=1.33.1-1.1 kubectl=1.33.1-1.1
9. Ensure the version remains if the system is updated.
sudo apt-mark hold kubeadm kubelet kubectl

10. Find the IP address of your cp server. The interface name will be different depending on where the node is running.
Currently inside of GCE the primary interface for this node type is ens4. Your interfaces names may be different. From
the output we know our cp node IP is 10.128.0.3.
hostname -i
10.128.0.3

ip addr show ens4 | grep inet
inet 10.128.0.3/32 brd 10.128.0.3 scope global ens4
inet6 fe80::4001:aff:fe8e:2/64 scope link

11. At this point we could copy and paste the join command from the cp node. That command only works for 2 hours, so
we will build our own join should we want to add nodes in the future. Find the token on the cp node. The token lasts 2
hours by default. If it has been longer, and no token is present you can generate a new one with the sudo kubeadm
token create command, seen in the following command.
sudo kubeadm token create --print-join-command
kubeadm join k8scp:6443 --token kcu55w.7jso85i0e2dsn05y \
--discovery-token-ca-cert-hash
sha256:0fb62b3c47bfd3af3c15d21f2ab6082fad1f913b244d5980816f8147ce9936ef

12. On the worker node add a local DNS alias for the cp server. Edit the /etc/hosts file and add the cp IP address and
assign the name k8scp. The entry should be exactly the same as the edit on the cp.
sudo vim /etc/hosts
10.128.0.3 k8scp
#<-- Add this line
10.128.0.3 cp
#<-- Add this line
127.0.0.1 localhost
....

13. Use the token and hash, in this case as sha256:long-hash to join the cluster from the second/worker node. Use the
private IP address of the cp server and port 6443. The output of the kubeadm init on the cp also has an example to
use, should it still be available.
sudo kubeadm join \
k8scp:6443 --token wcal99.hxv9v0gtnz42g6dr \
--discovery-token-ca-cert-hash \
sha256:0fb62b3c47bfd3af3c15d21f2ab6082fad1f913b244d5980816f8147ce9936ef --node-name=worker
[preflight] Running pre-flight checks


[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm
kubeadm-config -oyaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file
"/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

14. Try to run the kubectl command on the secondary system. It should fail. You do not have the cluster or authentication
keys in your local .kube/config file.
sudo exit
kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?

ls -l .kube
ls: cannot access '.kube': No such file or directory


---

## Exercise 3.3: Finish Cluster Setup

```bash
cd ~/lfs458/ch03-install/
```

1. View the available nodes of the cluster. It can take a minute or two for the status to change from NotReady to Ready.
The NAME field can be used to look at the details. Your node name may be different, use YOUR control-plane name
in future commands, if different than the book.
kubectl get node
NAME
cp
worker

STATUS
Ready
Ready

ROLES
control-plane
<none>

AGE
28m
50s

VERSION
v1.33.1
v1.33.1

2. Look at the details of the node. Work line by line to view the resources and their current status. Notice the status of
Taints. The cp won’t allow non-infrastructure pods by default for security and resource contention reasons. Take a
moment to read each line of output, some appear to be an error until you notice the status shows False.
kubectl describe node cp
Name:
Roles:
Labels:

cp
control-plane
beta.kubernetes.io/arch=amd64
beta.kubernetes.io/os=linux
kubernetes.io/arch=amd64
kubernetes.io/hostname=cp
kubernetes.io/os=linux
node-role.kubernetes.io/control-plane=
node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:
kubeadm.alpha.kubernetes.io/cri-socket:
unix:///var/run/containerd/containerd.sock


Taints:
<output_omitted>

node.alpha.kubernetes.io/ttl: 0
volumes.kubernetes.io/controller-managed-attach-detach: true
node-role.kubernetes.io/control-plane:NoSchedule

3. Allow the cp server to run non-infrastructure pods. The cp node begins tainted for security and performance reasons.
We will allow usage of the node in the training environment, but this step may be skipped in a production environment.
Note the minus sign (-) at the end, which is the syntax to remove a taint. As the second node does not have the taint
you will get a not found error. There may be more than one taint. Keep checking and removing them until all are
removed.
kubectl describe node | grep -i taint
Taints:
Taints:

node-role.kubernetes.io/control-plane:NoSchedule
<none>

kubectl taint nodes --all node-role.kubernetes.io/control-planenode/cp untainted
error: taint "node-role.kubernetes.io/control-plane" not found

kubectl describe node | grep -i taint
Taints:
Taints:

<none>
<none>

4. Determine if the DNS and Cilium pods are ready for use. They should all show a status of Running. It may take a minute
or two to transition from Pending.
kubectl get pods --all-namespaces
NAMESPACE
kube-system
kube-system
kube-system
kube-system

NAME
cilium-operator-788c7d7585-tnsph
cilium-swjsj
coredns-5d78c9869d-dwds8
coredns-5d78c9869d-t24p5

READY
1/1
1/1
1/1
1/1

STATUS
Running
Running
Running
Running

RESTARTS
0
0

AGE
95m
95m
100m
100m

<output_omitted>

5. Only if you notice the coredns- pods are stuck in ContainerCreating status you may have to delete them, causing
new ones to be generated. Delete both pods and check to see they show a Running state. Your pod names will be
different.
kubectl get pods --all-namespaces
NAMESPACE
NAME
kube-system
cilium-swjsj
kube-system
coredns-576cbf47c7-rn6v4
kube-system
coredns-576cbf47c7-vq5dz
<output_omitted>

READY
2/2
0/1
0/1

STATUS
RESTARTS
Running
ContainerCreating 0
ContainerCreating 0

kubectl -n kube-system delete \
pod coredns-576cbf47c7-vq5dz coredns-576cbf47c7-rn6v4


AGE
12m
3s
94m

pod "coredns-576cbf47c7-vq5dz" deleted
pod "coredns-576cbf47c7-rn6v4" deleted

6. When it finished you should see more interfaces will be created . It may take up to a minute to be created. You will notice
interfaces such as cilium interfaces when you deploy pods, as shown in the output below.
ip a
<output_omitted>
3: cilium_net@cilium_host: <BROADCAST,MULTICAST,NOARP,UP,LOWER_UP> mtu 1460 qdisc noqueue state UP
group default qlen 1000
link/ether be:19:22:da:62:ac brd ff:ff:ff:ff:ff:ff
inet6 fe80::bc19:22ff:feda:62ac/64 scope link
valid_lft forever preferred_lft forever
5: cilium_vxlan: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc noqueue state UNKNOWN group
default qlen 1000
link/ether ca:22:e7:23:42:89 brd ff:ff:ff:ff:ff:ff
inet6 fe80::c822:e7ff:fe23:4289/64 scope link
valid_lft forever preferred_lft forever
<output_omitted>

7. Containerd may still be using an out of date notation for the runtime-endpoint. You may see errors about an undeclared resource type such as unix//:. We will update the crictl configuration. There are many possible configuration
options. We will set one, and view the configuration file that is created. We will also set this configuration on worker
node as well for our convenience.
sudo crictl config --set \
runtime-endpoint=unix:///run/containerd/containerd.sock \
--set image-endpoint=unix:///run/containerd/containerd.sock
sudo crictl config --set \
runtime-endpoint=unix:///run/containerd/containerd.sock \
--set image-endpoint=unix:///run/containerd/containerd.sock
sudo cat /etc/crictl.yaml
runtime-endpoint: "unix:///run/containerd/containerd.sock"
image-endpoint: "unix:///run/containerd/containerd.sock"
timeout: 0
debug: false
pull-image-on-create: false
disable-pull-on-run: false


---

## Exercise 3.4: Deploy A Simple Application

```bash
cd ~/lfs458/ch03-install/
```

We will test to see if we can deploy a simple application, in this case the nginx web server.
1. Create a new deployment, which is a Kubernetes object, which will deploy an application in a container. Verify it is
running and the desired number of containers matches the available.
kubectl create deployment nginx --image=nginx
deployment.apps/nginx created

kubectl get deployments


NAME
nginx

READY
1/1

UP-TO-DATE

AVAILABLE

AGE
8s

2. View the details of the deployment. Remember auto-completion will work for sub-commands and resources as well.
kubectl describe deployment nginx
Name:
Namespace:
Labels:
Annotations:
Selector:
Replicas:
StrategyType:
MinReadySeconds:
RollingUpdateStrategy:
<output_omitted>

nginx
default
app=nginx
deployment.kubernetes.io/revision: 1
app=nginx
1 desired | 1 updated | 1 total | 1 ava....
RollingUpdate
25% max unavailable, 25% max surge

3. View the basic steps the cluster took in order to pull and deploy the new application. You should see several lines of
output. The first column shows the age of each message, note that due to JSON lack of order the time LAST SEEN time
does not print out chronologically. Eventually older messages will be removed.
kubectl get events
<output_omitted>

4. You can also view the output in yaml format, which could be used to create this deployment again or new deployments.
Get the information but change the output to yaml. Note that halfway down there is status information of the current
deployment.
kubectl get deployment nginx -o yaml

2
4
6

apiVersion: apps/v1
kind: Deployment
metadata:
annotations:
deployment.kubernetes.io/revision: "1"
<output_omitted>

5. Run the command again and redirect the output to a file. Then edit the file. Remove the creationTimestamp,
resourceVersion, and uid lines. Also remove all the lines including and after status:, which should be somewhere
around line 120, if others have already been removed.
kubectl get deployment nginx -o yaml > first.yaml
vim first.yaml
<Remove the lines mentioned above>

6. Delete the existing deployment.


kubectl delete deployment nginx
deployment.apps "nginx" deleted

7. Create the deployment again this time using the file.
kubectl create -f first.yaml
deployment.apps/nginx created

8. Look at the yaml output of this iteration and compare it against the first. The creation time stamp, resource version and
unique ID we had deleted are in the new file. These are generated for each resource we create, so we may need to
delete them from yaml files to avoid conflicts or false information. You may notice some time stamp differences as well.
The status should not be hard-coded either.
kubectl get deployment nginx -o yaml > second.yaml
diff first.yaml second.yaml
<output_omitted>

9. Now that we have worked with the raw output we will explore two other ways of generating useful YAML or JSON. Use
the --dry-run option and verify no object was created. Only the prior nginx deployment should be found. The output
lacks the unique information we removed before, but does have the same essential values.
kubectl create deployment two --image=nginx --dry-run=client -o yaml

2
4
6
8

apiVersion: apps/v1
kind: Deployment
metadata:
creationTimestamp: null
labels:
app: two
name: two
spec:
<output_omitted>

kubectl get deployment
NAME
nginx

READY
1/1

UP-TO-DATE

AVAILABLE

AGE
7m

10. Existing objects can be viewed in a ready to use YAML output. Take a look at the existing nginx deployment.
kubectl get deployments nginx -o yaml

2
4

apiVersion: apps/v1
kind: Deployment
metadata:
annotations:
deployment.kubernetes.io/revision: "1"


7
9

creationTimestamp: null
generation: 1
labels:
run: nginx
<output_omitted>

11. The output can also be viewed in JSON output.
kubectl get deployment nginx -o json

2
4
6
8

{

"apiVersion": "apps/v1",
"kind": "Deployment",
"metadata": {
"annotations": {
"deployment.kubernetes.io/revision": "1"
},
<output_omitted>

12. The newly deployed nginx container is a light weight web server. We will need to create a service to view the default
welcome page. Begin by looking at the help output. Note that there are several examples given, about halfway through
the output.
kubectl expose -h
<output_omitted>

13. Now try to gain access to the web server. As we have not declared a port to use you will receive an error.
kubectl expose deployment/nginx
error: couldn't find port via --port flag or introspection
See 'kubectl expose -h' for help and examples.

14. To change an object configuration one can use subcommands apply, edit or patch for non-disruptive updates. The
apply command does a three-way diff of previous, current, and supplied input to determine modifications to make.
Fields not mentioned are unaffected. The edit function performs a get, opens an editor, then an apply. You can
update API objects in place with JSON patch and merge patch or strategic merge patch functionality.
If the configuration has resource fields which cannot be updated once initialized then a disruptive update could be done
using the replace --force option. This deletes first then re-creates a resource.
Edit the file. Find the container name, somewhere around line 31 and add the port information as shown below.
vim first.yaml

first.yaml
2
4

....
spec:
containers:
- image: nginx
imagePullPolicy: Always


7
9
11

....

name: nginx
ports:
- containerPort: 80
protocol: TCP
resources: {}

# Add these
# three
# lines

15. Due to how the object was created we will need to use replace to terminate and create a new deployment.
kubectl replace -f first.yaml --force
deployment.apps/nginx replaced

16. View the Pod and Deployment. Note the AGE shows the Pod was re-created.
kubectl get deploy,pod
NAME
deployment.apps/nginx

1/1

READY

UP-TO-DATE

NAME
pod/nginx-7db75b8b78-qjffm

READY
1/1

STATUS
Running

AVAILABLE
2m4s

RESTARTS

AGE

AGE
8s

17. Try to expose the resource again. This time it should work.
kubectl expose deployment/nginx
service/nginx exposed

18. Verify the service configuration. First look at the service, then the endpoint information. Note the ClusterIP is not the
current endpoint. Cilium provides the ClusterIP. The Endpoint is provided by kubelet and kube-proxy. Take note
of the current endpoint IP. In the example below it is 192.168.1.5:80. We will use this information in a few steps.
kubectl get svc nginx
NAME
nginx

TYPE
ClusterIP

CLUSTER-IP
10.100.61.122

EXTERNAL-IP
<none>

PORT(S)
80/TCP

AGE
3m

kubectl get ep nginx
NAME
nginx

ENDPOINTS
192.168.1.5:80

AGE
26s

19. Test access to the Cluster IP, port 80. You should see the generic nginx installed and working page. The output should
be the same when you look at the ENDPOINTS IP address. If the curl command times out the pod may be running on
the other node. Run the same command on that node and it should work.
curl 10.100.61.122:80
<!DOCTYPE html>
<html>


<head>
<title>Welcome to nginx!</title>
<style>
<output_omitted>

curl 192.168.1.5:80

20. Now scale up the deployment from one to three web servers.
kubectl get deployment nginx
NAME
nginx

READY
1/1

UP-TO-DATE

AVAILABLE

AGE
12m

kubectl scale deployment nginx --replicas=3
deployment.apps/nginx scaled

kubectl get deployment nginx
NAME
nginx

READY
3/3

UP-TO-DATE

AVAILABLE

AGE
12m

21. View the current endpoints. There now should be three. If the UP-TO-DATE above said three, but AVAILABLE said two
wait a few seconds and try again, it could be slow to fully deploy.
kubectl get ep nginx
NAME
nginx

ENDPOINTS
192.168.0.3:80,192.168.1.5:80,192.168.1.6:80

AGE
7m40s

22. Find the oldest pod of the nginx deployment and delete it. The Tab key can be helpful for the long names. Use the
AGE field to determine which was running the longest. You may notice activity in the other terminal where tcpdump is
running, when you delete the pod. The pods with 192.168.0 addresses are probably on the cp and the 192.168.1
addresses are probably on the worker
kubectl get pod -o wide
NAME
nginx-1423793266-7f1qw
nginx-1423793266-8w2nk
nginx-1423793266-fbt4b

READY
1/1
1/1
1/1

STATUS
Running
Running
Running

RESTARTS
0

AGE
14m
86s
86s

IP
192.168.1.5
192.168.1.6
192.168.0.3

kubectl delete pod nginx-1423793266-7f1qw
pod "nginx-1423793266-7f1qw" deleted

23. Wait a minute or two then view the pods again. One should be newer than the others. In the following example nine
seconds instead of four minutes. If your tcpdump was using the veth interface of that container it will error out. Also
note we are using a short name for the object.


kubectl get po


NAME
nginx-1423793266-13p69
nginx-1423793266-8w2nk
nginx-1423793266-fbt4b

READY
1/1
1/1
1/1

STATUS
Running
Running
Running

RESTARTS
0

AGE
9s
4m1s
4m1s

24. View the endpoints again. The original endpoint IP is no longer in use. You can delete any of the pods and the service
will forward traffic to the existing backend pods.
kubectl get ep nginx
NAME
nginx

ENDPOINTS
192.168.0.3:80,192.168.1.6:80,192.168.1.7:80

AGE
12m


---

## Exercise 3.5: Access from Outside the Cluster

```bash
cd ~/lfs458/ch03-install/
```

You can access a Service from outside the cluster using a DNS add-on or environment variables. We will use environment variables to gain access to a Pod.
1. Begin by getting a list of the pods.
kubectl get po
NAME
nginx-1423793266-13p69
nginx-1423793266-8w2nk
nginx-1423793266-fbt4b

READY
1/1
1/1
1/1

STATUS
Running
Running
Running

RESTARTS
0

AGE
4m10s
8m2s
8m2s

2. Choose one of the pods and use the exec command to run printenv inside the pod. The following example uses the
first pod listed above.
kubectl exec nginx-1423793266-13p69 \
-- printenv |grep KUBERNETES
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
<output_omitted>

3. Find and then delete the existing service for nginx.
kubectl get svc
NAME
kubernetes
nginx

TYPE
ClusterIP
ClusterIP

CLUSTER-IP
10.96.0.1
10.100.61.122

EXTERNAL-IP
<none>
<none>

PORT(S)
443/TCP
80/TCP

AGE
4h
17m

4. Delete the service.
kubectl delete svc nginx
service "nginx" deleted


5. Create the service again, but this time pass the LoadBalancer type. Check to see the status and note the external ports
mentioned. The output will show the External-IP as pending. Unless a provider responds with a load balancer it will
continue to show as pending.
kubectl expose deployment nginx --type=LoadBalancer
service/nginx exposed

kubectl get svc
NAME
kubernetes
nginx

TYPE
ClusterIP
LoadBalancer

CLUSTER-IP
10.96.0.1
10.104.249.102

EXTERNAL-IP
<none>
<pending>

PORT(S)
443/TCP
80:32753/TCP

AGE
4h
6s

6. Open a browser on your local system, not the lab exercise node, and use the public IP of your node and node port
32753, shown in the output above. If running the labs on remote nodes like AWS or GCE use the public IP you used
with PuTTY or SSH to gain access. You may be able to find the IP address using curl.
curl ifconfig.io
54.214.214.156

Figure 3.1: External Access via Browser

7. Scale the deployment to zero replicas. Then test the web page again. Once all pods have finished terminating accessing
the web page should fail.
kubectl scale deployment nginx --replicas=0
deployment.apps/nginx scaled

kubectl get po
No resources found in default namespace.

8. Scale the deployment up to two replicas. The web page should work again.
kubectl scale deployment nginx --replicas=2
deployment.apps/nginx scaled


kubectl get po
NAME
nginx-1423793266-7x181
nginx-1423793266-s6vcz

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
6s
6s

9. Delete the deployment to recover system resources. Note that deleting a deployment does not delete the endpoints or
services.
kubectl delete deployments nginx
deployment.apps "nginx" deleted

kubectl delete svc nginx
service "nginx" deleted


4.1

Kubernetes Architecture . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

4.2

Networking . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

4.3

Other Cluster Systems . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

4.4

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

75
82


4.1

Kubernetes Architecture

Main Components

Figure 4.1: Architectural Overview

Main Components:
• Control Plane(s) and worker node(s)
• Operators
• Services
• Pods of Containers
• Namespaces and quotas
• Network and policies
• Storage
As mentioned in the previous chapter, a Kubernetes cluster is made of one or more cp node and a set of worker nodes. The
cluster is all driven via API calls to operators. A network plugin helps handle both interior as well as exterior traffic. We will
take a closer look at these components on following pages
Most of the processes are executed inside of a container. There are some differences depending on vendor and tool used to
build the cluster.
When upgrading a cluster be aware that each of these components are developed to work together by multiple teams. Care
should be taken ensure a proper match of versions. The kubeadm upgrade plan command is useful to discover this information.


4.1. KUBERNETES ARCHITECTURE

Control Plane Node
• kube-apiserver
• kube-scheduler
• etcd
• kube-controller-manager
• cloud-controller-manager
• CoreDNS
• Network plugin
• Add-ons
– Dashboard - Web UI
– Cluster-level resource monitoring
– cluster-level logging

The Kubernetes cp runs various server and manager processes for the cluster. As the software has matured, new components
have been created to handle dedicated needs, such as the cloud-controller-manager; it handles tasks once handled by the
kube-controller-manager to interact with other tools such as Rancher or DigitalOcean for third-party cluster management
and reporting.
There are several add-ons which have become essential to a typical production cluster, such as DNS services. Others are
third-party solutions where Kubernetes has not yet developed a local component such as cluster-level logging and monitoring.
As a concept the various pods responsible for ensuring the current state of the cluster matches the desired state are called
the control plane.
When building a cluster using kubeadm the kubelet process is managed by systemd. Once running it will start every pod
found in /etc/kubernetes/manifests/.


kube-apiserver

• Front-end of cluster’s shared state
• Control Plane for the cluster
• All components work through it
• Validates and configures data for API objects
• Services REST operations
• Only component to connect to etcd database

The kube-apiserver is central to the operation of the Kubernetes cluster.
All calls, both internal and external traffic are handled via this agent. All actions are accepted and validated by this agent and
it is the only connection to the etcd database. As a result it acts as a cp process for the entire cluster, and acts as a front-end
of the cluster’s shared state.
Starting as an beta feature in v1.18 Konnectivity service provides the ability to separate user-initiated traffic from serverinitiated traffic. Until these features are developed most network plugins commingle the traffic, which has performance, capacity, and security ramifications.


4.1. KUBERNETES ARCHITECTURE

kube-scheduler

• Uses algorithm to determine Pod placement
• Checks quota restrictions
• Custom scheduling policies possible
• Affinity rules to place pods on specific nodes
• Taints can be used to repel pods
• Pod bindings can force particular scheduling

The kube-scheduler uses an algorithm to determine which node will host a Pod of containers. The scheduler will try to view
available resources (such as volumes) to bind, and then try and retry to deploy the Pod based on availability and success.
There are several ways you can affect the algorithm, or a custom scheduler could be used instead. You can also bind a Pod
to a particular node, though the Pod may remain in a pending state due to other settings.
One of the first settings referenced is if the Pod can be deployed within the current quota restrictions. If so, then the taints,
tolerations, and labels of the Pods are used along with metadata of the nodes to determine the proper placement.
The details of the scheduler can be found at: https://raw.githubusercontent.com/kubernetes/kubernetes/master/pkg/
scheduler/scheduler.go


etcd Database

• Multiversion persistent b+tree key-value store
• Append only, regular compaction
• Shed oldest version of superseded data
• Works with curl and other HTTP libraries
• Provides reliable watch queries
• Distributed consensus protocol for leadership

The state of the cluster, networking and other persistent information is kept in an etcd database, or more accurately a b+tree
key-value store. Rather than find and change an entry, values are always appended to the end. Previous copies of the data
are then marked for future removal by a compaction process.
Simultaneous requests to update a value all travel via the kube-apiserver, which then passes along the request to etcd in a
series. The first request would update the database. The second request would no longer have the same version number, in
which case the kube-apiserver would reply with an error 409 to the requester. There is no logic past that response on the
server side, meaning the client needs to expect this and act upon the denial to update.
There is a Leader database along with possible Followers, or non-voting Learners who are in the process of joining the
cluster. They communicate with each other on an ongoing basis to determine which will be the Leader and determine another
in the event of failure. While very fast and potentially durable, there have been some hiccups with new tools such as kubeadm
and features like whole cluster upgrades.
While most objects of Kubernetes are designed to be decoupled, transient microservice which can be terminated without
much concern etcd is the exception. As it is the persistent state of the entire cluster it must be protected and secured.
Before upgrades or maintenance you should plan on backing up etcd. The etcdctl command allows for snapshot save and
snapshot restore.


4.1. KUBERNETES ARCHITECTURE

Other Agents
• kube-controller-manager
– Daemon which embeds core control-loops
– Watches state of cluster
– Works to make current state match desired state
• cloud-controller-manager (ccm)
– Interacts with outside cloud managers
– Allows features to be developed outside of core release cycle
– Each kubelet must use --cloud-provider-external
– Handles tasks once part of kube-controller-manager
– This is an optional agent which takes a few steps to enable
• Network plugins
• CoreDNS

The kube-controller-manager is a core control loop daemon which interacts with the kube-apiserver to determine the state
of the cluster. If the state does not match, the manager will contact the necessary controller to match the desired state. There
are several controllers in use, such as endpoints, namespace, and replication. The full list has expanded as Kubernetes
has matured.
Remaining in beta since v1.11 the cloud-controller-manager interacts with agents outside of the cloud. It handles tasks once
handled by kube-controller-manager. This allows faster changes without altering the core kubernetes control process. Each
kubelet must use the --cloud-provider-external settings passed to the binary. You can also develop your own CCM,
which can be deployed as a daemonset as an in-tree deployment or as a free-standing out-of-tree installation. More can be
found here: kubernetes.io/docs/tasks/administer-cluster/running-cloud-controller
Depending on which network plugin has been chosen there may be various pods to control network traffic. To handle DNS
queries, Kubernetes service discovery, and other functions the CoreDNS server has replaced kube-dns. Using chains of
plugins, one of many provided or custom written, the server is easily extensible.


Worker Nodes

• kubelet
• kube-proxy
• Docker engine, cri-o, containerd, etc.
• Network plugin pods
• Fluentd
• Prometheus

All nodes run the kubelet and kube-proxy, as well as the container engine such as containerd or cri-o, among several options. Other management daemons are deployed to watch these agents or provide services not yet included with Kubernetes.
The kubelet interacts with the underlying container engine also installed on all the nodes and makes sure that the containers
that need to run are actually running. The kube-proxy is in charge of managing the network connectivity to the containers.
It does so through the use of iptables entries. It also has userspace mode in which it monitors Services and Endpoints
using a random port to proxy traffic via ipvs. A network plugin pod, such as cilium-xxxxx, may be found depending on the
plugin in use.
Each node could run a a different engine. It is likely that Kubernetes will support additional container runtime engines.
Kubernetes does not have cluster-wide logging yet. Instead, another CNCF project is used, called Fluentd (http://www.fluentd.
org). When implemented it provides a unified logging layer for the cluster which filters, buffers and routes messages.
Cluster-wide metrics is another area with limited functionality the metrics-server SIG provides basic node and pod CPU and
memory utilization. For more metrics many use the Prometheus project.


4.1. KUBERNETES ARCHITECTURE

kubelet

• systemd process on each node
• Uses PodSpec
• Mounts volumes to Pod
• Downloads secrets
• Passes request to local container engine
• Reports status of Pods and node to cluster

The kubelet systemd process is the heavy-lifter for changes and configuration on worker nodes. It accepts the API calls for
Pod specifications. It will work to configure the local node until the specification has been met.
Should a Pod require access to storage, secrets or ConfigMaps the kubelet will ensure access or creation. It also sends
back status to the kube-apiserver for eventual persistence.
Kubelet calls other components such as the Topology Manager, which uses hints from other components to configure
topology aware resource NUMA assignments such as for CPU and hardware accelerators.


Operators
• Watch based control loop monitoring delta
• Informer / SharedInformer
• Workqueue
• Shipped operators
– Deployment operator
– replicaSet operator
– Service operator
– endpoints operator
– namespace operator
– serviceaccounts operator

An important concept for orchestration is the use of operators, otherwise known as controllers or watch-loops. Various operators ship with Kubernetes, and you can create your own as well. A simplified view of a operator is an agent, or Informer and
a downstream store. Using a DeltaFIFO queue, the source and downstream are compared. A loop process received an obj
or object, which is an array of deltas from the FIFO queue. As long as the delta is not of the type Deleted, the logic of the
operator is used to create or modify some object until it matches the spec.
The Informer which uses the API server as a source, requests the state of an object via API call. The data is cached to
minimize API server transactions. A similar agent is the SharedInformer; objects are often used by multiple other objects. It
creates a shared cache of the state for multiple requests.
A Workqueue uses a key to hand out tasks to various workers. The standard Go work queues of rate limiting, delayed, and
time queue are typically used.
The endpoints, namespace, and serviceaccounts controllers each manage the eponymous resources for Pods.
Deployments manage replicaSets which manage Pods running the same podSpec, or replicas.


4.1. KUBERNETES ARCHITECTURE

Service Operator

• Connect Pods together
• Expose Pods to Internet
• Decouple settings
• Define Pod access policy
• Operator monitoring a different operator

With every object and agent decoupled we need a flexible and scalable agent which connects resources together and will
reconnect should something die and a replacement is spawned. A service is an operator which listens to endpoint operator
to provide a persistent IP for Pods. Pods have ephemeral IP addresses chosen from a pool.
Then the service operator sends messages via the kube-apiserver which forwards settings to kube-proxy on every node
as well as the network plugin such as cilium-agent
A service also handles access policies for inbound requests, useful for resource control as well as for security.


Pods

• One or more containers
• Smallest unit to work with
• Only one, shared IP address per Pod

The whole point of Kubernetes is to orchestrate the life cycle of a container. We do not interact with particular containers.
Instead the smallest unit we can work with is a Pod. Some would say a pod of whales or peas-in-a-pod. Due to shared
resources, the design of a Pod typically follows a one process per container architecture.
Containers in a Pod are started in parallel. As a result, there is no way to determine which container becomes available
first inside a pod. The use of InitContainers can order startup, to some extent. To support a single process running in a
container, you may need logging, a proxy, or special adapter. These tasks are often handled by other containers in the same
pod.
There is only one IP address per Pod, for almost every network plugin. If there is more than one container in a pod, they must
share the IP. To communicate with each other they can either use IPC, the loopback interface, or a shared filesystem.
While Pods are often deployed with one application container in each, a common reason to have multiple containers in a Pod
is for logging. You may find the term sidecar for a container dedicated to performing a helper task, like handling logs and
responding to requests as the primary application container may have this ability. The term sidecar, like ambassador and
adapter, does not have a special setting but refers to the concept of what secondary pods are included to do.


4.1. KUBERNETES ARCHITECTURE

Containers

• Not worked with directly
• Usage limits passed to container engine
resources:
limits:
cpu: "1"
memory: "4Gi"
requests:
cpu: "0.5"
memory: "500Mi"

• ResourceQuota
• PriorityClass

While Kubernetes orchestration does not allow direct manipulation on a container level, we can manage the resources containers are allowed to consume.
In the resources section of the PodSpec you can pass parameters which will be passed to the container runtime on the
scheduled node.
Another way to manage resource usage of the containers is by creating a ResourceQuota object which allows hard and soft
limits to be set in a namespace. The quotas allow management of more resources than just CPU and memory and allows
limiting several objects.

scopeSelector field in the quota spec is used to run a pod at a specific priority if it has the appropriate priorityClassName
in its pod spec.


Init Containers

• Block app containers until precondition met
• Can contain code or utilities not in an app
• Independent security from app container
spec:
containers:
- name: main-app
image: databaseD
initContainers:
- name: wait-database
image: busybox
command: ['sh', '-c', 'until ls /db/dir ; do sleep 5; done; ']

Not all containers are the same. Standard containers are sent to the container engine at the same time, and may start in
any order. LivenessProbes, ReadinessProbes, and StatefulSets can be used to determine order but can add complexity.
Another option can be an Init Container which must complete before app containers will be started. Should the init container
fail it will be restarted until completion, without the app container running.
The init container can have a different view of the storage and security settings which allow utilities and commands to be used
which the application would not be allowed to use.
The code above will run the init container until the ls command succeeds, then will database container.


4.1. KUBERNETES ARCHITECTURE

Component Review

Figure 4.2: K8s Architectural Review

Now that we have seen some of the components, lets take another look with some of the connections shown. Not all
connections are shown in this diagram. Note that all of the components are communicating with kube-apiserver. Only
kube-apiserver communicates with the etcd database.
We also see some commands, which we may need to install separately, to work with various components. There is a etcdctl
command to interrogate the database and cilium to view more of how the network is configured.


Node

• Created outside of cluster
• NodeStatus
• NodeLease
• View resource usage with kubectl describe node

A node is an API object created outside of the cluster representing an instance. While a cp must be Linux, worker nodes
can also be Microsoft Windows Server 2019. Once the node has the necessary software installed it is ingested into the API
server.
At the moment you can create a cp node with kubeadm init and worker nodes by passing join. In the near future secondary
cp nodes and/or etcd nodes may be joined.
If the kube-apiserver cannot communicate with the kubelet on a node for five minutes, the default NodeLease, it will
schedule the node for deletion, and the NodeStatus will change from ready. The pods will be evicted once connection is
reestablished. They are no longer forcibly removed and rescheduled by the cluster.
Each node object exists in the kube-node-lease namespace. To remove a node from the cluster first use kubectl delete
node <node-name > to remove it from the API server. This will cause pods to be evacuated. Then use kubeadm reset to
remove cluster specific information. You may also need to remove iptables information, depending on if you plan on re-using
the node.
To view CPU, memory, and other resource usage, requests and limits use the kubectl describe node command. The output
will show capacity and pods allowed as well as details on current pods and resource utilization.


4.2. NETWORKING

4.2

Networking

Single IP per Pod

Figure 4.3: Pod Network

As well as a single IP address, a Pod represents a group of co-located containers with some associated data volumes. All
containers in a pod share the same network namespace.
The graphic shows a pod with two containers, A and B, and two data volumes, 1 and 2. Containers A and B share the network
namespace of a third container, known as the pause container. The pause container is used to get an IP address, then all
the containers in the pod will use its network namespace. Volumes 1 and 2 are shown for completeness.
To communicate with each other containers within pods can use the loopback interface, write to files on a common filesystem
or via inter-process communication (IPC). There is now a network plugin from HPE Labs which allows multiple IP addresses
per pod, but this feature has not grown past this new plugin.


Container to Outside Path

Figure 4.4: Container/Services Networking

This graphic shows a node with a single, dual-container pod. A NodePort service connects the Pod to the outside network.
Even though there are two containers they share the same namespace and the same IP address, which would be configured
by kubelet working with kube-proxy. The IP address is assigned before the containers are started and will be inserted into
the containers. The container will have an interface like eth0@tunl0. This IP is set for the life of the pod.
The endpoint is created at the same time as the service. Note that it uses the pod IP address, but also includes a port. The
service connects network traffic from a node high-number port to the endpoint using iptables with ipvs on the way. The
kube-controller-manager handles the watch loops to monitor the need for endpoints and services, as well as any updates
or deletions.


4.2. NETWORKING

Services

Figure 4.5: Services

We can use a service to connect one pod to another, or to outside of the cluster. This graphic shows a pod with a primary
container, App, with an optional sidecar Logger. Also seen is the pause container, which is used by the cluster to reserve the
IP address in the namespace prior to starting the other pods. This container is not seen from within Kubernetes, but can be
seen using docker and crictl.
This graphic shows a ClusterIP which is used to connect inside the cluster, not the IP of the cluster. As the graphic shows
this can be used to connect to a NodePort for outside the cluster, an IngressController or proxy, or another ”backend” pod
or pods.


Networking Setup

• Main networking challenges:
– Coupled container-to-container communications
– Pod-to-pod communications
– External-to-pod communications (solved by the services concept, to
be discussed later)
• Admin configuration required
• IP assigned to pod, not container

Getting all the previous components running is a common task for system administrators who are accustomed to configuration
management. But to get a fully functional Kubernetes cluster, the network will need to be setup properly as well.
A detailed explanation about the Kubernetes networking model can be seen at: https://kubernetes.io/docs/concepts/
cluster-administration/networking/.
If you have experience deploying virtual machines (VMs) based on IaaS solutions, this will sound familiar. The only caveat is
that in Kubernetes, the lowest compute unit is not a container, but what we call a pod.
A pod is a group of co-located containers that share the same IP address. From a networking perspective, a pod can be seen
as a virtual machine or a physical host. The network needs to assign IP addresses to pods, and needs to provide traffic routes
between all pods on any nodes. There are some network plugins which can allocate more than one IP to a pod, but most do
not.
The three main networking challenges to solve in a container orchestration system are:
• Coupled container-to-container communications which is solved by the pod concept.
• Pod-to-pod communications.
• External-to-pod communications which is solved by the services concept.
Kubernetes expects the network configuration to enable pod-to-pod communications to be available; it will not do it for you.
Tim Hockin, one of the lead Kubernetes developers, has created a very useful slide deck to understand Kubernetes networking.
You can check it out at: https://speakerdeck.com/thockin/illustrated-guide-to-kubernetes-networking.


4.2. NETWORKING

CNI Network Configuration File
{

}

"cniVersion": "1.3.0",
"name": "mynet",
"type": "bridge",
"bridge": "cni0",
"isGateway": true,
"ipMasq": true,
"ipam": {
"type": "host-local",
"subnet": "10.22.0.0/16",
"routes": [
{ "dst": "0.0.0.0/0" }
]
}

To provide container networking, Kubernetes is standardizing on the Container Network Interface (CNI) specification. Since
v1.6.0, kubeadm (the Kubernetes cluster bootstrapping tool) the goal has been to use CNI as the default network interface
mechanism. Various plugins can use CNI, but you may need to recompile to do so.
CNI is an emerging specification with associated libraries to write plugins that configure container networking, and remove
allocated resources when the container is deleted. Its aim is to provide a common interface between the various networking
solutions and container runtimes. As the CNI spec is language agnostic, there are many plugins from Amazon ECS to SR-IOV
to Cloud Foundry and many more.
This configuration defines a standard Linux bridge named cni0, which will give out IP addresses in the subnet
10.22.0.0./16. The bridge plugin will configure the network interfaces in the correct namespaces to define the container
network properly.
The main README of the CNI GitHub repository (https://github.com/containernetworking/cni) has more information.


Pod-to-Pod Communication

• All pods can communicate with each other across nodes.
• All nodes can communicate with all pods.
• No Network Address Translation (NAT).

While a CNI plugin can be used to configure the network of a pod and provide a single IP per pod, CNI does not help you with
pod-to-pod communications across nodes.
Basically, all IPs involved (nodes and pods) are routable without NAT. This can be achieved at the physical network infrastructure if you have access to it (e.g. GKE). Or this can be achieved with a software defined overlay with solutions like:

• Cilium
https://docs.cilium.io/en/stable/
• Flannel
https://github.com/flannel-io/flannel/
• Calico
https://www.tigera.io/project-calico/
• Multus
https://github.com/k8snetworkplumbingwg/multus-cni
See this documentation page or the list of networking add-ons for a more complete list.


4.3. OTHER CLUSTER SYSTEMS

4.3

Other Cluster Systems

Mesos

Figure 4.6: Mesos Architecture

At a high level, there is nothing different between Kubernetes and other clustering system.
A central manager exposes an API, a scheduler places the workloads on a set of nodes, and the state of the cluster is stored
in a persistent layer.
For example, you could compare Kubernetes with Mesos, and you would see the similarities. In Kubernetes, however, the
persistence layer is implemented with etcd, instead of Zookeeper for Mesos.
You should also consider systems like OpenStack and CloudStack. Think about what runs on their head node, and what
runs on their worker nodes. How do they keep state? How do they handle networking? If you are familiar with those systems,
Kubernetes will not seem that different.
What really sets Kubernetes apart is its features oriented towards fault-tolerance, self-discovery, and scaling, coupled with a
mindset that is purely API-driven.
Note: Mesos retired in August 2025 and the move to the Attic was completed in October 2025.


4.4

Labs


---


---

# Chapter 4: Kubernetes Architecture
**Working directory: `~/lfs458/ch04-architecture/`**

## Exercise 4.1: Basic Node Maintenance

```bash
cd ~/lfs458/ch04-architecture/
```

In this section we will backup the etcd database then update the version of Kubernetes used on control plane nodes and
worker nodes.

Backup The etcd Database
While the upgrade process has become stable, it remains a good idea to backup the cluster state prior to upgrading. There
are many tools available in the market to backup and manage etcd, each with a distinct backup and restore process. We will
use the included snapshot command, but be aware the exact steps to restore will depend on the tools used, the version of the
cluster, and the nature of the disaster being recovered from.
1. Find the data directory of the etcd daemon. All of the settings for the pod can be found in the manifest.
sudo grep data-dir /etc/kubernetes/manifests/etcd.yaml
- --data-dir=/var/lib/etcd

2. Log into the etcd container and look at the options etcdctl provides. Use tab to complete the container name, which
has the node name appended to it.
kubectl -n kube-system exec -it etcd-<Tab> -- sh

On Container
(a) View the arguments and options to the etcdctl command. Take a moment to view the options and arguments available. As the Bourne shell does not have may features it may be easier to copy/paste the
majority of the command and arguments after typing them out the first time.
# etcdctl -h
NAME:

etcdctl - A simple command line client for etcd3.

USAGE:

etcdctl [flags]
<output_omitted>

(b) In order to use TLS, find the three files that need to be passed with the etcdctl command. Change into
the directory and view available files. Newer versions of etcd image have been minimized. As a result
you may no longer have the find command, or really most commands. One must remember the URL
/etc/kubernetes/pki/etcd. As the ls command is also missing we can view the files using echo instead.
# cd /etc/kubernetes/pki/etcd
# echo *
ca.crt ca.key healthcheck-client.crt healthcheck-client.key
peer.crt peer.key server.crt server.key

(c) Typing out each of these keys, especially in a locked-down shell can be avoided by using an environmental
parameter. Log out of the shell and pass the various paths to the necessary files.


# exit

3. Check the health of the database using the loopback IP and port 2379. You will need to pass then peer cert and key as
well as the Certificate Authority as environmental variables. The command is commented, you do not need to type out
the comments or the backslashes.
kubectl -n kube-system exec -it etcd-cp -- sh \
#Same as before
-c "ETCDCTL_API=3 \
#Version to use
ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
#Pass the certificate authority
ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
#Pass the peer cert and key
ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
etcdctl endpoint health"
#The command to test the endpoint
https://127.0.0.1:2379 is healthy: successfully committed proposal: took = 11.942936ms

4. Determine how many databases are part of the cluster. Three and five are common in a production environment to
provide 50%+1 for quorum. In our current exercise environment we will only see one database. Remember you can use
up-arrow to return to the previous command and edit the command without having to type the whole command again.
kubectl -n kube-system exec -it etcd-cp -- sh \
-c "ETCDCTL_API=3 \
ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
etcdctl --endpoints=https://127.0.0.1:2379 member list"
fb50b7ddbf4930ba, started, cp, https://10.128.0.35:2380,
https://10.128.0.35:2379, false

5. You can also view the status of the cluster in a table format, among others passed with the -w option. Again, up-arrow
allows you to edit just the last part of the long string easily.
kubectl -n kube-system exec -it etcd-cp -- sh \
-c "ETCDCTL_API=3 \
ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
etcdctl --endpoints=https://127.0.0.1:2379 member list -w table"
+------------------+---------+-------+--------------------------+--------------------------+------------+
|
ID
| STATUS | NAME |
PEER ADDRS
|
CLIENT ADDRS
| IS
LEARNER |
+------------------+---------+-------+--------------------------+--------------------------+------------+
| 802d78549985d5a8 | started | cp | https://10.128.0.15:2380 | https://10.128.0.15:2379 |
false |
+------------------+---------+-------+--------------------------+--------------------------+------------+

6. Now that we know how many etcd databases are in the cluster, and their health, we can back it up. Use the snapshot
argument to save the snapshot into the container data directory/var/lib/etcd/
kubectl -n kube-system exec -it etcd-cp -- sh \
-c "ETCDCTL_API=3 \
ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \


etcdctl --endpoints=https://127.0.0.1:2379 snapshot save /var/lib/etcd/snapshot.db "
{"level":"info","ts":1598380941.6584022,"caller":"snapshot/v3_snapshot.go:110","
msg":"created temporary db file","path":"/var/lib/etcd/snapshot.db.part"}
{"level":"warn","ts":"2023-02-25T18:42:21.671Z","caller":"clientv3/retry_interceptor.go
:116","msg":"retry stream intercept"}
{"level":"info","ts":1598380941.6736135,"caller":"snapshot/v3_snapshot.go:121","
msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}
{"level":"info","ts":1598380941.7519674,"caller":"snapshot/v3_snapshot.go:134","
msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","took":0.093466104}
{"level":"info","ts":1598380941.7521122,"caller":"snapshot/v3_snapshot.go:143","
msg":"saved","path":"/var/lib/etcd/snapshot.db"}
Snapshot saved at /var/lib/etcd/snapshot.db

7. Verify the snapshot exists from the node perspective, the file date should have been moments earlier.
sudo ls -l /var/lib/etcd/
total 3888
drwx------ 4 root root
4096 Aug 25 11:22 member
-rw------- 1 root root 3973152 Aug 25 18:42 snapshot.db

8. Backup the snapshot as well as other information used to create the cluster both locally as well as another system in
case the node becomes unavailable. Remember to create snapshots on a regular basis, perhaps using a cronjob to
ensure a timely restore. When using the snapshot restore it’s important the database not be in use. An HA cluster
would remove and replace the control plane node, and not need a restore.
mkdir $HOME/backup
sudo cp /var/lib/etcd/snapshot.db $HOME/backup/snapshot.db-$(date +%m-%d-%y)
sudo cp /root/kubeadm-config.yaml $HOME/backup/
sudo cp -r /etc/kubernetes/pki/etcd $HOME/backup/

9. Any mistakes during restore may render the cluster unusable. Instead of issues, and having to rebuild the cluster, please
attempt a database restore after the final lab exercise of the course. More on the restore process can be found here:
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#restoring-an-etcd-cluster

Upgrade the Cluster
1. Begin by updating the package metadata for APT.
sudo apt update
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic InRelease
Get:2 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
Get:3 http://us-central1.gce.archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
Get:5 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
<output_omitted>

Introducing Kubernetes Community-Owned Package Repositories
Note: The legacy package repositories (apt.kubernetes.io and yum.kubernetes.io) have been deprecated and
frozen starting from September 13, 2023. Using the new package repositories hosted at pkgs.k8s.io is strongly
recommended and required in order to install Kubernetes versions released after September 13, 2023. The deprecated legacy repositories, and their contents, might be removed at any time in the future and without a further
notice period. The new package repositories provide downloads for Kubernetes versions starting with v1.24.0.


For further information please refer the URL https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/
2. Replace the apt repository definition so that apt points to the new repository instead of the Google-hosted repository.
Make sure to replace the Kubernetes minor version in the command below with the minor version that you are going to
upgrade.
sudo sed -i 's/33/34/g' /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update

3. View the available packages.
sudo apt-cache madison kubeadm
kubeadm | 1.34.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb
kubeadm | 1.34.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb
<output_omitted>

Packages
Packages

4. Remove the hold on kubeadm and update the package. Remember to update to the next major release’s update 1.
sudo apt-mark unhold kubeadm
Canceled hold on kubeadm.

sudo apt-get install -y kubeadm=1.34.1-1.1
Reading package lists... Done
Building dependency tree
Reading state information... Done
<output_omitted>

5. Hold the package again to prevent updates along with other software.
sudo apt-mark hold kubeadm
kubeadm set on hold.

6. Verify the version of Kubeadm. It should indicate the new version you just installed.
sudo kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"34", EmulationMajor:"", EmulationMinor:"",
MinCompatibilityMajor:"", MinCompatibilityMinor:"", GitVersion:"v1.34.1",
GitCommit:"93248f9ae092f571eb870b7664c534bfc7d00f03", GitTreeState:"clean",
BuildDate:"2025-09-09T19:43:15Z", GoVersion:"go1.24.6", Compiler:"gc", Platform:"linux/amd64"}

7. To prepare the cp node for update we first need to evict as many pods as possible. The nature of daemonsets is to have
them on every node, and some such as Cilium must remain. Change the node name to your node’s name, and add
the option to ignore the daemonsets.
kubectl drain cp --ignore-daemonsets


node/cp cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/cilium-5tv9d, kube-system/kube-proxy-8x9c5
evicting pod kube-system/coredns-5d78c9869d-z5ngb
evicting pod kube-system/cilium-operator-788c7d7585-wnb5b
evicting pod kube-system/coredns-5d78c9869d-4h2bs
pod/cilium-operator-788c7d7585-wnb5b evicted
pod/coredns-5d78c9869d-z5ngb evicted
pod/coredns-5d78c9869d-4h2bs evicted
node/cp drained

8. Use the upgrade plan argument to check the existing cluster and then update the software. You may notice that there
are versions available later than the .1 update in use. If you initialized the cluster in a previous lab using 1.30.1, use
upgrade to 1.31.1. Read through the output and get a feel for what would be changed in an upgrade.
sudo kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm
kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.33.1
[upgrade/versions] kubeadm version: v1.34.1
[upgrade/versions] Target version: v1.34.2
[upgrade/versions] Latest version in the v1.34 series: v1.34.1
<output_omitted>

9. We are now ready to actually upgrade the software. There will be a lot of output. Be aware the command will ask if you
want to proceed with the upgrade, answer y for yes. Take a moment and look for any errors or suggestions, such as
upgrading the version of etcd, or some other package. Again, Use the next minor release, update 1 from the version
used previous lab which initialized the cluster. The process will take several minutes to complete.
sudo kubeadm upgrade apply v1.34.1
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm
kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.34.1"
[upgrade/versions] Cluster version: v1.33.1
[upgrade/versions] kubeadm version: v1.34.1
[upgrade] Are you sure you want to proceed? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet
connection
[upgrade/prepull] You can also perform this action beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.34.1" (timeout:
5m0s)...
<output_omitted>

10. Check the status of the nodes. The cp should show scheduling disabled. Also as we have not updated all the software
and restarted the daemons it will show the previous version.
kubectl get node
NAME
cp


STATUS
Ready,SchedulingDisabled

ROLES
control-plane

AGE
109m

VERSION
v1.33.1


worker

Ready

<none>

61m

v1.33.1

11. Release the hold on kubelet and kubectl.
sudo apt-mark unhold kubelet kubectl
Canceled hold on kubelet.
Canceled hold on kubectl.

12. Upgrade both packages to the same version as kubeadm.
sudo apt-get install -y kubelet=1.34.1-1.1 kubectl=1.34.1-1.1
Reading package lists... Done
Building dependency tree
Reading state information... Done
<output_omitted>

13. Again add the hold so other updates don’t update the Kubernetes software.
sudo apt-mark hold kubelet kubectl
kubelet set on hold.
kubectl set on hold.

14. Restart the daemons.
sudo systemctl daemon-reload
sudo systemctl restart kubelet

15. Verify the cp node has been updated to the new version. Then update other cp nodes, if you should have them, using
the same process except sudo kubeadm upgrade node instead of sudo kubeadm upgrade apply.
kubectl get node
NAME
cp
worker

STATUS
Ready,SchedulingDisabled
Ready

ROLES
control-plane
<none>

AGE
113m
65m

VERSION
v1.34.1
v1.33.1

16. Now make the cp available for the scheduler, again change the name to match the cluster node name on your control
plane.
kubectl uncordon cp
node/cp uncordoned

17. Verify the cp now shows a Ready status.
kubectl get node


NAME
cp
worker

STATUS
Ready
Ready

ROLES
control-plane
<none>

AGE
114m
66m

VERSION
v1.34.1
v1.33.1

18. Now update the worker node(s) of the cluster. Open a second terminal session to the worker. Note that you will need
to run a couple commands on the cp as well, having two sessions open may be helpful. Begin by allowing the software
to update on the worker.
sudo apt-mark unhold kubeadm
Canceled hold on kubeadm.

Introducing Kubernetes Community-Owned Package Repositories
Note: The legacy package repositories (apt.kubernetes.io and yum.kubernetes.io) have been deprecated and
frozen starting from September 13, 2023. Using the new package repositories hosted at pkgs.k8s.io is strongly
recommended and required in order to install Kubernetes versions released after September 13, 2023. The
deprecated legacy repositories, and their contents, might be removed at any time in the future and without a
further notice period. The new package repositories provide downloads for Kubernetes versions starting with
v1.24.0. https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/
19. Replace the apt repository definition so that apt points to the new repository instead of the Google-hosted repository.
Make sure to replace the Kubernetes minor version in the command below with the minor version that you are going to
upgrade.
sudo sed -i 's/33/34/g' /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update

20. View the available packages.
sudo apt-cache madison kubeadm
kubeadm | 1.34.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb
kubeadm | 1.34.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb
kubeadm | 1.34.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb
<output_omitted>

Packages
Packages
Packages

kubeadm issue 3244
Due to an unresolved bug in kubeadm, we will use version 1.34.2 specifically for kubeadm. This applies only
to kubeadm, all other Kubernetes components can remain on the planned versions. More information about this
issue can be found at the following URL: https://github.com/kubernetes/kubeadm/issues/3244.
21. Update the kubeadm package to the same version as the cp node.
sudo apt-get update && sudo apt-get install -y kubeadm=1.34.2-1.1
<output_omitted>
Setting up kubeadm (1.34.2-1.1) ...

22. Hold the package again.
sudo apt-mark hold kubeadm


kubeadm set on hold.

23. Back on the cp terminal session drain the worker node, but allow the daemonsets to remain.
kubectl drain worker --ignore-daemonsets
node/worker cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/cilium-gzdk6, kube-system/kube-proxy-lpsmq
evicting pod kube-system/cilium-operator-788c7d7585-hc9wf
evicting pod kube-system/coredns-5d78c9869d-h4p7v
evicting pod kube-system/coredns-5d78c9869d-d4nv8
pod/cilium-operator-788c7d7585-hc9wf evicted
pod/coredns-5d78c9869d-h4p7v evicted
pod/coredns-5d78c9869d-d4nv8 evicted
node/worker drained

24. Return to the worker node and download the updated node configuration.
sudo kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm
kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.

25. Remove the hold on the software then update to the same version as set on the cp.
sudo apt-mark unhold kubelet kubectl
Canceled hold on kubelet.
Canceled hold on kubectl.

sudo apt-get install -y kubelet=1.34.1-1.1 kubectl=1.34.1-1.1
Reading package lists... Done
Building dependency tree
<output_omitted>
Setting up kubectl (1.34.1-1.1) ...
Setting up kubelet (1.34.1-1.1) ...

26. Ensure the packages don’t get updated when along with regular updates.
sudo apt-mark hold kubelet kubectl
kubelet set on hold.
kubectl set on hold.

27. Restart daemon processes for the software to take effect.


sudo systemctl daemon-reload
sudo systemctl restart kubelet

28. Return to the cp node. View the status of the nodes. Notice the worker status.

NAME
cp
worker

kubectl get node
STATUS
Ready
Ready,SchedulingDisabled

ROLES
control-plane
<none>

AGE
118m
70m

VERSION
v1.34.1
v1.34.1

29. Allow pods to be deployed to the worker node. Remember to use YOUR worker name. TAB can be helpful to enter the
name if command line completion is enabled.
kubectl uncordon worker
node/worker uncordoned

30. Verify the nodes both show a Ready status and the same upgraded version.
kubectl get nodes
NAME
cp
worker

STATUS
Ready
Ready

ROLES
control-plane
<none>

AGE
119m
71m

VERSION
v1.34.1
v1.34.1


---

## Exercise 4.2: Working with CPU and Memory Constraints

```bash
cd ~/lfs458/ch04-architecture/
```

Overview
We will continue working with our cluster, which we built in the previous lab. We will work with resource limits, more
with namespaces and then a complex deployment which you can explore to further understand the architecture and
relationships.
Use SSH or PuTTY to connect to the nodes you installed in the previous exercise. We will deploy an application called
stress inside a container, and then use resource limits to constrain the resources the application has access to
use.
1. Use a container called stress, in a deployment which we will name hog, to generate load. Verify you have the container
running.
kubectl create deployment hog --image vish/stress
deployment.apps/hog created

kubectl get deployments
NAME
hog

READY
1/1

UP-TO-DATE

AVAILABLE

AGE
13s

2. Use the describe argument to view details, then view the output in YAML format. Note there are no settings limiting
resource usage. Instead, there are empty curly brackets.
kubectl describe deployment hog


Name:
Namespace:
Labels:
Annotations:
<output_omitted>

hog
default
app=hog
deployment.kubernetes.io/revision: 1

kubectl get deployment hog -o yaml
apiVersion: apps/v1
kind: Deployment
Metadata:
<output_omitted>
template:
metadata:
creationTimestamp: null
labels:
app: hog
spec:
containers:
- image: vish/stress
imagePullPolicy: Always
name: stress
resources: {}
terminationMessagePath: /dev/termination-log
<output_omitted>

3. We will use the YAML output to create our own configuration file.
kubectl get deployment hog -o yaml > hog.yaml

4. Probably good to remove the status output, creationTimestamp and other settings. We will also add in memory limits
found below.
vim hog.yaml

hog.yaml

....

3
5
7
9
11

....

imagePullPolicy: Always
name: hog
resources:
# Edit to remove {}
limits:
# Add these 4 lines
memory: "4Gi"
requests:
memory: "2500Mi"
terminationMessagePath: /dev/termination-log
terminationMessagePolicy: File

5. Replace the deployment using the newly edited file.
kubectl replace -f hog.yaml
deployment.apps/hog replaced


6. Verify the change has been made. The deployment should now show resource limits.
kubectl get deployment hog -o yaml
....

....

resources:
limits:
memory: 4Gi
requests:
memory: 2500Mi
terminationMessagePath: /dev/termination-log

7. View the stdio of the hog container. Note how much memory has been allocated.
kubectl get po
NAME
hog-64cbfcc7cf-lwq66

READY
1/1

STATUS
Running

RESTARTS

AGE
2m

kubectl logs hog-64cbfcc7cf-lwq66
I1102 16:16:42.638972
1 main.go:26] Allocating "0" memory, in
"4Ki" chunks, with a 1ms sleep between allocations
I1102 16:16:42.639064
1 main.go:29] Allocated "0" memory

8. Open a second and third terminal to access both cp and second nodes. Run top to view resource usage. You should not
see unusual resource usage at this point. The containerd and top processes should be using about the same amount
of resources. The stress command should not be using enough resources to show up. Use the kubectl get events to
see if any pods are evicted when too many resources are in use.
9. Edit the hog configuration file and add arguments for stress to consume CPU and memory. The args: entry should be
indented the same number of spaces as resources:.
vim hog.yaml

hog.yaml

....

3
5
7
9
11
13
15
17

....


resources:
limits:
cpu: "1"
memory: "4Gi"
requests:
cpu: "0.5"
memory: "500Mi"
args:
- -cpus
- "2"
- -mem-total
- "950Mi"
- -mem-alloc-size
- "100Mi"
- -mem-alloc-sleep
- "1s"


10. Delete and recreate the deployment. You should see increased CPU usage almost immediately and memory allocation
happen in 100M chunks, allocated to the stress program via the running top command. Check both nodes as the
container could deployed to either. Be aware that nodes with a small amount of memory or CPU may encounter issues.
Symptoms include cp node infrastructure pods failing. Adjust the amount of resources used to allow standard pods to
run without error.
kubectl delete deployment hog
deployment.apps "hog" deleted

kubectl create -f hog.yaml
deployment.apps/hog created

Only if top does not show high usage
Should the resources not show increased use, there may have been an issue inside of the container. Kubernetes
may show it as running, but the actual workload has failed. Or the container may have failed; for example if you
were missing a parameter the container may panic.
kubectl get pod
NAME
hog-1985182137-5bz2w

READY
0/1

STATUS
Error

RESTARTS

AGE
5s

kubectl logs hog-1985182137-5bz2w
panic: cannot parse '150mi': unable to parse quantity's suffix
goroutine 1 [running]:
panic(0x5ff9a0, 0xc820014cb0)
/usr/local/go/src/runtime/panic.go:481 +0x3e6
k8s.io/kubernetes/pkg/api/resource.MustParse(0x7ffe460c0e69, 0x5, 0x0, 0x0, 0x0, 0x0, 0x0,
0x0, 0x0)
/usr/local/google/home/vishnuk/go/src/k8s.io/kubernetes/pkg/api/resource/quantity.go:134
+0x287
main.main()
/usr/local/google/home/vishnuk/go/src/github.com/vishh/stress/main.go:24 +0x43

Here is an example of an improper parameter. The container is running, but not allocating memory. It should
show the usage requested from the YAML file.
kubectl get po
NAME
hog-1603763060-x3vnn

READY
1/1

STATUS
Running

RESTARTS

AGE
8s

kubectl logs hog-1603763060-x3vnn


I0927 21:09:23.514921
sleep \

1 main.go:26] Allocating "0" memory, in "4ki" chunks, with a 1ms
between allocations
1 main.go:39] Spawning a thread to consume CPU
1 main.go:39] Spawning a thread to consume CPU
1 main.go:29] Allocated "0" memory

I0927 21:09:23.514984
I0927 21:09:23.514991
I0927 21:09:23.514997


---

## Exercise 4.3: Resource Limits for a Namespace

```bash
cd ~/lfs458/ch04-architecture/
```

The previous steps set limits for that particular deployment. You can also set limits on an entire namespace. We will
create a new namespace and configure another hog deployment to run within. When set hog should not be able to use
the previous amount of resources.
1. Begin by creating a new namespace called low-usage-limit and verify it exists.
kubectl create namespace low-usage-limit
namespace/low-usage-limit created

kubectl get namespace
NAME
default
kube-node-lease
kube-public
kube-system
low-usage-limit

STATUS
Active
Active
Active
Active
Active

AGE
1h
1h
1h
1h
42s

2. Create a YAML file which limits CPU and memory usage. The kind to use is LimitRange. Remember the file may be
found in the example tarball.
cp /home/student/LFS458/SOLUTIONS/s_04/low-resource-range.yaml .
vim low-resource-range.yaml

low-resource-range.yaml
2
4
6
8
10
12

apiVersion: v1
kind: LimitRange
metadata:
name: low-resource-range
spec:
limits:
- default:
cpu: 1
memory: 500Mi
defaultRequest:
cpu: 0.5
memory: 100Mi


type: Container

3. Create the LimitRange object and assign it to the newly created namespace low-usage-limit. You can use --namespace
or -n to declare the namespace.
kubectl create -f low-resource-range.yaml -n low-usage-limit
limitrange/low-resource-range created

4. Verify it works. Remember that every command needs a namespace and context to work. Defaults are used if not
provided.
kubectl get LimitRange
No resources found in default namespace.

kubectl get LimitRange --all-namespaces
NAMESPACE
low-usage-limit

NAME
low-resource-range

CREATED AT
2024-06-23T10:23:57Z

5. Create a new deployment in the namespace.
kubectl -n low-usage-limit \
create deployment limited-hog --image vish/stress
deployment.apps/limited-hog created

6. List the current deployments. Note hog continues to run in the default namespace. If you chose to use the Cilium
network policy you may see a couple more than what is listed below.
kubectl get deployments --all-namespaces
NAMESPACE
default
kube-system
kube-system
low-usage-limit

NAME
hog
cilium-operator
coredns
limited-hog

READY
1/1
1/1
2/2
1/1

UP-TO-DATE
1
1

AVAILABLE
1
1

AGE
7m57s
2d10h
2d10h
9s

7. View all pods within the namespace. Remember you can use the tab key to complete the namespace. You may want to
type the namespace first so that tab-completion is appropriate to that namespace instead of the default namespace.
kubectl -n low-usage-limit get pods
NAME
limited-hog-2556092078-wnpnv

READY
1/1

STATUS
Running

RESTARTS

AGE
2m11s

8. Look at the details of the pod. You will note it has the settings inherited from the entire namespace. The use of shell
completion should work if you declare the namespace first.


kubectl -n low-usage-limit \
get pod limited-hog-2556092078-wnpnv -o yaml
<output_omitted>
spec:
containers:
- image: vish/stress
imagePullPolicy: Always
name: stress
resources:
limits:
cpu: "1"
memory: 500Mi
requests:
cpu: 500m
memory: 100Mi
terminationMessagePath: /dev/termination-log
<output_omitted>

9. Copy and edit the config file for the original hog file. Add the namespace: line so that a new deployment would be in the
low-usage-limit namespace. Delete the selflink line, if it exists.
cp hog.yaml hog2.yaml
vim hog2.yaml

hog2.yaml
2
4
6
8

....
labels:
app: hog
name: hog
namespace: low-usage-limit
#<<--- Add this line, delete following
selfLink: /apis/apps/v1/namespaces/default/deployments/hog
spec:
....

10. Open up extra terminal sessions so you can have top running in each. When the new deployment is created it will
probably be scheduled on the node not yet under any stress.
Create the deployment.
kubectl create -f hog2.yaml
deployment.apps/hog created

11. View the deployments. Note there are two with the same name, hog but in different namespaces. You may also find the
cilium deployment has no pods, nor has any requested. Our small cluster does not need to add Cilium pods via this
autoscaler.
kubectl get deployments --all-namespaces
NAMESPACE
default
kube-system
kube-system
low-usage-limit
low-usage-limit


NAME
hog
cilium-operator
coredns
hog
limited-hog

READY
1/1
1/1
2/2
1/1
1/1

UP-TO-DATE
0
1

AVAILABLE
0
1


AGE
24m
4h
4h
26s
5m11s


12. Look at the top output running in other terminals. You should find that both hog deployments are using about the
same amount of resources, once the memory is fully allocated. Per-deployment settings override the global namespace
settings. You should see something like the following lines one from each node, which indicates use of one processor
and about 12 percent of your memory, were you on a system with 8G total.
25128 root
24875 root

20

0

958532 954672
958532 954800

3180 R 100.0 11.7
3180 R 100.3 11.7

0:52.27 stress
41:04.97 stress

13. Delete the hog deployments to recover system resources.
kubectl -n low-usage-limit delete deployment hog limited-hog
deployment.apps "hog" deleted
deployment.apps "limited-hog" deleted

kubectl delete deployment hog
deployment.apps "hog" deleted


5.1

API Access . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 100

5.2

Annotations . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 105

5.3

Working with A Simple Pod

5.4

kubectl and API . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 107

5.5

Swagger and OpenAPI . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 114

5.6

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 106

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 116


5.1


API Access

API Access

• API driven architecture
• API groups
• RESTful style
• Standard HTTP verbs
• Deprecation process now being honored

Kubernetes has a powerful REST-based API. The entire architecture is API driven. Knowing where to find resource endpoints
and understanding how the API changes between versions can be important to ongoing administrative tasks, as there is much
ongoing change and growth. Starting with v1.16 deprecated objects are no longer honored by the API server.
As we learned in the Architecture chapter, the main agent for communication between cluster agents and from outside the
cluster is the kube-apiserver. A curl query to the agent will expose the current API groups. Groups may have multiple
versions which evolve independently of other groups, and follow a domain-name format with several names reserved, such as
single-word domains, the empty group and any name ending in .k8s.io.


5.1. API ACCESS

RESTful

• Responds to typical HTTP verbs (GET, POST, DELETE ...)
• Allows easy interaction with other ecosystems
• Scripting and interaction with deployment tools
• User impersonation headers

kubectl makes API calls on your behalf. You can also make calls externally, using curl or other program. With the appropriate
certs and keys, you can make requests or pass json files to make configuration changes.
$ curl --cert userbob.pem --key userBob-key.pem \
--cacert /path/to/ca.pem \
https://k8sServer:6443/api/v1/pods

The ability to impersonate other users or groups, subject to RBAC configuration, allows a manual override authentication. This
can be helpful for debugging authorization policies of other users.


Checking Access

• Several ways to authenticate
• auth can-i subcommand to query authorization
• Accepts can-i and reconcile arguments
• More in Security chapter to follow.


5.1. API ACCESS

While there is more detail on security in a later chapter, it is helpful to check the current authorizations, both as an admin, as
well as another user. The following shows what user bob could do in the default namespace and the developer namespace:
$ kubectl auth can-i create deployments
yes

$ kubectl auth can-i create deployments --as bob
no

$ kubectl auth can-i create deployments --as bob --namespace developer
yes

There are currently three APIs which can be applied to set who and what can be queried.
• SelfSubjectAccessReview
Access review for any user, helpful for delegating to others.
• LocalSubjectAccessReview
Review is restricted to a specific namespace.
• SelfSubjectRulesReview
A review which shows allowed actions for a user within a particular namespace.
The use of reconcile allows a check of authorization necessary to create an object from a file. No output indicates the
creation would be allowed.


Optimistic Concurrency

• Currently leverage JSON
• resourceVersion
• Clients must handle 409 CONFLICT Errors

The default serialization for API calls must be JSON. There is an effort to use Google’s protobuf serialization, but this remains
experimental. While we may work with files in YAML format, they are converted to and from JSON.
Kubernetes uses the resourceVersion value to determine API updates and implement optimistic concurrency. In other
words, an object is not locked from the time it has been read until the object is written.
Instead, upon an updated call to an object, the resourceVersion is checked and a 409 CONFLICT is returned should the
number have changed. The resourceVersion is currently backed via the modifedIndex parameter in the etcd database,
and is unique to the namespace, kind and server. Operations which do not change an object such as WATCH or GET do not
update this value.


5.2. ANNOTATIONS

5.2

Annotations

Using Annotations

• Distinct from Labels
• Non-identifying metadata
• Key/value maps
• Metadata otherwise held in exterior databases
• Useful for third-party automation

Labels are used to work with objects or collections of objects; annotations are not.
Instead annotations allow for metadata to be included with an object that may be helpful outside of Kubernetes object
interaction. Similar to labels, they are key to value maps. They also are able to hold more information, and more human
readable information than labels.
Having this kind of metadata can be used to track information such as a timestamp, pointers to related objects from other
ecosystems, or even an email from the developer responsible for that object’s creation.
The annotation data could otherwise be held in an exterior database, but that would limit the flexibility of the data. The more
this metadata is included, the easier to integrate management and deployment tools or shared client libraries.
For example, to annotate only Pods within a namespace, then overwrite the annotation and finally delete it:
$ kubectl annotate pods --all description='Production Pods' -n prod
$ kubectl annotate --overwrite pod webpod description="Old Production Pods" -n prod
$ kubectl -n prod annotate pod webpod description-


5.3

Working with A Simple Pod

Simple Pod
• Lowest compute unit of K8s
• Typically multiple containers grouped together
• Created from PodSpec
• Few required, many optional
– apiVersion
Must match existing API group
– kind
The type of object to create
– metadata
At least a name
– spec
What to create and parameters

As discussed earlier, a Pod is the lowest compute unit and individual object we can work with in Kubernetes. It can be a single
container, but often it will consist of a primary application container and one or more supporting containers.
Below is an example of a simple pod manifest in YAML format. You can see the apiVersion, the kind, the metadata, and its
spec, which define the container that actually runs in this pod:

2
4
6
8

apiVersion: v1
kind: Pod
metadata:
name: firstpod
spec:
containers:
- image: nginx
name: stan

You can use the kubectl create command to create this pod in Kubernetes. Once it is created, you can check its status
with kubectl get pods. Output is omitted to save space:
$ kubectl create -f simple.yaml
$ kubectl get pods
$ kubectl get pod firstpod -o yaml
$ kubectl get pod firstpod -o json


5.4. KUBECTL AND API

5.4

kubectl and API

Manage API Resources with kubectl

• API exposed via RESTful interface
• Use curl to access and test
• Use verbose mode
• Leverages HTTP verbs

Kubernetes exposes resources via RESTful API calls, which allows all resources to be managed via HTTP, JSON or even XML. the typical
protocol being HTTP. The state of the resources can be changed using standard HTTP verbs (e.g. GET, POST, PATCH, DELETE, etc.).
kubectl has a verbose mode argument which shows details from where the command gets and updates information. Other output includes
curl commands you could use to obtain the same result. While the verbosity accepts levels from zero to any number, there is currently no
verbosity value greater than ten. You can check this out for kubectl get. The output below has been formatted for clarity:

$ kubectl --v=10 get pods firstpod
....
I1215 17:46:47.860958
29909 round_trippers.go:417]
curl -k -v -XGET -H "Accept: application/json"
-H "User-Agent: kubectl/v1.8.5 (linux/amd64) kubernetes/cce11c6"
https://10.128.0.3:6443/api/v1/namespaces/default/pods/firstpod
....


Access From Outside The Cluster

• Can use curl from outside cluster
• Must use SSL/TLS for secure access
• Information found in ~/.kube/config
• View server information via kubectl config view

The primary tool used from the command line will be kubectl, which calls curl on your behalf. You can also use the curl
command from outside the cluster to view or make changes.
The basic server information, with redacted TLS certificate information can be found in the output of
$ kubectl config view

If you view verbose output from a previous page, you will note that the first line references a config file where this information
is pulled from, ~/.kube/config .
I1215 17:35:46.725407
27695 loader.go:357]
Config loaded from file /home/student/.kube/config

Without the certificate authority, key and certificate from this file, only insecure curl commands can be used, which will not
expose much due to security settings. We will use curl to access our cluster using TLS in an upcoming lab.


5.4. KUBECTL AND API

~/.kube/config
apiVersion: v1
clusters:
- cluster:
certificate-authority-data: LS0tLS1CRUdF.....
server: https://10.128.0.3:6443
name: kubernetes
contexts:
- context:
cluster: kubernetes
user: kubernetes-admin
name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
user:
client-certificate-data: LS0tLS1CRUdJTib.....
client-key-data: LS0tLS1CRUdJTi....

The output above shows 19 lines of output with each of the keys being heavily truncated. While the keys may look similar close examination
shows them to be distinct.
• apiVersion
As with other objects, this instructs the kube-apiserver where to assign the data.
• clusters
This contains the name of the cluster as well as where to send the API calls. The certificate-authority-data is passed to
authenticate the curl request.
• contexts
A setting which allows easy access to multiple clusters, possibly as various users, from one config file. It can be used to set
namespace, user, and cluster.
• current-context
Shows which cluster and user kubectl would use. These settings can also be passed on a per-command basis.
• kind
Every object within Kubernetes must have this setting, in this case a declaration of object type Config.
• preferences
Currently not used optional settings for the kubectl command, such as colorizing output.
• users
A nickname associated with client credentials which can be client key and certificate, username and password, and a token. Token
and username/password are mutually exclusive. These can be configured via the kubectl config set-credentials command.


Namespaces
• Linux kernel feature
– Segregates system resources
– Core functionality of containers
• API Object
– Four namespaces to begin with:
* default
* kube-node-lease
* kube-public
* kube-system
– - -all-namespaces

The term namespace is used to both reference the Kernel feature as well as the segregation of API objects by Kubernetes.
Both are means to keep resources distinct.
Every API call includes a namespace, using default if not otherwise declared: https://10.128.0.3:6443/api/v1/namespaces/
default/pods.
Namespaces are intended to isolate multiple groups and the resources they have access to work with via quotas. Eventually, access control policies will work on namespace boundaries as well. One could use Labels to group resources for
administrative reasons.
There are four namespaces when a cluster is first created.
• default
This is where all resources are assumed unless set otherwise.
• kube-node-lease The namespace where worker node lease information is kept.
• kube-public
A namespace readable by all, even those not authenticated. General information is often included in this namespaces.
• kube-system
Contains infrastructure pods.
Should you want to see all resources on a system you must pass the --all-namespaces option to the kubectl command.


5.4. KUBECTL AND API

Working with Namespaces

$ kubectl get ns
$ kubectl create ns linuxcon
$ kubectl describe ns linuxcon
$ kubectl get ns/linuxcon -o yaml
$ kubectl delete ns/linuxcon

The above commands show how to view, create and delete namespaces. Note that the describe subcommand shows several
settings such as Labels, Annotations, resource quotas, and resource limits which we will discus later in the course.
Once a namespace has been created you can reference via YAML when creating resource:
$ cat redis.yaml

redis.yaml
2
4
6

apiVersion: V1
kind: Pod
metadata:
name: redis
namespace: linuxcon
...


API Resources with kubectl

• All available via kubectl
• kubectl [command] [type] [Name] [flag]
• kubectl help for more information
• Abbreviated names

All API resources exposed are available via kubectl. Expect the list to change.
• all

• endpoints (ep)

• pods (po)

• certificatesigningrequests (csr)

• events (ev)

• podsecuritypolicies (psp)

• clusterrolebindings

• horizontalpodautoscalers (hpa)

• podtemplates

• clusterroles

• ingresses (ing)

• replicasets (rs)

• clusters (valid only for federation
apiservers)

• jobs

• replicationcontrollers rc)

• limitranges (limits)

• resourcequotas (quota)

• componentstatuses (cs)

• namespaces (ns)

• rolebindings

• configmaps (cm)

• networkpolicies (netpol)

• roles

• controllerrevisions

• nodes (no)

• secrets

• cronjobs

• persistentvolumeclaims (pvc)

• serviceaccounts (sa)

• customresourcedefinition (crd)

• persistentvolumes (pv)

• services (svc)

• daemonsets (ds)

• poddisruptionbudgets (pdb)

• statefulsets

• deployments (deploy)

• podpreset

• storageclasses


5.4. KUBECTL AND API

Additional Resource Methods

• Various Endpoints
• CLI --help
• Online documentation

In addition to basic resource management via REST, the API also provides some extremely useful endpoints for certain
resources.
For example, you can access the logs of a container, exec into it, and watch changes to it with the following endpoints:
$ curl --cert /tmp/client.pem --key /tmp/client-key.pem \
--cacert /tmp/ca.pem -v -XGET \
https://10.128.0.3:6443/api/v1/namespaces/default/pods/firstpod/log

This would be the same as the following. If the container does not have any standard out, there would be no logs.
$ kubectl logs firstpod

Other calls you could make, following the various API groups on your cluster:
GET /api/v1/namespaces/{namespace}/pods/{name}/exec
GET /api/v1/namespaces/{namespace}/pods/{name}/log
GET /api/v1/watch/namespaces/{namespace}/pods/{name}


5.5


Swagger and OpenAPI

Swagger

Figure 5.1: Swagger Screenshot

The entire Kubernetes API was built using a Swagger specification. This has been evolving towards the OpenAPI initiative. It
is extremely useful, as it allows, for example, to auto-generate client code. All the stable resources definitions are available on
the documentation site.
You can browse some of the API groups via a Swagger UI at https://swagger.io/specification/.


5.5. SWAGGER AND OPENAPI

API Maturity

• Versioning of API levels for easier growth
• Not directly tied to software versioning
• Versions imply level of support
– Alpha
– Beta
– Stable

The use of API groups and different versions allows for development to advance without changes to an existing group of APIs.
This allows for easier growth and separation of work among separate teams. While there is an attempt to maintain some
consistency between API and software versions they are only indirectly linked.
The use of JSON and Google’s Protobuf serialization scheme will follow the same release guidelines.
An Alpha level release, noted with alpha in the names, may be buggy and is disabled by default. Features could change or
disappear at any time, and backward compatibility is not guaranteed. Only use these features on a test cluster which can often
be rebuilt.
The Beta levels, found with beta in the names, has more well tested code and is enabled by default. It also ensures that as
changes move forward they will be tested for backwards compatibility between versions. It has not been adopted and tested
enough to be called stable. Expect some bugs and issues.
Use of the Stable version, denoted by only an integer which may be preceded by the letter v, is for stable APIs. At the moment
v1 is the only stable version.


5.6


Labs


---


---

# Chapter 5: APIs and Access
**Working directory: `~/lfs458/ch05-api-access/`**

## Exercise 5.1: Configuring TLS Access

```bash
cd ~/lfs458/ch05-api-access/
```

Overview
Using the Kubernetes API, kubectl makes API calls for you. With the appropriate TLS keys you could run curl as well
use a golang client. Calls to the kube-apiserver get or set a PodSpec, or desired state. If the request represents
a new state the Kubernetes Control Plane will update the cluster until the current state matches the specified state.
Some end states may require multiple requests. For example, to delete a ReplicaSet, you would first set the number
of replicas to zero, then delete the ReplicaSet.
An API request must pass information as JSON. kubectl converts .yaml to JSON when making an API request on
your behalf. The API request has many settings, but must include apiVersion, kind and metadata, and spec settings
to declare what kind of container to deploy. The spec fields depend on the object being created.
We will begin by configuring remote access to the kube-apiserver then explore more of the API.
1. Begin by reviewing the kubectl configuration file. We will use the three certificates and the API server address.
less $HOME/.kube/config
<output_omitted>

2. We will create a variables using certificate information. You may want to double-check each parameter as you set it.
Begin with setting the client-certificate-data key.
export client=$(grep client-cert $HOME/.kube/config |cut -d" " -f 6)
echo $client
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQWRxZ0F3SUJ
BZ0lJRy9wbC9rWEpNdmd3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0
ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB4TnpFeU1UTXhOelEyTXpKY
UZ3MHhPREV5TVRNeE56UTJNelJhTURReApGekFWQmdOVkJBb1REbk41YzNS
<output_omitted>

3. Almost the same command, but this time collect the client-key-data as the key variable.
export key=$(grep client-key-data $HOME/.kube/config |cut -d " " -f 6)
echo $key
<output_omitted>

4. Finally set the auth variable with the certificate-authority-data key.
export auth=$(grep certificate-authority-data $HOME/.kube/config |cut -d " " -f 6)
echo $auth
<output_omitted>

5. Now encode the keys for use with curl.


echo $client | base64 -d - > ./client.pem
echo $key | base64 -d - > ./client-key.pem
echo $auth | base64 -d - > ./ca.pem

6. Pull the API server URL from the config file. Your hostname or IP address may be different.
kubectl config view |grep server
server: https://k8scp:6443

7. Use curl command and the encoded keys to connect to the API server. Use your hostname, or IP, found in the previous
command, which may be different than the example below.
curl --cert ./client.pem \
--key ./client-key.pem \
--cacert ./ca.pem \
https://k8scp:6443/api/v1/pods
{

"kind": "PodList",
"apiVersion": "v1",
"metadata": {
"selfLink": "/api/v1/pods",
"resourceVersion": "239414"
},
<output_omitted>

8. If the previous command was successful, create a JSON file to create a new pod. Remember to use find and search
for this file in the tarball output, it can save you some typing.
cp /home/student/LFS458/SOLUTIONS/s_05/curlpod.json .
vim curlpod.json
{

}

"kind": "Pod",
"apiVersion": "v1",
"metadata":{
"name": "curlpod",
"namespace": "default",
"labels": {
"name": "examplepod"
}
},
"spec": {
"containers": [{
"name": "nginx",
"image": "nginx",
"ports": [{"containerPort": 80}]
}]
}

9. The previous curl command can be used to build a XPOST API call. There will be a lot of output, including the scheduler
and taints involved. Read through the output. In the last few lines the phase will probably show Pending, as it’s near the
beginning of the creation process.


curl --cert ./client.pem \
--key ./client-key.pem --cacert ./ca.pem \
https://k8scp:6443/api/v1/namespaces/default/pods \
-XPOST -H'Content-Type: application/json' \
-d@curlpod.json
{

"kind": "Pod",
"apiVersion": "v1",
"metadata": {
"name": "curlpod",
<output_omitted>

10. Verify the new pod exists and shows a Running status.
kubectl get pods
NAME
curlpod

READY
1/1

STATUS
Running

RESTARTS

AGE
45s


---

## Exercise 5.2: Explore API Calls

```bash
cd ~/lfs458/ch05-api-access/
```

1. One way to view what a command does on your behalf is to use strace. In this case, we will look for the current
endpoints, or targets of our API calls. Install the tool, if not present.
sudo apt-get install -y strace
kubectl get endpoints
NAME
kubernetes

ENDPOINTS
10.128.0.3:6443

AGE
3h

2. Run this command again, preceded by strace. You will get a lot of output. Near the end you will note several openat
functions to a local directory, /home/student/.kube/cache/discovery/k8scp 6443. If you cannot find the lines, you
may want to redirect all output to a file and grep for them. This information is cached, so you may see some differences
should you run the command multiple times. As well your IP address may be different.
strace kubectl get endpoints
execve("/usr/bin/kubectl", ["kubectl", "get", "endpoints"], [/*....
....
openat(AT_FDCWD, "/home/student/.kube/cache/discovery/k8scp_6443..
<output_omitted>

3. Change to the parent directory and explore. Your endpoint IP will be different, so replace the following with one suited
to your system.
cd /home/student/.kube/cache/discovery/
ls
k8scp_6443

cd k8scp_6443/

4. View the contents. You will find there are directories with various configuration information for kubernetes.
ls


admissionregistration.k8s.io
apiextensions.k8s.io
apiregistration.k8s.io
apps
authentication.k8s.io
authorization.k8s.io
autoscaling
batch

certificates.k8s.io
coordination.k8s.io
cilium.io
discovery.k8s.io
events.k8s.io
extensions
flowcontrol.apiserver.k8s.io
networking.k8s.io

node.k8s.io
policy
rbac.authorization.k8s.io
scheduling.k8s.io
servergroups.json
storage.k8s.io
v1

5. Use the find command to list out the subfiles. The prompt has been modified to look better on this page.
find .
.
./storage.k8s.io
./storage.k8s.io/v1beta1
./storage.k8s.io/v1beta1/serverresources.json
./storage.k8s.io/v1
./storage.k8s.io/v1/serverresources.json
./rbac.authorization.k8s.io
<output_omitted>

6. View the objects available in version 1 of the API. For each object, or kind:, you can view the verbs or actions for that
object, such as create seen in the following example. Note the prompt has been truncated for the command to fit on one
line. Some are HTTP verbs, such as GET, others are product specific options, not standard HTTP verbs. The command
may be python, depending on what version is installed.
python3 -m json.tool v1/serverresources.json

serverresources.json
2
4
6
8
10
12
14

{

"apiVersion": "v1",
"groupVersion": "v1",
"kind": "APIResourceList",
"resources": [
{
"kind": "Binding",
"name": "bindings",
"namespaced": true,
"singularName": "",
"verbs": [
"create"
]
},
<output_omitted>

7. Some of the objects have shortNames, which makes using them on the command line much easier. Locate the
shortName for endpoints.
python3 -m json.tool v1/serverresources.json | less


serverresources.json
2
4
6
8
10
12

....
{
"kind": "Endpoints",
"name": "endpoints",
"namespaced": true,
"shortNames": [
"ep"
],
"singularName": "",
"verbs": [
"create",
"delete",
....

8. Use the shortName to view the endpoints. It should match the output from the previous command.
kubectl get EndpointSlice
NAME
kubernetes

ADDRESSTYPE
IPv4

PORTS
6443

ENDPOINTS
10.2.0.31

AGE
3h

9. We can see there are 37 objects in version 1 file.
python3 -m json.tool v1/serverresources.json | grep kind
"kind": "APIResourceList",
"kind": "Binding",
"kind": "ComponentStatus",
"kind": "ConfigMap",
"kind": "Endpoints",
"kind": "Event",
<output_omitted>

10. Looking at another file we find nine more.
python3 -m json.tool apps/v1/serverresources.json | grep kind
"kind": "APIResourceList",
"kind": "ControllerRevision",
"kind": "DaemonSet",
"kind": "DaemonSet",
"kind": "Deployment",
<output_omitted>

11. Delete the curlpod to recoup system resources.
kubectl delete po curlpod
pod "curlpod" deleted

12. Take a look around the other files in this directory as time permits.


6.1

API Objects . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 122

6.2

The v1 Group . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 123

6.3

API Resources . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 125

6.4

RBAC APIs . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 130

6.5

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 131


6.1


API Objects

Overview
• Ongoing growth in API objects
• Track release notes to find new objects
• In this chapter we will introduce common API objects:
– Deployment the typical object used
– DaemonSets, ReplicaSets now apps/v1
– StatefulSets (once called PetSets) part of apps/v1 since v1.9
– Jobs and CronJob now batch/v1
– RBAC moved from v1alpha1 all the way to v1 in one release
• Explain which API group contains these new API objects.
• Find additional resources to start using the new API objects.

This chapter is about additional API resources or objects. We will learn about resources in the v1 API group, among others.
Stability increases and code becomes more stable as objects move from alpha versions, to beta, then v1 indicating stability.

DaemonSets, which ensure a Pod on every node, and StatefulSets, which stick a container to a node and otherwise act like
a deployment, have progressed to apps/v1 stability.
As a fast moving project keeping track of changes, and possible changes can be an important part of ongoing system administration. Release notes, as well as discussions to release notes can be found in version-dependent subdirectories at: https://github.com/kubernetes/enhancements/. For example, the release feature status can be found here:
https://kubernetes.io/docs/setup/release/notes/.
Starting with v1.16 deprecated API object versions will respond with an error instead of being accepted. This is an important
change from historic behavior.


6.2. THE V1 GROUP

6.2

The v1 Group

v1 API Group

• Pod
• Node
• Service Account
• Resource Quota
• Endpoint
• More added with each release

The v1 API is no longer a single group, but rather a collection of groups for each main object category. For example there is a
v1 group, a storage.k8s.io/v1 group, and rbac.authorization.k8s.io/v1 etc... Currently there are many v1 groups.
We have touched on several objects in lab exercises. Here are details for some of them:
• Node
Represents a machine (physical or virtual) that is part of your Kubernetes cluster. You can get more information
about nodes with the kubectl get nodes command. You can turn on and off the scheduling to a node with the
kubectl cordon/uncordon commands.
• Service Account
Provides an identifier for processes running in a pod to access the API server and performs actions that it is authorized
to do.
• Resource Quota
It is an extremely useful tool, allowing you to define quotas per namespace. For example, if you want to limit a specific
namespace to only run a given number of pods, you can write a resourcequota manifest, create it with kubectl and
the quota will be enforced.
• Endpoint
Generally, you do not manage endpoints. They represent the set of IPs for pods that match a particular service. They
are handy when you want to check that a service actually matches some running pods. If an endpoint is empty, then it
means that there are no matching pods and something is most likely wrong with your service definition.


Discovering API Groups
$ curl https://localhost:6443/apis \
--header "Authorization: Bearer $token" -k
{

"kind": "APIGroupList",
"apiVersion": "v1",
"groups": [
{
"name": "apiregistration.k8s.io",
"versions": [
{
"groupVersion": "apiregistration.k8s.io/v1",
"version": "v1"
}
],
"preferredVersion": {
"groupVersion": "apiregistration.k8s.io/v1",
"version": "v1"
}

We can take a closer look at the output of request for current APIs. Each of the name values can be appended to the
URL to see details of that group. For example you could drill down to find included objects at this URL: https://localhost:
6443/apis/apiregistration.k8s.io/v1beta1
If you follow this URL you will find only one resource, with a name of apiservices. If it seems to be listed twice the lower
output is for status. You’ll note there are different verbs or actions for each. Another entry is if this object is namespaced, or
restricted to only one namespace. In this case it is not.
You could curl each of these URIs and discover additional API objects, their characteristics and associated verbs.


6.3. API RESOURCES

6.3

API Resources

Deploying an Application

• Deployment
• ReplicaSet
• Pod

Using the kubectl create command we can quickly deploy an application. We have looked at the Pods created running the
application, like nginx. Looking closer you will find that a Deployment was created which manages a ReplicaSet which then
deploys the Pod. Lets take a closer look at each object.
• Deployment - A controller which manages the state of ReplicaSets and the pods within. The higher level control allows
for more flexibility with upgrades and administration. Unless you have a good reason, use a deployment.
• ReplicaSet - Orchestrates individual Pod life cycle and updates. These are newer versions of Replication Controllers
which differ only in selector support.
• Pod - As we’ve mentioned the lowest unit we can manage, runs the application container, possibly support containers.


DaemonSets

• Ensures every node runs a single pod
• Similar to ReplicaSet
• Often used for logging, metrics and security pods.
• Can be configured to avoid nodes

Should you want to have a logging application on every node a DaemonSet may be a good choice. The controller ensures that
a single pod, of the same type, runs on every node in the cluster. When a new node is added to the cluster a Pod, same as
deployed on the other nodes, is started. When the node is removed the DaemonSet makes sure the local Pod is deleted.
As usual, you get all the CRUD operations via kubectl:
$ kubectl get daemonsets
$ kubectl get ds


6.3. API RESOURCES

StatefulSet

• Similar to Deployment
• Ensures unique pods
• Guarantee ordering

Pods deployed using a StatefulSet use the same Pod specification. How this is different than a Deployment is that a
StatefulSet considers each Pod as unique and provides ordering to Pod deployment.
In order to track each Pod as a unique object the controller uses identity composed of stable storage, stable network identity,
and an ordinal. This identity remains with the node regardless to which node the Pod is running on at any one time.
The default deployment scheme is sequential starting with 0, such as app-0, app-1, app-2 etc. A following Pod will not launch
until the current Pod reaches a running and ready state. They are not deployed in parallel.


Autoscaling

• Agents which add or remove resources from the cluster.
• Horizontal Pod Autoscaling (HPA)
– Scale based on current CPU usage, or custom metric
– Must have Metrics Server or custom component running
• Vertical Pod Autoscaler
• Cluster Autoscaler (CA)
– Add or remove nodes based on utilization
– Makes request to cloud provider
– Pods which cannot be evicted prevent scale-down

In the autoscaling group we find the Horizontal Pod Autoscalers (HPA). This is a stable resource. HPAs automatically
scale Replication Controllers, ReplicaSets, or Deployments based on a target of 80% CPU usage by default. The
usage is checked by kubelet every 15 seconds and retrieved by Metrics Server API call every minute. HPA checks with
Metrics Server every 15 seconds. When a Pod needs to be added, the system takes immediate action. However, when a
Pod needs to be removed, the Horizontal Pod Autoscaler (HPA) waits 300 seconds before taking further action.
Other metrics can be used and queried via REST. The autoscaler does not collect the metrics, it only makes a request for the
aggregated information and increases or decreases the number of replicas to match the configuration.
The Cluster Autoscaler (CA) adds or removes nodes to the cluster based off of inability to deploy a Pod or having nodes
with low utilization for at least 10 minutes. This allows dynamic requests of resources from the cloud provider and minimizes
expense for unused nodes. If you are using CA nodes should be added and removed through cluster-autoscaler- commands. Scale-up and down of nodes is checked every 10 seconds, but decisions are made on a node every 10 minutes.
Should a scale-down fail the group will be rechecked in 3 minutes, with the failing node being eligible in five minutes. The total
time to allocate a new node is largely dependent on the cloud provider.
You can automatically scale a workload vertically using the Vertical Pod Autoscaler (VPA). Unlike the Horizontal Pod Autoscaler (HPA), the VPA is not included by default in Kubernetes — it’s a separate project maintained on GitHub.
https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler


6.3. API RESOURCES

Jobs
• Part of Batch API group
• Jobs run Pod until number of completions reached
– Batch processing or one-off Pods
– Ensure specified number of pods successfully terminate
– Can run multiple Pods in parallel
• Cronjob to run Job on regular basis
– Creates a Job about once per executing time
– Some issues, job should be idempotent
– Can run in serial or parallel
– Same time syntax as Linux cron job

Jobs are part of the batch API group. They are used to run a set number of pods to completion. If a pod fails, it will be
restarted until the number of completion is reached.
While they can be seen as a way to do batch processing in Kubernetes, they can also be used to run one-off pods. A Job
specification will have a parallelism and a completion key. If omitted, they will be set to one. If they are present, the parallelism
number will set the number of pods that can be running concurrently and the completion number will set how many pods need
to run successfully for the Job itself to be considered done. Several Job patterns can be implemented, like a traditional work
queue.

Cronjobs work in the similar manner to Linux jobs with the same time syntax. There are some cases where a job would not
be run during a time period or could run twice, as a result the requested Pod should be idempotent.
An option spec field is .spec.concurrencyPolicy which determines how to handle existing jobs should the time segment
expire. If set to Allow, the default, another concurrent job will be run. If set to Forbid the current job continues and the new
job is skipped. A value of Replace cancels the current job and starts a new job in its place.


6.4

RBAC APIs

RBAC

• rbac.authorization.k8s.io
• Provide resources
– ClusterRole
– ClusterRoleBinding
– RoleBinding
– Role
• Often combined with quotas for typical production deployments

The last API resources that we will look at are in the rbac.authorization.k8s.io group. We actually have four resources:
ClusterRole, Role, ClusterRoleBinding, and RoleBinding. They are used for Role Based Access Control (RBAC) to
Kubernetes.
$ curl localhost:8080/apis/rbac.authorization.k8s.io/v1
...

...

"groupVersion": "rbac.authorization.k8s.io/v1",
"resources": [
"kind": "ClusterRoleBinding"
"kind": "ClusterRole"
"kind": "RoleBinding"
"kind": "Role"

These resources allow us to define Roles within a cluster and associate users to these Roles. For example, we can define a
Role for someone who can only read pods in a specific namespace, or a Role that can create deployments, but no services.

Please Note
More on RBAC is covered later in the Security chapter.


6.5

Labs


---


---

# Chapter 6: API Objects
**Working directory: `~/lfs458/ch06-api-objects/`**

## Exercise 6.1: RESTful API Access

```bash
cd ~/lfs458/ch06-api-objects/
```

Overview
We will continue to explore ways of accessing the control plane of our cluster. In the security chapter we will discuss
there are several authentication methods, one of which is use of a Bearer token We will work with one then deploy a
local proxy server for application-level access to the Kubernetes API.
We will use the curl command to make API requests to the cluster, in an insecure manner. Once we know the IP address
and port, then the token we can retrieve cluster data in a RESTful manner. By default most of the information is restricted, but
changes to authentication policy could allow more access.
1. First we need to know the IP and port of a node running a replica of the API server. The cp system will typically have
one running. Use kubectl config view to get overall cluster configuration, and find the server entry. This will give us
both the IP and the port.
kubectl config view
apiVersion: v1
clusters:
- cluster:
certificate-authority-data: DATA+OMITTED
server: https://k8scp:6443
name: kubernetes
<output_omitted>

2. The creation of token secrets by Kubernetes is no longer automatic in recent releases. It then falls on the user to design
them as desired. Use the command shown below to create the token.
export token=$(kubectl create token default)

3. Test to see if you can get basic API information from your cluster. We will pass it the server name and port, the token
and use the -k option to avoid using a cert.
curl https://k8scp:6443/apis --header "Authorization: Bearer $token" -k
{

"kind": "APIGroupList",
"apiVersion": "v1",
"groups": [
{
"name": "apiregistration.k8s.io",
"versions": [
{
"groupVersion": "apiregistration.k8s.io/v1",
"version": "v1"
<output_omitted>

4. Try the same command, but look at API v1. Note that the path has changed to api.
curl https://k8scp:6443/api/v1 --header "Authorization: Bearer $token" -k
<output_omitted>


5. Now try to get a list of namespaces. This should return an error. It shows our request is being seen as
systemserviceaccount:, which does not have the RBAC authorization to list all namespaces in the cluster.
curl \
https://k8scp:6443/api/v1/namespaces --header "Authorization: Bearer $token" -k
<output_omitted>
"message": "namespaces is forbidden: User \"system:serviceaccount:default...
<output_omitted>


---

## Exercise 6.2: Using the Proxy

```bash
cd ~/lfs458/ch06-api-objects/
```

Another way to interact with the API is via a proxy. The proxy can be run from a node or from within a Pod through the use of
a sidecar. In the following steps we will deploy a proxy listening to the loopback address. We will use curl to access the API
server. If the curl request works, but does not from outside the cluster, we have narrowed down the issue to authentication
and authorization instead of issues further along the API ingestion process.
1. Begin by starting the proxy. It will start in the foreground by default. There are several options you could pass. Begin by
reviewing the help output.
kubectl proxy -h
Creates a proxy server or application-level gateway between localhost
and the Kubernetes API Server. It also allows serving static content
over specified HTTP path. All incoming data enters through one port
and gets forwarded to the remote kubernetes API Server port, except
for the path matching the static content path.
Examples:
# To proxy all of the kubernetes api and nothing else, use:
$ kubectl proxy --api-prefix=/
<output_omitted>

2. Start the proxy while setting the API prefix, and put it in the background. You may need to use enter to view the prompt.
Take note of the process ID, 225000 in the example below, we’ll use it to kill the process when we are done.
kubectl proxy --api-prefix=/ &
[1] 22500
Starting to serve on 127.0.0.1:8001

3. Now use the same curl command, but point toward the IP and port shown by the proxy. The output should be the same
as without the proxy, but may be formatted differently.
curl http://127.0.0.1:8001/api/
<output_omitted>

4. Make an API call to retrieve the namespaces. The command did not work in the previous section due to permissions,
but should work now as the proxy is making the request on your behalf.
curl http://127.0.0.1:8001/api/v1/namespaces
{

"kind": "NamespaceList",


"apiVersion": "v1",
"metadata": {
"selfLink": "/api/v1/namespaces",
"resourceVersion": "86902"
<output_omitted>

5. Stop the proxy service as we won’t need it any more. Use the process ID from a previous step. Your process ID may be
different.
kill 22500


---

## Exercise 6.3: Working with Jobs

```bash
cd ~/lfs458/ch06-api-objects/
```

While most API objects are deployed such that they continue to be available there are some which we may want to run a
particular number of times called a Job, and others on a regular basis called a CronJob

Create A Job
1. Create a job which will run a container which sleeps for three seconds then stops.
cp /home/student/LFS458/SOLUTIONS/s_06/job.yaml .
vim job.yaml

job.yaml
2
4
6
8
10
12

apiVersion: batch/v1
kind: Job
metadata:
name: sleepy
spec:
template:
spec:
containers:
- name: resting
image: busybox
command: ["/bin/sleep"]
args: ["3"]
restartPolicy: Never

2. Create the job, then verify and view the details. The example shows checking the job three seconds in and then again
after it has completed. You may see different output depending on how fast you type.
kubectl create -f job.yaml
job.batch/sleepy created

kubectl get job
NAME
sleepy

COMPLETIONS
0/1

DURATION
3s

AGE
3s

kubectl describe jobs.batch sleepy


Name:
Namespace:
Selector:
Labels:
Annotations:
Parallelism:
Completions:
Start Time:
Completed At:
Duration:
Pods Statuses:
<output_omitted>

sleepy
default
controller-uid=24c91245-d0fb-11e8-947a-42010a800002
controller-uid=24c91245-d0fb-11e8-947a-42010a800002
job-name=sleepy
<none>
1
Thu, 23 Aug 2024 10:47:53 +0000
Thu, 23 Aug 2024 10:48:00 +0000
5s
0 Running / 1 Succeeded / 0 Failed

kubectl get job
NAME
sleepy

COMPLETIONS
1/1

DURATION
5s

AGE
17s

3. View the configuration information of the job. There are three parameters we can use to affect how the job runs. Use -o
yaml to see these parameters. We can see that backoffLimit, completions, and the parallelism. We’ll add these
parameters next.
kubectl get jobs.batch sleepy -o yaml
<output_omitted>
uid: c2c3a80d-d0fc-11e8-947a-42010a800002
spec:
backoffLimit: 6
completions: 1
parallelism: 1
selector:
matchLabels:
<output_omitted>

4. As the job continues to AGE in a completion state, delete the job.
kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted

5. Edit the YAML and add the completions: parameter and set it to 5.
vim job.yaml

job.yaml
2
4
6
8

<output_omitted>
metadata:
name: sleepy
spec:
completions: 5
template:
spec:
containers:


#<--Add this line


<output_omitted>

6. Create the job again. As you view the job note that COMPLETIONS begins as zero of 5.
kubectl create -f job.yaml
job.batch/sleepy created

kubectl get jobs.batch
NAME
sleepy

COMPLETIONS
0/5

DURATION
5s

AGE
5s

7. View the pods that running. Again the output may be different depending on the speed of typing.
kubectl get pods
NAME
sleepy-z5tnh
sleepy-zd692
<output_omitted>

READY
0/1
1/1

STATUS
Completed
Running

RESTARTS
0

AGE
8s
3s

8. Eventually all the jobs will have completed. Verify then delete the job.
kubectl get jobs
NAME
sleepy

COMPLETIONS
5/5

DURATION
26s

AGE
10m

kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted

9. Edit the YAML again. This time add in the parallelism: parameter. Set it to 2 such that two pods at a time will be
deployed.
vim job.yaml

job.yaml
2
4
6
8

<output_omitted>
name: sleepy
spec:
completions: 5
parallelism: 2
template:
spec:
<output_omitted>

#<-- Add this line


10. Create the job again. You should see the pods deployed two at a time until all five have completed.
kubectl create -f job.yaml
job.batch/sleepy created

kubectl get pods
NAME
sleepy-8xwpc
sleepy-xjqnf
<output_omitted>

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
5s
5s

kubectl get jobs
NAME
sleepy

COMPLETIONS
3/5

DURATION
11s

AGE
11s

11. Add a parameter which will stop the job after a certain number of seconds. Set the activeDeadlineSeconds: to 15.
The job and all pods will end once it runs for 15 seconds. We will also increase the sleep argument to five, just to be
sure does not expire by itself.
vim job.yaml

2
4
6
8
10
12

<output_omitted>
completions: 5
parallelism: 2
activeDeadlineSeconds: 15
#<-- Add this line
template:
spec:
containers:
- name: resting
image: busybox
command: ["/bin/sleep"]
#<-- Edit this line
args: ["5"]
<output_omitted>

12. Delete and recreate the job again. It should run for 15 seconds, usually 3/5, then continue to age without further
completions.
kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted

kubectl create -f job.yaml
job.batch/sleepy created

kubectl get jobs
NAME
sleepy


COMPLETIONS
1/5

DURATION
6s

AGE
6s


kubectl get jobs
NAME
sleepy

COMPLETIONS
3/5

DURATION
16s

AGE
16s

13. View the message: entry in the Status section of the object YAML output.
kubectl get job sleepy -o yaml
<output_omitted>
status:
conditions:
- lastProbeTime: 2024-08-23T10:48:00Z
lastTransitionTime: 2024-08-23T10:48:00Z
message: Job was active longer than specified deadline
reason: DeadlineExceeded
status: "True"
type: Failed
failed: 2
startTime: 2024-08-23T10:48:00Z
succeeded: 3

14. Delete the job.
kubectl delete jobs.batch sleepy
job.batch "sleepy" deleted

Create a CronJob
A CronJob creates a watch loop which will create a batch job on your behalf when the time becomes true. We Will use our
existing Job file to start.
1. Copy the yaml file from the tarball.
cp /home/student/LFS458/SOLUTIONS/s_06/cronjob.yaml .

2. Verify the file to look like the annotated file shown below. Edit the lines mentioned below if needed. The three parameters
we added will need to be removed. Other lines will need to be further indented if needed.
vim cronjob.yaml

2
4
6
8
10
12

apiVersion: batch/v1
kind: CronJob
metadata:
name: sleepy
spec:
schedule: "*/2 * * * *"
jobTemplate:
spec:
template:
spec:
containers:
- name: resting


#<-- Update this line to CronJob

#<-- Add Linux style cronjob syntax
#<-- New jobTemplate and spec move
#<-- This and following lines move
#<-- four spaces to the right


image: busybox
command: ["/bin/sleep"]
args: ["5"]
restartPolicy: Never

14
16

3. Create the new CronJob. View the jobs. It will take two minutes for the CronJob to run and generate a new batch Job.
kubectl create -f cronjob.yaml
cronjob.batch/sleepy created

kubectl get cronjobs.batch
NAME
sleepy

SCHEDULE
*/2 * * * *

SUSPEND
False

ACTIVE

LAST SCHEDULE
<none>

AGE
8s

LAST SCHEDULE
21s

AGE
2m1s

kubectl get jobs.batch
No resources found.

4. After two minutes you should see jobs start to run.
kubectl get cronjobs.batch
NAME
sleepy

SCHEDULE
*/2 * * * *

SUSPEND
False

ACTIVE

kubectl get jobs.batch
NAME
sleepy-1539722040

COMPLETIONS
1/1

DURATION
5s

AGE
18s

DURATION
5s
6s
6s

AGE
5m17s
3m17s
77s

kubectl get jobs.batch
NAME
sleepy-1539722040
sleepy-1539722160
sleepy-1539722280

COMPLETIONS
1/1
1/1
1/1

5. Ensure that if the job continues for more than 10 seconds it is terminated. We will first edit the sleep command to run
for 30 seconds then add the activeDeadlineSeconds: entry to the container.
vim cronjob.yaml

2
4

....
jobTemplate:
spec:
template:
spec:


activeDeadlineSeconds: 10
containers:
- name: resting

7
9

....

11
13

....

command: ["/bin/sleep"]
args: ["30"]
restartPolicy: Never

#<-- Add this line

#<-- Edit this line

6. Delete and recreate the CronJob. It may take a couple of minutes for the batch Job to be created and terminate due to
the timer.
kubectl delete cronjobs.batch sleepy
cronjob.batch "sleepy" deleted

kubectl create -f cronjob.yaml
cronjob.batch/sleepy created

kubectl get jobs
NAME
sleepy-1539723240

COMPLETIONS
0/1

DURATION
61s

AGE
61s

kubectl get cronjobs.batch
NAME
sleepy

SCHEDULE
*/2 * * * *

SUSPEND
False

ACTIVE

LAST SCHEDULE
72s

AGE
94s

kubectl get jobs
NAME
sleepy-1539723240

COMPLETIONS
0/1

DURATION
75s

AGE
75s

DURATION
2m19s
19s

AGE
2m19s
19s

kubectl get jobs
NAME
sleepy-1539723240
sleepy-1539723360

COMPLETIONS
0/1
0/1

kubectl get cronjobs.batch
NAME
sleepy

SCHEDULE
*/2 * * * *

SUSPEND
False

ACTIVE

LAST SCHEDULE
31s

AGE
2m53s

7. Clean up by deleting the CronJob.


kubectl delete cronjobs.batch sleepy


cronjob.batch "sleepy" deleted


7.1

Deployment Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 142

7.2

Deployments and Replica Sets

7.3

DaemonSets . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 152

7.4

Labels

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 153

7.5

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 155

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 143


7.1


Deployment Overview

Overview

• Deployments
• Application Updates
• Labels
• ReplicaSet

As with other objects a, deployment can be made from a YAML or JSON spec file. When added to the cluster, the controller
will create a ReplicaSet and a Pod automatically. The containers, their settings and applications can be modified via an
update, which generates a new ReplicaSet which in turn generates new Pods.
The updated objects can be staged to replace previous objects as a block or as a rolling update, which is determined as part
of the deployment spec. Most updates can be configured by editing a YAML file and running kubectl apply. You can also use
kubectl edit to modify the in-use configuration. Previous versions of the ReplicaSets are kept, allowing a roll-back to return
to a previous configuration.
We will also talk more about labels. Labels are essential to administration in Kubernetes, but are not an API resource. They
are user defined key-value pairs which can be attached to any resource, and are stored in the metadata. Labels are used to
query or select resources in your cluster, allowing for flexible and complex management of the cluster.
As a label is arbitrary, you could select all resources used by developers, or belonging to a user, or any attached string without
having to figure out what kind or how many of such resources exist.


7.2. DEPLOYMENTS AND REPLICA SETS

7.2

Deployments and Replica Sets

Deployment Details
• Generate Deployment
$ kubectl create deployment dev-web --image=nginx:1.21
deployment "dev-web" created

• Generate YAML of newly created objects
kubectl get deployments,rs,pods -o yaml

• Sometimes JSON output can make it more clear
kubectl get deployments,rs,pods -o json
apiVersion: v1
items:
- apiVersion: apps/v1
kind: Deployment

We created a new deployment running a particular version of the nginx web server. Now we will look at the YAML output
which also shows default values, not passed to the object when created.
• apiVersion
A value of v1 indicates this object is considered to be a stable resource. In this case it is not the deployment. It is a
reference to the List type.
• items
As the previous line is a List this declares the list of items the command is showing.
• - apiVersion
The dash is a YAML indication of the first item of the list, which declares the apiVersion of the object as apps/v1. This
indicates the object is considered stable. Deployments are an operator used in many cases.
• kind
This is where the type of object to create is declared, in this case a deployment.


Deployment Configuration Metadata

metadata:
annotations:
deployment.kubernetes.io/revision: "1"
creationTimestamp: 2024-10-21T13:57:07Z
generation: 1
labels:
app: dev-web
name: dev-web
namespace: default
resourceVersion: "774003"
uid: d52d3a63-e656-11e7-9319-42010a800003

Continuing with the YAML output we see the next general block of output concerns the metadata of the deployment. This
is where we would find labels, annotations, and other non-configuration information. Note that this output will not show all
possible configuration. Many settings which are set to false by default are not shown, like podAffinity or nodeAffinity.
• annotations: These values do not configure the object, but provide further information that could be helpful to thirdparty applications or administrative tracking. Unlike labels they cannot be used to select an object with kubectl.
• creationTimestamp : When the object was originally created. Does not update if object edited.
• generation : How many times this object has been edited, changing the number of replicas for example.
• labels : Arbitrary strings used to select or exclude objects for use with kubectl, or other API calls. Helpful for admins
to select objects outside of typical object boundaries.
• name : This is a required string, which we passed from the command line. The name must be unique to the namespace.
• resourceVersion : A value tied to the etcd database to help with concurrency of objects. Any changes to the database
will cause this number to change.
• uid : remains a unique ID for the life of the object.


7.2. DEPLOYMENTS AND REPLICA SETS

Deployment Configuration Spec

spec:
progressDeadlineSeconds: 600
replicas: 1
revisionHistoryLimit: 10
selector:
matchLabels:
app: dev-web
strategy:
rollingUpdate:
maxSurge: 25%
maxUnavailable: 25%
type: RollingUpdate

There are two spec declarations for the deployment. The first will modify the ReplicaSet created, while the second will pass along Pod
configuration.
• spec : A declaration that following items will configure the object being created.
• progressDeadlineSeconds : Time in seconds until a progress error is reported during a change. Reasons could be quotas, image
issues, or limit ranges.
• replicas : As the object being created is a ReplicaSet this parameter determines how many Pods should be created. If you were
to use kubectl edit and change this value to two, a second Pod would be generated.
• revisionHistoryLimit : How many old ReplicaSet specifications to retain for rollback.
• selector : A collection of values ANDed together. All must be satisfied for the replica to match. Do not create Pods which match
these selectors as the deployment controller may try to control the resource leading to issues.
• matchLabels : set-based requirements of the Pod selector. Often found with matchExpressions statement to further designate
where the resource should be scheduled.
• strategy : A header for values having to do with updating Pods. Works with the later listed type. Could also be set to Recreate,
which would delete all existing pods before new pods are created. With RollingUpdate you can control how many Pods are deleted at
a time with the following parameters.
• maxsurge : Maximum number of Pods over desired number of Pods to create. Can be a percentage, default of 25%, or an absolute
number. This creates a certain number of new Pods before deleting old, for continued access.
• maxUnavailable: A number or percentage of Pods which can be in a state other than Ready during the update process.
• type : Even though listed last in the section, due to level of white space indentation it is read as the type of object being configured.

rollingUpdate


Deployment Configuration Pod Template
template:
metadata:
creationTimestamp: null
labels:
app: dev-web
spec:
containers:
- image: nginx:1.17.7-alpine
imagePullPolicy: IfNotPresent
name: dev-web
resources: {}
terminationMessagePath: /dev/termination-log
terminationMessagePolicy: File
dnsPolicy: ClusterFirst
restartPolicy: Always
schedulerName: default-scheduler
securityContext: {}
terminationGracePeriodSeconds: 30

We will see some similar values as we view the configuration for the Pods to be deployed. If the meaning is basically the same
we will not define it again.
• template :
Data being passed to the ReplicaSet to determine how
to deploy an object, in this case containers.
• containers : Key word indicating that the following
items of this indentation are for a container.
• image : This is the image name passed to the container
engine, typically Docker. The engine will pull the image
and create the Pod.
• imagePullPolicy : Policy settings passed along to
container engine about when and if an image should
be downloaded or used from a local cache.
• name : The leading stub of the Pod names. A unique
string will be appended.

• terminationMessagePolicy : The default value is
File which holds the termination method. Could also
be set to FallbackToLogsOnError which will use the
last chunk of container log if the message file is empty
and the container shows an error.
• dnsPolicy : Determine if DNS queries should go to
coredns or, if set to Default, use the node’s DNS resolution configuration.
• restartPolicy : Should the container be restarted if
killed. Automatic restarts is part of the typical strength
of Kubernetes.
• scheduleName : Allows for the use of a custom scheduler instead of the Kubernetes default.

• resources : By default empty this is where you would
set resource restrictions and settings such as a limit on
CPU or memory for the containers.

• securityContext : Flexible setting to pass one or
more security settings such as SELinux context, APParmor values, users and UIDs for the containers to
use.

• terminationMessagePath : A customizable location of
where to output success or failure information of a container

• terminationGracePeriodSeconds : Amount of time
to wait for a SIGTERM to run until a SIGKILL is used to
terminate the container.


7.2. DEPLOYMENTS AND REPLICA SETS

Deployment Configuration Status
status:
availableReplicas: 2
conditions:
- lastTransitionTime: 2024-10-21T13:57:07Z
lastUpdateTime: 2024-10-21T13:57:07Z
message: Deployment has minimum availability.
reason: MinimumReplicasAvailable
status: "True"
type: Available
- lastTransitionTime: "2024-10-29T06:00:24Z"
lastUpdateTime: "2024-10-29T06:00:33Z"
message: ReplicaSet "test-5f6778868d" has successfully progressed.
reason: NewReplicaSetAvailable
status: "True"
type: Progressing
observedGeneration: 2
readyReplicas: 2
replicas: 2
updatedReplicas: 2
The Status output is generated when the information is requested. The output above shows what the same deployment
were to look like if the number of replicas were increased to two. The times are different than when the deployment was first
generated.
• availableReplicas :
indicates how many were configured by the ReplicaSet. This would be compared to the later value of readyReplicas
which would be used to determine if all replicas have been fully generated and without error.
• observedGeneration :
shows how often the deployment has been updated. This information can be used to understand the roll-out and rollback situation of the deployment.


Scaling and Rolling Updates

• Deployment configuration can be dynamically updated Controllers
• Use set argument to create new Replica Set
• Also use edit to trigger update
• Update YAML file and use kubectl apply


7.2. DEPLOYMENTS AND REPLICA SETS

The API server allows for the configurations settings to be updated for most values. There are some immutable values, which may be
different depending on the version of Kubernetes you have deployed.
A common update is to change the number of replicas running. If this number is set to zero there would be no containers, but there would
still be a ReplicaSet and Deployment. This is the backend process when a Deployment is deleted.

$ kubectl scale deploy/dev-web --replicas=4
deployment "dev-web" scaled

$ kubectl get deployments
NAME
dev-web

READY
4/4

UP-TO-DATE

AVAILABLE

AGE
20s

Non-immutable values can be edited via text editor as well. For example to change the deployed version of the nginx web server to an older
version:

$ kubectl edit deployment nginx
....

....

containers:
- image: nginx:1.8
#<<---Set to an older version
imagePullPolicy: IfNotPresent
name: dev-web

This would trigger a rolling update of the deployment. While the deployment would show an older age, a review of the Pods would show a
recent update and older version of the web server application deployed.


Deployment Rollbacks

• Previous ReplicaSets retained for rollback
• Can pause and resume
• Deployment edits replica counts decrementing old and incrementing
new replicaSet

With some of previous the replicaSets of a Deployment being kept, you can also roll back to a previous revision by scaling up and down.
The number of previous configurations kept is configurable, and has changed from version to version.

$ kubectl create deploy ghost --image=ghost
$ kubectl annotate deployment/ghost kubernetes.io/change-cause="kubectl create deploy ghost --image=ghost"
$ kubectl get deployments ghost -o yaml
deployment.kubernetes.io/revision: "1"
kubernetes.io/change-cause: kubectl create deploy

ghost --image=ghost

Should an update fail, an improper image version for example, you can roll-back the change to a working version with
kubectl rollout undo:

$ kubectl set image deployment/ghost ghost=ghost:09 --all
$ kubectl get pods
NAME
ghost-2141819201-tcths

READY
0/1

STATUS
ImagePullBackOff

RESTARTS

AGE
1m

$ kubectl rollout undo deployment/ghost ; kubectl get pods
NAME
ghost-3378155678-eq5i6


READY
1/1

STATUS
Running

RESTARTS

AGE
7s


7.2. DEPLOYMENTS AND REPLICA SETS

Deployment Rollbacks (cont.)

$ kubectl rollout pause deployment/ghost
$ kubectl rollout resume deployment/ghost

You can roll back to a specific revision with the --to-revision=2 option.
You can also edit a Deployment using the kubectl edit command.
You can also pause a Deployment, and then resume.
Please note that you can still do a rolling update on Replication Controllers with the kubectl rolling-update command,
but this is done on the client side. Hence, if you close your client, the rolling update will stop.


7.3


DaemonSets

Using DaemonSets

• Runs on every node
• Same image on each
• Added and removed dynamically
• Use kind:

DaemonSet

A newer object to work with is the DaemonSet. This controller ensures that a single pod exists on each node in the cluster.
Every Pod uses the same image. Should a new node be added, the DaemonSet controller will deploy a new Pod on your
behalf. Should a node be removed, the controller will delete the Pod also.
The use of a DaemonSet allows for ensuring a particular container is always running. In a large and dynamic environment, it
can be helpful to have a logging or metric generation application on every node without an admin remembering to deploy that
application.
There are ways of effecting kube-scheduler such that some nodes will not run a DaemonSet.


7.4. LABELS

7.4

Labels

Labels

• Can exist in every resource metadata
• Hash label created by default
• Immutable as of API version apps/v1

Part of the metadata of an object is a label. Though they are not an API object, they are an important tool for cluster
administration. They can be used to select an object based on an arbitrary string, regardless of object type.
Every resource can contain labels in its metadata. By default, creating a Deployment with kubectl create adds a label as
we saw:
....
labels:
pod-template-hash: "3378155678"
run: ghost
....

You could then view labels in new columns:
$ kubectl get pods -l run=ghost
NAME
ghost-3378155678-eq5i6

READY
1/1

STATUS
Running

RESTARTS

AGE
10m

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
1

AGE
10m
1h

$ kubectl get pods -L run
NAME
ghost-3378155678-eq5i6
nginx-3771699605-4v27e


RUN
ghost
nginx


Labels (cont.)

• During creation or on the fly
• Easy to query or select

While you typically define labels in pod templates and in specifications of Deployments, you can also add labels on the fly:
$ kubectl label pods ghost-3378155678-eq5i6 foo=bar
$ kubectl get pods --show-labels
NAME
READY STATUS
ghost-3378155678-eq5i6
1/1
Running
pod-template-hash=3378155678,run=ghost

RESTARTS

AGE
11m

LABELS
foo=bar,

For example, if you want to force the scheduling of a pod on a specific node, you can use a nodeSelector in a pod definition,
add specific labels to certain nodes in your cluster and use those labels in the pod.
....
spec:
containers:
- image: nginx
nodeSelector:
disktype: ssd


7.5

Labs


---


---

# Chapter 7: Managing State with Deployments
**Working directory: `~/lfs458/ch07-deployments/`**

## Exercise 7.1: Working with ReplicaSets

```bash
cd ~/lfs458/ch07-deployments/
```

Overview
Understanding and managing the state of containers is a core Kubernetes task. In this lab we will first explore the API
objects used to manage groups of containers. The objects available have changed as Kubernetes has matured, so
the Kubernetes version in use will determine which are available. Our first object will be a ReplicaSet, which does
not include newer management features found with Deployments. A Deployment operator manages ReplicaSet
operators for you. We will also work with another object and watch loop called a DaemonSet which ensures a container
is running on newly added node.
Then we will update the software in a container, view the revision history, and roll-back to a previous version.
A ReplicaSet is a next-generation of a Replication Controller, which differs only in the selectors supported. The only
reason to use a ReplicaSet anymore is if you have no need for updating container software or require update orchestration
which won’t work with the typical process.
1. View any current ReplicaSets. If you deleted resources at the end of a previous lab, you should have none reported in
the default namespace.
kubectl get rs
No resources found in default namespace.

2. Create a YAML file for a simple ReplicaSet. The apiVersion setting depends on the version of Kubernetes you are
using. The object is stable using the apps/v1 apiVersion. We will use an older version of nginx then update to a newer
version later in the exercise.
cp /home/student/LFS458/SOLUTIONS/s_07/rs.yaml .
vim rs.yaml

rs.yaml
2
4
6
8
10
12
14
16
18

apiVersion: apps/v1
kind: ReplicaSet
metadata:
name: rs-one
spec:
replicas: 2
selector:
matchLabels:
system: ReplicaOne
template:
metadata:
labels:
system: ReplicaOne
spec:
containers:
- name: nginx
image: nginx:1.22.1
ports:
- containerPort: 80


3. Create the ReplicaSet:
kubectl create -f rs.yaml
replicaset.apps/rs-one created

4. View the newly created ReplicaSet:
kubectl describe rs rs-one
Name:
rs-one
Namespace:
default
Selector:
system=ReplicaOne
Labels:
<none>
Annotations:
<none>
Replicas:
2 current / 2 desired
Pods Status:
2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
Labels:
system=ReplicaOne
Containers:
nginx:
Image:
nginx:1.22.1
Port:
80/TCP
Host Port:
0/TCP
Environment:
<none>
Mounts:
<none>
Volumes:
<none>
Events:
<none>

5. View the Pods created with the ReplicaSet. From the yaml file created there should be two Pods. You may see a
Completed busybox which will be cleared out eventually.
kubectl get pods
NAME
rs-one-2p9x4
rs-one-3c6pb

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
5m4s
5m4s

6. Now we will delete the ReplicaSet, but not the Pods it controls.
kubectl delete rs rs-one --cascade=orphan
replicaset.apps "rs-one" deleted

7. View the ReplicaSet and Pods again:
kubectl describe rs rs-one
Error from server (NotFound): replicasets.apps "rs-one" not found

kubectl get pods
NAME
rs-one-2p9x4
rs-one-3c6pb


READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
7m
7m


8. Create the ReplicaSet again. As long as we do not change the selector field, the new ReplicaSet should take
ownership. Pod software versions cannot be updated this way.
kubectl create -f rs.yaml
replicaset.apps/rs-one created

9. View the age of the ReplicaSet and then the Pods within:
kubectl get rs
NAME
rs-one

DESIRED

CURRENT

READY

AGE
46s

kubectl get pods
NAME
rs-one-2p9x4
rs-one-3c6pb

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
8m
8m

10. We will now isolate a Pod from its ReplicaSet. Begin by editing the label of a Pod. We will change the system:
parameter to be IsolatedPod.
kubectl edit pod rs-one-3c6pb
....
labels:
system: IsolatedPod
managedFields:
....

#<-- Change from ReplicaOne

11. View the number of pods within the ReplicaSet. You should see two running.
kubectl get rs
NAME
rs-one

DESIRED

CURRENT

READY

AGE
4m

12. Now view the pods with the label key of system. You should note that there are three, with one being newer than others.
The ReplicaSet made sure to keep two replicas, replacing the Pod which was isolated.
kubectl get po -L system
NAME
rs-one-3c6pb
rs-one-2p9x4
rs-one-dq5xd

READY
1/1
1/1
1/1

STATUS
Running
Running
Running

RESTARTS
0

AGE
10m
10m
30s

SYSTEM
IsolatedPod
ReplicaOne
ReplicaOne

13. Delete the ReplicaSet, then view any remaining Pods.
kubectl delete rs rs-one
replicaset.apps "rs-one" deleted


kubectl get po
NAME
rs-one-3c6pb
rs-one-dq5xd

READY
1/1
0/1

STATUS
Running
Terminating

RESTARTS
0

AGE
14m
4m

14. In the above example the Pods had not finished termination. Wait for a bit and check again. There should be no
ReplicaSets, but one Pod.
kubectl get rs
No resources found in default namespaces.

kubectl get pod
NAME
rs-one-3c6pb

READY
1/1

STATUS
Running

RESTARTS

AGE
16m

15. Delete the remaining Pod using the label.
kubectl delete pod -l system=IsolatedPod
pod "rs-one-3c6pb" deleted


---

## Exercise 7.2: Working with Deployments

```bash
cd ~/lfs458/ch07-deployments/
```

A Deployment is a watch loop object which we have been working with in the previous labs. A Deployment provides a
declarative update to Pods and ReplicaSets and ensure a particular number of pods are created in general, several could be
on a single node. Deployment is a high-level resource object that is used to manage the rollout and scaling of containerized
applications. A Deployment describes the desired state of the application, such as the number of replicas, and the container
image to use. When a Deployment is created, Kubernetes will automatically create and manage the necessary replica
sets, which in turn will create and manage the necessary pods to ensure that the desired state of the application is met.
Deployment also provide rolling updates, which allow for updating an application to a new version without downtime by
gradually replacing the old replicas with new ones. Using Deployment in Kubernetes makes it easy to manage and scale
containerized applications while ensuring high availability and reliability.
1. We begin by creating a yaml file. In this case the kind would be set to deployment. We can generate the yaml file using
the imperative method
kubectl create deploy webserver --image nginx:1.22.1 --replicas=2 \
--dry-run=client -o yaml | tee dep.yaml
cat dep.yaml

dep.yaml
2
4
6

....
kind: Deployment
....
name: webserver
....
replicas: 2


....

9

....

app: webserver

2. Create and verify the newly formed Deployment. There should be two replicas of Pods created in the cluster.
kubectl create -f dep.yaml
deployment.apps/webserver created

kubectl get deploy
NAME
webserver

READY
2/2

UP-TO-DATE

AVAILABLE

AGE
14s

kubectl get pod
NAME
webserver-6cbc654ddc-lssbm
webserver-6cbc654ddc-xpmtl

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
42s
42s

3. Verify the image running inside the Pods. We will use this information in the next section.
kubectl describe pod webserver-6cbc654ddc-lssbm
Image:

| grep Image:

nginx:1.22.1


---

## Exercise 7.3: Rollout and Rollback using Deployment

```bash
cd ~/lfs458/ch07-deployments/
```

One of the advantages of micro-services is the ability to replace and upgrade a container while continuing to respond to client
requests. We will use the recreate setting that upgrades a container when the predecessor is deleted, then the use the
RollingUpdate feature as well, which begins a rolling update immediately.

nginx versions
The nginx software updates on a distinct timeline from Kubernetes. If the lab shows an older version please use the
current default, and then a newer version. Versions can be verified on the repositories on the registry
1. Begin by viewing the current strategy setting for the Deployment created in the previous section.
kubectl get deploy webserver -o yaml | grep -A 4 strategy
strategy:
rollingUpdate:
maxSurge: 25%
maxUnavailable: 25%
type: RollingUpdate


2. Edit the object to use the Recreate update strategy. This would allow the manual termination of some of the pods,
resulting in an updated image when they are recreated.
kubectl edit deploy webserver
....
strategy:
rollingUpdate:
maxSurge: 25%
maxUnavailable: 25%
type: Recreate
:q....

# <-- remove this line
# <-- remove this line
# <-- remove this line
# <-- Edit this line

3. Update the Deployment to use a newer version of the nginx server. This time use the set command instead of edit.
Set the version to be 1.23.1-alpine.
kubectl set image deploy webserver nginx=nginx:1.23.1-alpine --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/webserver image updated

4. Verify that the Image: parameter for the Pod checked in the previous section is unchanged.
kubectl get pod
NAME
webserver-6cf9cd5c74-qjph4
webserver-6cf9cd5c74-zc6x9

1/1
1/1

READY
STATUS
Running
Running

RESTARTS
35s
35s

AGE

kubectl describe po webserver-6cf9cd5c74-qjph4 |grep Image:
Image:

nginx:1.23.1-alpine

5. View the history of changes for the Deployment. You should see two revisions listed. As we did not add the the
change-cause annotation we didn’t see why the object updated.
kubectl rollout history deploy webserver
deployment.apps/webserver
REVISION CHANGE-CAUSE
<none>
kubectl set image deploy webserver nginx=nginx:1.23.1-alpine --record=true

6. View the settings for the various versions of the Deployment. The Image: line should be the only difference between
the two outputs.
kubectl rollout history deploy webserver --revision=1
deployment.apps/webserver with revision #1
Pod Template:
Labels:
app=webserver
pod-template-hash=6cbc654ddc
Containers:
nginx:


Image:
nginx:1.22.1
Port:
<none>
Host Port: <none>
Environment:
<none>
Mounts:
<none>
Volumes:
<none>

kubectl rollout history deploy webserver --revision=2
....
Image:
.....

nginx:1.23.1-alpine

7. Use kubectl rollout undo to change the Deployment back to previous version.
kubectl rollout undo deploy webserver
deployment.apps/webserver rolled back

kubectl get pod
NAME
webserver-6cbc654ddc-7wb5q
webserver-6cbc654ddc-svbtj

READY
1/1
1/1

STATUS
Running
Running

0

RESTARTS

AGE
37s
37s

kubectl describe pod webserver-6cbc654ddc-7wb5q |grep Image:
Image:

nginx:1.22.1

8. Let’s try the ”RollingUpdate” strategy next. First, open the deployment configuration file and change the update strategy
to ”RollingUpdate.” Then, just as you did before, update the container image to a new version (for example, set it to
nginx:1.26-alpine). After making these changes, apply the update and observe how the rollout is executed, ensuring
that the new version is deployed gradually.
9. Clean up the system by removing the Deployment.
kubectl delete deploy webserver
deployment.apps "webserver" deleted


---

## Exercise 7.4: Working with DaemonSets

```bash
cd ~/lfs458/ch07-deployments/
```

A DaemonSet is a watch loop object like a Deployment which we have been working with in the rest of the labs. The DaemonSet
ensures that when a node is added to a cluster, a pod will be created on that node. A Deployment would only ensure a
particular number of pods are created in general, several could be on a single node. Using a DaemonSet can be helpful to
ensure applications are on each node, helpful for things like metrics and logging especially in large clusters where hardware
may be swapped out often. Should a node be removed from a cluster the DaemonSet would ensure the Pods are garbage
collected before removal. Starting with Kubernetes v1.12 the scheduler handles DaemonSet deployment which means we can
now configure certain nodes to not have a particular DaemonSet pods.


This extra step of automation can be useful for using with products like ceph where storage is often added or removed, but
perhaps among a subset of hardware. They allow for complex deployments when used with declared resources like memory,
CPU or volumes.
1. We begin by creating a yaml file. In this case the kind would be set to DaemonSet. For ease of use we will copy the
previously created rs.yaml file and make a couple edits. Remove the Replicas: 2 line.
cp rs.yaml ds.yaml
vim ds.yaml

ds.yaml
2
4
6
8

....
kind: DaemonSet
....
name: ds-one
....
replicas: 2 #<<<----Remove this line
....
system: DaemonSetOne #<<-- Edit both references
....

2. Create and verify the newly formed DaemonSet. There should be one Pod per node in the cluster.
kubectl create -f ds.yaml
daemonset.apps/ds-one created

kubectl get ds
NAME
ds-one

DESIRED

CURRENT

READY

UP-TO-DATE

AVAILABLE

NODE-SELECTOR
<none>

AGE
1m

kubectl get pod
NAME
ds-one-b1dcv
ds-one-z31r4

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
2m
2m

3. Verify the image running inside the Pods. We will use this information in the next section.
kubectl describe pod ds-one-b1dcv | grep Image:
Image:

nginx:1.22.1


---

## Exercise 7.5: Rollout and Rollback using DaemonSet

```bash
cd ~/lfs458/ch07-deployments/
```

One of the advantages of micro-services is the ability to replace and upgrade a container while continuing to respond to client
requests. We will use the OnDelete setting that upgrades a container when the predecessor is deleted.


nginx versions
The nginx software updates on a distinct timeline from Kubernetes. If the lab shows an older version please use the
current default, and then a newer version. Versions can be seen with this command: sudo docker image ls nginx
1. Begin by viewing the current updateStrategy setting for the DaemonSet created in the previous section.
kubectl get ds ds-one -o yaml | grep -A 4 Strategy
updateStrategy:
rollingUpdate:
maxSurge:; 0
maxUnavailable: 1
type: RollingUpdate

2. Edit the object to use the OnDelete update strategy. This would allow the manual termination of some of the pods,
resulting in an updated image when they are recreated.
kubectl edit ds ds-one
....
updateStrategy:
rollingUpdate:
maxUnavailable: 1
type: OnDelete
status:
....

#<-- Edit to be this line

3. Update the DaemonSet to use a newer version of the nginx server. This time use the set command instead of edit. Set
the version to be 1.26.1-alpine.
kubectl set image ds ds-one nginx=nginx:1.26-alpine --record
Flag --record has been deprecated, --record will be removed in the future
daemonset.apps/ds-one image updated

4. Verify that the Image: parameter for the Pod checked in the previous section is unchanged.
kubectl describe po ds-one-b1dcv |grep Image:
Image:

nginx:1.22.1

5. Delete the Pod. Wait until the replacement Pod is running and check the version.
kubectl delete po ds-one-b1dcv
pod "ds-one-b1dcv" deleted

kubectl get pod
NAME
ds-one-xc86w
ds-one-z31r4

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
19s
4m8s

kubectl describe pod ds-one-xc86w |grep Image:


Image:

nginx:1.26-alpine

6. View the image running on the older Pod. It should still show version 1.22.1.
kubectl describe pod ds-one-z31r4 |grep Image:
Image:

nginx:1.22.1

7. View the history of changes for the DaemonSet. You should see two revisions listed. As we did not add the the
change-cause annotation we didn’t see why the object updated.
kubectl rollout history ds ds-one
daemonset.apps/ds-one
REVISION CHANGE-CAUSE
<none>
kubectl set image ds ds-one nginx=nginx:1.26-alpine --record=true

8. View the settings for the various versions of the DaemonSet. The Image: line should be the only difference between the
two outputs.
kubectl rollout history ds ds-one --revision=1
daemonsets "ds-one" with revision #1
Pod Template:
Labels:
system=DaemonSetOne
Containers:
nginx:
Image:
nginx:1.22.1
Port:
80/TCP
Environment:
<none>
Mounts:
<none>
Volumes:
<none>

kubectl rollout history ds ds-one --revision=2
....

Image:
.....

nginx:1.26-alpine

9. Use kubectl rollout undo to change the DaemonSet back to an earlier version. As we are still using the OnDelete
strategy there should be no change to the Pods.
kubectl rollout undo ds ds-one --to-revision=1
daemonset.apps/ds-one rolled back

kubectl describe pod ds-one-xc86w |grep Image:
Image:


nginx:1.26-alpine


10. Delete the Pod, wait for the replacement to spawn then check the image version again.
kubectl delete pod ds-one-xc86w
pod "ds-one-xc86w" deleted

kubectl get pod
NAME
ds-one-qc72k
ds-one-xc86w
ds-one-z31r4

READY
1/1
0/1
1/1

STATUS
Running
Terminating
Running

RESTARTS
0

AGE
10s
12m
28m

kubectl describe po ds-one-qc72k |grep Image:
Image:

nginx:1.22.1

11. View the details of the DaemonSet. The Image should be v1.22.1 in the output.
kubectl describe ds |grep Image:
Image:

nginx:1.22.1

12. View the current configuration for the DaemonSet in YAML output. Look for the updateStrategy: the the type:
kubectl get ds ds-one -o yaml
apiVersion: apps/v1
kind: DaemonSet
.....
terminationGracePeriodSeconds: 30
updateStrategy:
type: OnDelete
status:
currentNumberScheduled: 2
.....

13. Create a new DaemonSet, this time setting the update policy to RollingUpdate. Begin by generating a new config file.
kubectl get ds ds-one -o yaml > ds2.yaml

14. Edit the file.

Change the name, around line 69 and the update strategy around line 100, back to the default
RollingUpdate.
vim ds2.yaml
....
name: ds-two
....
type: RollingUpdate

15. Create the new DaemonSet and verify the nginx version in the new pods.
kubectl create -f ds2.yaml


daemonset.apps/ds-two created

kubectl get pod
NAME
ds-one-qc72k
ds-one-z31r4
ds-two-10khc
ds-two-kzp9g

READY
1/1
1/1
1/1
1/1

STATUS
Running
Running
Running
Running

RESTARTS
0
0

AGE
28m
57m
5m
5m

kubectl describe po ds-two-10khc |grep Image:
Image:

nginx:1.22.1

16. Edit the configuration file and set the image to a newer version such as 1.26-alpine.
kubectl edit ds ds-two
....
.....

- image: nginx:1.26-alpine

17. View the age of the DaemonSets. It should be around ten minutes old, depending on how fast you type.
kubectl get ds ds-two
NAME
ds-two

DESIRED

CURRENT

READY

UP-TO-DATE

AVAILABLE

NODE-SELECTOR
<none>

AGE
10m

18. Now view the age of the Pods. Two should be much younger than the DaemonSet. They are also a few seconds apart
due to the nature of the rolling update where one then the other pod was terminated and recreated.
kubectl get pod
NAME
ds-one-qc72k
ds-one-z31r4
ds-two-2p8vz
ds-two-8lx7k

READY
1/1
1/1
1/1
1/1

STATUS
Running
Running
Running
Running

RESTARTS
0
0

AGE
36m
1h
34s
32s

19. Verify the Pods are using the new version of the software.
kubectl describe po ds-two-8lx7k |grep Image:
Image:

nginx:1.26-alpine

20. View the rollout status and the history of the DaemonSets.
kubectl rollout status ds ds-two
daemon set "ds-two" successfully rolled out


21. Clean up the system by removing the DaemonSets.
kubectl delete ds ds-one ds-two
daemonset.apps "ds-one" deleted
daemonset.apps "ds-two" deleted


8.1

Helm Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 170

8.2

Helm . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 171

8.3

Using Helm . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 174

8.4

Kustomize Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 176

8.5

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 180


8.1


Helm Overview

Deploying Complex Applications

• Package your Kubernetes application via chart template
• Helm client to request install of chart
• Helm creates cluster resources according to chart
• Version 3 was major update.

We have used Kubernetes tools to deploy simple containers and services. Also necessary was to have a canonical location
for software. Helm is similar to a package manager like yum or apt, with a chart being similar to a package, in that it has the
binaries as well as installation and removal scripts.
A typical containerized application will have several manifests. Manifests for deployments, services, and configMaps. You will
probably also create some secrets, Ingress, and other objects. Each of these will need a manifest.
With Helm, you can package all those manifests and make them available as a single tarball. You can put the tarball in
a repository, search that repository, discover an application, and then, with a single command, deploy and start the entire
application, one or more times.
The tarballs can be collected in a repository for sharing. You can connect to multiple repositories of applications, including
those provided by vendors.
You will also be able to upgrade, or roll-back, an application easily from the command line.


8.2. HELM

8.2

Helm

Benefits of Using Helm

• Simplifies Deployment
• Reusable and Shareable Packages
• Version Control
• Easy Rollback and Recovery
• Parameterization and Customization

Helm provides a standard way to manage Kubernetes applications, reducing the complexity of deploying, upgrading, and
maintaining applications. Helm charts are reusable templates that can be shared across teams or publicly via repositories.
This promotes consistency and reusability in application deployment.
Helm charts can be versioned, allowing for precise control over which versions of applications are deployed. This facilitates
tracking changes and managing different application versions. Helm supports easy rollback to previous versions of applications. If an update causes issues, you can quickly revert to a stable state, minimizing downtime and disruption.
Charts can be customized using values.yaml files or command-line arguments. This flexibility allows for different configurations for various environments (e.g., development, staging, production). Helm integrates well with continuous integration and
continuous deployment (CI/CD) pipelines, automating the deployment process and ensuring consistent environments.
Helm allows for smooth and controlled upgrades of applications, managing changes to Kubernetes resources efficiently without
manual intervention. Helm has a large and active community, providing a wealth of pre-built charts for common applications.
This ecosystem support can save time and effort in setting up and managing applications.


Chart Contents

|-- Chart.yaml
|-- README.md
|-- templates
|
|-- NOTES.txt
|
|-- _helpers.tpl
|
|-- configmap.yaml
|
|-- deployment.yaml
|
|-- pvc.yaml
|
|-- secrets.yaml
|
|-- svc.yaml
|-- values.yaml

A chart is an archive set of Kubernetes resource manifests that make up a distributed application. You can check out
the GitHub repository where the Kubernetes community is curating charts. Others exist and can be easily created, for
example by a vendor providing software. Similar to the use of independent YUM repositories.

Chart.yaml contains some metadata about the chart, like its name, version, keywords, and so on, in this case for MariaDB.
values.yaml contains keys and values that are used to generate the release in your Cluster. These values are replaced in
the resource manifests using the Go templating syntax. And finally, the templates directory contains the resource manifests
that make up this MariaDB application.
More about creating charts can found at:
https://helm.sh/docs/topics/charts/.


8.2. HELM

Templates
apiVersion: v1
kind: Secret
metadata:
name: {{ template "fullname" . }}
labels:
app: {{ template "fullname" . }}
chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
release: "{{ .Release.Name }}"
heritage: "{{ .Release.Service }}"
type: Opaque
data:
mariadb-root-password: {{ default "" .Values.mariadbRootPassword | b64enc\
| quote }}
mariadb-password: {{ default "" .Values.mariadbPassword | b64enc | quote }}

The template are resource manifests which use the Go templating syntax. Variables defined in the values file, for example,
get injected in the template when a release is created. In the MariadDB example provided, the database passwords are stored
in a Kubernetes secret, and the database configuration is stored in a Kubernetes ConfigMap.
We can see that a set of labels are defined in the secret metadata using the chart name, Release name, etc. The actual
values of the passwords are read from values.yaml.


8.3

Using Helm

Chart Repositories and Hub

• Search for charts
• Add a new repository
• Simple HTTP servers with index file and tarball of charts
• helm repo
• ArtifactHub to replace Docker hub

Repositories are currently simple HTTP servers that contain an index file and a tarball of all the Charts present. Prior to adding
a repository you can only search Artifact Hub https://artifacthub.io/, using helm search hub.
$ helm search hub redis

You can interact with a repository using the helm repo commands.
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo list
NAME
bitnami

URL
https://charts.bitnami.com/bitnami

Once you have a repository available, you can search for Charts based on keywords. Below, we search for a redis Chart:
$ helm search repo bitnami

Once you can find the chart within a repository you can deploy it on your cluster.


8.3. USING HELM

Deploying a Chart

$ helm fetch bitnami/apache --untar
$ cd apache/
$ ls
Chart.lock Chart.yaml README.md
values.schema.json values.yaml

charts

ci

files

templates

$ helm install anotherweb .

To deploy a Chart, you can use the helm install command. There may be several required resources for the installation to be
successful, such as available PVs to match chart PVC. Currently the only way to discover which resources need to exist is by
reading the READMEs for each chart. This can be found by downloading the tarball and expanding it into the current directory.
Once requirements are met and edits are made you can install using the local files.
You will be able to list the release, delete it, even upgrade it and roll back.
The output of deployment should be carefully reviewed. It often includes information on access to the applications within. If
your cluster did not have a required cluster resource, the output is often the first place to begin troubleshooting.


8.4


Kustomize Overview

Overview of Kustomize

• Declarative Configuration Management
• Overlay System for Customization
• No Templating Required
• Integrated with kubectl
• Resource Generation

Kustomize was created to overcome the limitations of using pure templating systems, which often require duplicating YAML
files or using complex templating logic. With Kustomize, you define a base set of resources and then apply overlays to make
environment-specific adjustments, such as changing image tags, adding labels, or modifying resource limits. This method
supports a more modular and maintainable approach to managing Kubernetes configurations.
Unlike tools such as Helm, Kustomize does not rely on templating languages to inject variables into your YAML. Instead, it
uses strategic merging and patches to adjust configurations.
Kustomize is now a built-in feature of kubectl, which means you don’t need to install additional tools to leverage its capabilities.
This integration offers a streamlined workflow for deploying your applications. This integration allows you to run commands
like kubectl kustomize dir to view the Kustomized YAML and kubectl apply -k dir to apply it to your cluster


8.4. KUSTOMIZE OVERVIEW

Concepts of Kustomize

• Central Kustomization File - kustomization.yaml
• Bases and Overlays
• Resources
• Patches and Transformers

At the heart of Kustomize is the kustomization.yaml file. This file lists the resources that you want to manage, along with
instructions on how to modify them.
By structuring your files into bases and overlays, you avoid repetition and ensure consistency across different environments.
For example, a base might contain the common configuration for a deployment, while overlays can adjust the replica count or
image tags for a development environment without altering the base files. Patches let you apply fine-grained changes, such as
adding a specific environment variable, while transformers help you apply broad modifications like injecting a common label to
all resources.
Kustomize includes generators for creating resources such as ConfigMaps and Secrets dynamically. This is particularly useful
for scenarios where you need to create these resources from literal values or files at deployment time.


Installation and Setup

• Integrated with kubectl
• Standalone kustomize tool
• Directory Structure

If you have a recent version of kubectl (v1.14+), Kustomize functionality is already included.
Alternatively, you can install the standalone Kustomize binary for advanced features or to use it outside of kubectl.
kustomize can be installed with a simple installation script give below
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/\
kustomize/master/hack/install_kustomize.sh" | bash

Organize your files into a clean hierarchy, separating bases from overlays. A typical directory structure might have a base/
folder containing common YAML files and separate folders such as overlays/dev/ or overlays/prod/ for environment-specific
customizations.


8.4. KUSTOMIZE OVERVIEW

kustomization.yaml

• Centralized Configuration
• Resource Aggregation
• Common Metadata Injection
• Name Modification
• Dynamic Resource Generation
• Patching and Overrides

kustomization.yaml is a key configuration file used by Kustomize to customize and manage Kubernetes YAML manifests.
Serves as the primary file that tells Kustomize what resources to use and how to modify them.
Contains the details of all the resources you want to include, example dpeloyment, service.
Allows you to apply environment-specific changes (like development, staging, production) without duplicating YAML files.
Overlays let you modify parts of your configuration (e.g., image versions, labels) on top of the base setup.
You can automatically add prefixes or suffixes to resource names to differentiate between environments. For instance, adding
a prefix - namePrefix: lf-. This would transform a resource named myapp-deployment into lf-myapp-deployment.
Adds common labels or annotations to all the resources defined. This is useful for organizing and filtering resources in your
cluster.
Generates ConfigMaps or Secrets on the fly from literal values or files.
Applies patches to the base resources, enabling you to override or modify specific fields without altering the original files.


8.5


Labs


---


---

# Chapter 8: Helm and Kustomize
**Working directory: `~/lfs458/ch08-helm/`**

## Exercise 8.1: Working with Helm and Charts

```bash
cd ~/lfs458/ch08-helm/
```

Overview
helm allows for easy deployment of complex configurations. This could be handy for a vendor to deploy a multi-part
application in a single step. Through the use of a Chart, or template file, the required components and their relationships are declared. Local agents like Tiller use the API to create objects on your behalf. Effectively its orchestration for
orchestration.
There are a few ways to install Helm. The newest version may require building from source code. We will download a
recent, stable version. Once installed we will deploy a Chart, which will configure MariaDB on our cluster.

Install Helm
1. On the cp node use wget to download the compressed tar file. Various versions can be found here: https://github.com/
helm/helm/releases/
wget https://get.helm.sh/helm-v3.19.0-linux-amd64.tar.gz
<output_omitted>
helm-v3.19.0-linux-amd64.tar.gz 100%[===================>] 17.18M --.-KB/s
in 0.1s
2025-10-18 13:38:09 (101 MB/s) - ‘helm-v3.19.0-linux-amd64.tar.gz’ saved [16624839/16624839]

2. Uncompress and expand the file.
tar -xvf helm-v3.19.0-linux-amd64.tar.gz
linux-amd64/
linux-amd64/helm
linux-amd64/README.md
linux-amd64/LICENSE

3. Copy the helm binary to the /usr/local/bin/ directory, so it is usable via the shell search path.
sudo cp linux-amd64/helm /usr/local/bin/helm

4. A Chart is a collection of files to deploy an application. There is a good starting repo available on https://github.com/
kubernetes/charts/tree/master/stable, provided by vendors, or you can make your own. Search the current Charts in
the Helm Hub or an instance of Monocular for available stable databases. Repos change often, so the following output
may be different from what you see.
helm search hub database
URL

CHART VERSION
APP VERSION
DESCRIPTION
https://artifacthub.io/packages/helm/drycc/data...
1.0.2
A PostgreSQL database used by Drycc Workflow.
https://artifacthub.io/packages/helm/drycc-cana...
1.0.0
A PostgreSQL database used by Drycc
Workflow.
https://artifacthub.io/packages/helm/camptocamp...
0.0.6
1.0
Expose services and secret to access postgres
d...
https://artifacthub.io/packages/helm/cnieg/h2-d...
1.0.3
1.4.199
A helm chart to deploy h2-database


<output_omitted>

5. You can also add repositories from various vendors, often found by searching artifacthub.io such as ealenn, who has
an echo program.
helm repo add ealenn https://ealenn.github.io/charts
"ealenn" has been added to your repositories

helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "ealenn" chart repository
Update Complete. Happy Helming!

6. We will install the tester tool. The - -debug option will create a lot of output. The output will typically suggest ways to
access the software.
helm upgrade -i tester ealenn/echo-server

--debug

history.go:56: [debug] getting history for release tester
Release "tester" does not exist. Installing it now.
install.go:173: [debug] Original chart version: ""
install.go:190: [debug] CHART PATH: /home/student/.cache/helm/repository/echo-server-0.5.0.tgz
client.go:122: [debug] creating 4 resource(s)
NAME: tester
<output_omitted>

7. Ensure the newly created tester-echo-server pod is running. Fix any issues, if not.
8. Look for the newly created service. Send a curl to the ClusterIP. You should get a lot of information returned.
kubectl get svc
NAME
kubernetes
tester-echo-server

TYPE
ClusterIP
ClusterIP

CLUSTER-IP
10.96.0.1
10.98.252.11

EXTERNAL-IP
<none>
<none>

PORT(S)
443/TCP
80/TCP

AGE
26h
11m

curl 10.98.252.11
{"host":{"hostname":"10.98.252.11","ip":"::ffff:192.168.74.128","ips":
[]},"http":{"method":"GET","baseUrl":"","originalUrl":"/","protocol":
"http"},"request":{"params":{"0":"/"},"query":{},"cookies":{},"body":
{},"headers":{"host":"10.98.252.11","user-agent":"curl/7.58.0","accept":
"*/*"}},"environment":{"PATH":"/usr/local/sbin:/usr/local/bin:/usr/sbin:
/usr/bin:/sbin:/bin","TERM":"xterm","HOSTNAME":"tester-echo-server786768d9f4-4zsz9","ENABLE__HOST":"true","ENABLE__HTTP":"true","ENABLE__
<output_omitted>

9. View the Chart history on the system. The use of the -a option will show all Charts including deleted and failed
attempts.


helm list
NAME
STATUS
tester
deployed

NAMESPACE
REVISION
CHART
default
echo-server-0.5.0

UPDATED
APP VERSION
2025-10-19 13:42:38.262327888 +0000 UTC
0.6.0

10. Delete the tester Chart. No releases of tester should be found.
helm uninstall tester
release "tester" uninstalled

helm list
NAME

NAMESPACE

REVISION

UPDATED

STATUS

CHART

APP VERSION

11. Find the downloaded chart. It should be a compressed tarball under the user’s home directory. Your echo version may
be slightly different.
find $HOME -name *echo*
/home/student/.cache/helm/repository/echo-server-0.5.0.tgz

12. Move to the archive directory and extract the tarball. Take a look at the files within.
cd $HOME/.cache/helm/repository ; tar -xvf echo-server-*
echo-server/Chart.yaml
echo-server/values.yaml
echo-server/templates/_helpers.tpl
echo-server/templates/configmap.yaml
echo-server/templates/deployment.yaml
<output_omitted>

13. Examine the values.yaml file to see some of the values that could have been set.
cat echo-server/values.yaml
<output_omitted>

14. You can also download the values file to review or modify it before proceeding with the installation. Once ready, add an
additional repository and download the Metrics Server chart.
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm fetch metrics-server/metrics-server --untar
cd metrics-server/

15. Take a look at the chart. You’ll note it looks similar to the previous. Read through the :values.yaml:. Make changes to
values.yaml file to include the –kubelet-insecure-tls argument before deploying.
ls


Chart.lock Chart.yaml README.md
values.schema.json values.yaml

charts

ci

files

templates

less values.yaml
# Default values for metrics-server.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.
image:
repository: registry.k8s.io/metrics-server/metrics-server
##
defaultArgs:
- --cert-dir=/tmp
- --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
- --kubelet-use-node-status-port
- --metric-resolution=15s
- --kubelet-insecure-tls
#
<---- add this line
<output_omitted>

16. Use the values.yaml file to install the chart. Take a look at the output and ensure the pod is running.
helm install metrics-server . -n kube-system
NAME: metrics-server
LAST DEPLOYED: Tue Oct 21 11:09:16 2025
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
* Metrics Server
<output_omitted>

*

17. Verify the installed metrics server is working fine and able to display the metrics.
kubectl get pods -n kube-system | grep metrics
metrics-server-799f7ccf68-b9t8d

1/1

Running

2m38s

kubectl top nodes
NAME
cp
worker

CPU(cores)
202m
41m

CPU(%)
10%
2%

MEMORY(bytes)
1156Mi
717Mi

MEMORY(%)
14%
9%


---

## Exercise 8.2: Horizontal Pod Autoscaler (HPA)

```bash
cd ~/lfs458/ch08-helm/
```

1. In this exercise, you will explore how Kubernetes automatically scales applications based on real-time resource usage.
The Horizontal Pod Autoscaler (HPA) adjusts the number of pod replicas to maintain target CPU utilization. You already
have the metrics-server installed in the previous lab, which is required for HPA to collect performance metrics.
2. Deploy a lightweight web application that consumes measurable CPU resources. The image registry.k8s.io/hpa-example
is a small HTTP server that performs CPU work on each request. By setting explicit resources.requests and


resources.limits, you define the CPU boundaries that the HPA will monitor. The limit of 10m (10 millicores) ensures
that even a small load triggers noticeable scaling events.
cp /home/student/LFS458/SOLUTIONS/s_08/hpa-deploy.yaml .
vim hpa-deploy.yaml

rs.yaml
2
4
6
8
10
12
14
16
18
20
22
24
26
28
30
32
34
36

apiVersion: apps/v1
kind: Deployment
metadata:
name: hpa-app
spec:
replicas: 2
selector:
matchLabels:
app: hpa-app
template:
metadata:
labels:
app: hpa-app
spec:
containers:
- name: web
image: registry.k8s.io/hpa-example
ports:
- containerPort: 80
resources:
requests:
cpu: 5m
limits:
cpu: 10m
--apiVersion: v1
kind: Service
metadata:
name: hpa-app
spec:
type: ClusterIP
selector:
app: hpa-app
ports:
- port: 80
targetPort: 80

3. Create the Deployment:
kubectl create -f hpa-deploy.yaml
deployment.apps/hpa-app created
service/hpa-app created

4. Verify that your deployment is running as expected. The kubectl get command will show your Deployment, ReplicaSet,
Pods, and Service. Each Pod should be Running on one of your worker nodes, indicating successful scheduling and
networking.


kubectl get deploy,rs,pods,svc -o wide
NAME

READY
SELECTOR
deployment.apps/hpa-app
2/2
registry.k8s.io/hpa-example

UP-TO-DATE

AVAILABLE

AGE

CONTAINERS

app=hpa-app

2m6s

web

IMAGES

NAME

DESIRED
CURRENT
READY
AGE
CONTAINERS
SELECTOR
replicaset.apps/hpa-app-844785cc56
2
2m6s
web
registry.k8s.io/hpa-example
app=hpa-app,pod-template-hash=844785cc56

IMAGES

NAME

READY
NOMINATED NODE
READINESS GATES
pod/hpa-app-844785cc56-djrtq
1/1
<none>
pod/hpa-app-844785cc56-zbklc
1/1
<none>

STATUS

RESTARTS

AGE

IP

NODE

Running

2m6s

192.168.0.1

cp

<none>

Running

2m6s

192.168.1.68

worker

<none>

NAME
service/hpa-app
service/kubernetes

TYPE
ClusterIP
ClusterIP

CLUSTER-IP
10.107.208.86
10.96.0.1

EXTERNAL-IP
<none>
<none>

PORT(S)
80/TCP
443/TCP

AGE
2m6s
20m

SELECTOR
app=hpa-app
<none>

5. Before creating an HPA resource, you can generate its YAML file using the --dry-run=client flag. This is a safe
and powerful way to preview what Kubernetes will create without making actual changes. The generated YAML can be
version-controlled or customized later.
kubectl autoscale deployment hpa-app \
--cpu=50% --min=2 --max=10 \
--dry-run=client -o yaml > hpa.yaml
cat hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
...

Here, the HPA targets 50% CPU utilization: when average CPU usage across Pods exceeds 50% of their requested
value, Kubernetes will add replicas; when usage falls below, it will gradually reduce replicas back down.
6. Apply the HPA definition and confirm it’s being tracked by the cluster. You should immediately see an entry in the HPA
list, though the TARGETS column will remain 0% until load is detected.
kubectl apply -f hpa.yaml
kubectl get hpa
horizontalpodautoscaler.autoscaling/hpa-app created
NAME
hpa-app

REFERENCE
Deployment/hpa-app

TARGETS
cpu: 20%/50%

MINPODS

MAXPODS

REPLICAS

AGE
33s

At this point, your HPA is active and continuously monitoring metrics-server data.
7. In a separate terminal, use the siege tool to simulate concurrent HTTP traffic to your service. Each request triggers
CPU work inside the container, causing CPU utilization to spike. Run the test for a couple of minutes to allow HPA
metrics to stabilize and trigger scaling.


sudo apt-get update && sudo apt-get install -y siege
siege -q -c 5 -t 2m http://10.107.208.86
New configuration template added to /home/student/.siege
Run siege -C to view the current settings in that file

The spike in requests will push average CPU usage well above 50%, forcing the HPA to scale up pods automatically.
8. While the load test is running, observe HPA behavior in real time. The TARGETS column will show CPU utilization
percentage and you’ll see the number of replicas gradually increase.
watch kubectl get hpa
NAME
hpa-app

REFERENCE
Deployment/hpa-app

TARGETS
210%/50%

MINPODS

MAXPODS

REPLICAS

AGE
3m

The HPA controller polls metrics every 15 seconds. It scales conservatively to avoid thrashing, so scaling might appear
delayed.
9. You can also watch Pods scale dynamically. The new Pods will be distributed across both nodes, confirming the scheduler is balancing workloads properly.
watch kubectl get pods -o wide
NAME
hpa-app-6d7d9c7c9c-hbw9t
hpa-app-6d7d9c7c9c-8qj9d
hpa-app-6d7d9c7c9c-7k2h4
...

READY
1/1
1/1
1/1

STATUS
Running
Running
Running

RESTARTS
0

AGE
6m
2m
1m

NODE
worker
cp
worker

10. Once you stop the load test, the average CPU usage will drop, and the HPA will gradually reduce the number of replicas.
Scaling down is intentionally slower to ensure stability and avoid oscillation.
# Press Ctrl+C in siege window
watch kubectl get hpa
NAME
hpa-app

REFERENCE
Deployment/hpa-app

TARGETS
cpu: 20%/50%

MINPODS

MAXPODS

REPLICAS

AGE
7m20s

The Deployment eventually stabilizes back to 2 pods when idle.
11. Troubleshooting Tips:
• If HPA shows <unknown> metrics, ensure metrics-server is healthy and running in kube-system.
• Verify that the container defines a cpu request; HPA relies on requests, not limits, to compute utilization.
• Check kubectl top pods to view live CPU usage.
• Scaling behavior can be tuned in HPA v2 using custom metrics and stabilization windows.
12. Finally, clean up your resources to free up the cluster for other labs.
kubectl delete -f hpa.yaml -f hpa-deploy.yaml
horizontalpodautoscaler.autoscaling "hpa-app" deleted from default namespace
deployment.apps "hpa-app" deleted from default namespace


service "hpa-app" deleted from default namespace


---

## Exercise 8.3: Working with Kustomize

```bash
cd ~/lfs458/ch08-helm/
```

1. Kustomize is a tool for customizing Kubernetes configurations. Its built into kubectl CLI.
kubectl kustomize --help
Build a set of KRM resources using a 'kustomization.yaml' file. The DIR argument must be a path to
a directory
containing 'kustomization.yaml', or a git repository URL with a path suffix specifying same with
respect to the
repository root. If DIR is omitted, '.' is assumed.
Examples:
# Build the current working directory
kubectl kustomize
...
<output_omitted>

2. Create a directory structure and copy the resource files in appropriate directory.
mkdir -p myapp/base myapp/overlays/dev myapp/overlays/prod
tree myapp
myapp
|--base
|-- overlays
|-- dev
|-- prod

3. Copy appropriate resource yaml files from the Solutions directory to directory structure created above.
cp /home/student/LFS458/SOLUTIONS/s_08/*.yaml-base myapp/base/
cp /home/student/LFS458/SOLUTIONS/s_08/*.yaml-dev myapp/overlays/dev
cp /home/student/LFS458/SOLUTIONS/s_08/*.yaml-prod myapp/overlays/prod
tree myapp
myapp
|-- base
|
|-- deployment.yaml-base
|
|-- kustomization.yaml-base
|
|-- service.yaml-base
|-- overlays
|-- dev
|
|-- deployment-patch.yaml-dev
|
|-- kustomization.yaml-dev
|
|-- service-patch.yaml-dev
|-- prod
|-- deployment-patch.yaml-prod
|-- kustomization.yaml-prod
|-- service-patch.yaml-prod


5 directories, 9 files

4. Rename the manifest files in the base directory.
cd myapp/base
for file in *.yaml-base; do mv "$file" "${file/-base/}"; done
ls *.yaml
deployment.yaml

kustomization.yaml

service.yaml

5. Rename the manifest files in the overlays/dev directory.
cd ../overlays/dev/
for file in *.yaml-dev; do mv "$file" "${file/-dev/}"; done
ls *.yaml
deployment.yaml

kustomization.yaml

service.yaml

6. Rename the manifest files in the overlays/prod directory.
cd ../prod
for file in *.yaml-prod; do mv "$file" "${file/-prod/}"; done
ls *.yaml
deployment.yaml

kustomization.yaml

service.yaml

7. Verify to see if the files and directory structure match the below output.
cd
tree myapp
myapp
|-- base
|
|-- deployment.yaml
|
|-- kustomization.yaml
|
|-- service.yaml
|-- overlays
|-- dev
|
|-- deployment-patch.yaml
|
|-- kustomization.yaml
|
|-- service-patch.yaml
|-- prod
|-- deployment-patch.yaml
|-- kustomization.yaml
|-- service-patch.yaml
5 directories, 9 files


8. The kustomization.yaml manifest file in the base directory has the details of the resources needs tobe created along
with additional metadata injection and name modification.
vim myapp/base/kustomization.yaml

2
4
6
8
10

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namePrefix: lfresources:
- deployment.yaml
- service.yaml
labels:
- includeSelectors: true
pairs:
company: linux-foundation

12

9. Build the configuration and preview the output before applying it to the cluster. The base resource configurations are used
as a foundation, and then environment-specific modifications are applied when building for a particular environment.
kubectl kustomize myapp/base
kubectl kustomize myapp/overlays/dev
kubectl kustomize myapp/overlays/prod

10. Once Verified, the configuration can be applied to the cluster using the -k option along with the kubectl
kubectl apply -k myapp/base/
service/lf-myapp created
deployment.apps/lf-myapp created

11. Verify the resources have been deployed correctly and the metadata has been injected as per kustomization.yaml
kubectl get all -l company=linux-foundation
NAME
pod/lf-myapp-5b68c7d779-ngsq4
pod/lf-myapp-5b68c7d779-z8pth

READY
1/1
1/1

NAME
service/lf-myapp

CLUSTER-IP
10.104.21.181

TYPE
ClusterIP

NAME
deployment.apps/lf-myapp

READY
2/2

NAME
replicaset.apps/lf-myapp-5b68c7d779

STATUS
Running
Running

UP-TO-DATE
DESIRED

RESTARTS
0

AGE
3m23s
3m23s

EXTERNAL-IP
<none>

PORT(S)
80/TCP

AVAILABLE

AGE
3m23s

CURRENT

READY

AGE
3m23s

AGE
3m23s

12. Likewise, the remaining configurations can also be patched
kubectl apply -k myapp/overlays/dev/


service/lf-myapp configured
deployment.apps/lf-myapp configured

13. Verify if the resources have been configured as per the manifest files present in the overlays/dev directory.
14. Clean up by deleting the resources.
kubectl delete

-k myapp/overlays/dev/

service "lf-myapp" deleted
deployment.apps "lf-myapp" deleted


9.1

Volumes Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 192

9.2

Volumes

9.3

Persistent Volumes . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 197

9.4

Rook . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 201

9.5

Passing Data To Pods . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 202

9.6

ConfigMaps . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 205

9.7

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 193

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 207


9.1


Volumes Overview

Overview

• Object to save data longer than container lifetime
• Many Volume types to choose from
• Define Persistent Volumes (PV)
• Define Persistent Volume Claims (PVC)
• Create Secrets
• Create ConfigMaps

Container engines have traditionally not offered storage that outlives the container. As containers are considered transient
this could lead to a loss of data, or complex exterior storage options. A Kubernetes Volume shares the Pod lifetime, not the
containers within. Should a container terminate, the data would continue to be available to the new container.
A volume is a directory, possibly pre-populated, made available to containers in a Pod. The creation of the directory, the
back-end storage of the data, and the contents depend on the volume type. There are many different volume types ranging
from rbd to gain access to Ceph, to NFS, to dynamic volumes from a cloud provider like Google’s gcePersistentDisk. Each
has particular configuration options and dependencies.
The Container Storage Interface (CSI) adoption enables the goal of an industry standard interface for container orchestration allow access to arbitrary storage systems. Currently volume plugins are ”in-tree”, meaning they are compiled and
built with the core Kubernetes binaries. This ”out-of-tree” object will allow storage vendors to develop a single driver and allow
the plugin to be containerized. This will replace the existing Flex plugin which requires elevated access to the host node, a
large security concern.
Should you want your storage lifetime to be distinct from a Pod you can use Persistent Volumes. These allow for empty
or pre-populated volumes to be claimed by a Pod using a Persistent Volume Claim then outlive the Pod. Data inside the
volume could then be used by another Pod or as a means of retrieving data.
There are two API Objects which exist to provide data to a Pod already. Encoded data can be passed using a Secret and
non-encoded data passed with a ConfigMap. These can be used to pass important data like SSH keys, passwords or even a
configuration file like /etc/hosts.


9.2. VOLUMES

9.2

Volumes

Introducing Volumes

Figure 9.1: K8s Pod Volumes

A Pod specification can declare one or more volumes and where they are made available. Each requires a name, a type, and a
mount point. The same volume can be made available to multiple containers within a Pod, which can be a method of containerto-container communication. A a volume can also be made available to multiple Pods, with each given an access mode to
write. There is no concurrency checking which means data corruption is probable unless outside locking takes place.
Part of a Pod request is a particular access mode. As a request the user may be granted more, but not less access, though
a direct match is attempted first. The cluster groups volumes with the same mode together, then sorts volumes by size from
smallest to largest. The claim is checked against each in that access mode group until a volume of sufficient size matches. The
three access modes are ReadWriteOnce, which allows read-write by a single node, ReadOnlyMany, which allows read-only
by multiple nodes, and ReadWriteMany which allows read-write by many nodes. Thus two pods on the same node can write
to a ReadWriteOnce, but a third pod on a different node would not become ready due to a FailedAttachVolume error.
When a volume is requested the local kubelet uses the kubelet pods.go script to map the raw devices, determine and make
the mount point for the container, then create the symbolic link on the host node filesystem to associate the storage to the
container. The API server makes a request for the storage to the StorageClass plugin, but the specifics of the requests to
the back-end storage depend on the plugin in use.
If a request for a particular StorageClass was not made then the only parameters used will be access mode and size. The
volume could come from any of the storage types available and there is no configuration to determine which of the available
will be used.


Volume Spec
apiVersion: v1
kind: Pod
metadata:
name: fordpinto
namespace: default
spec:
containers:
- image: simpleapp
name: gastank
command:
- sleep
- "3600"
volumeMounts:
- mountPath: /scratch
name: scratch-volume
volumes:
- name: scratch-volume
emptyDir: {}

One of the many types of storage available is an emptyDir. The kubelet will create the directory in the container, but not
mount any storage. Any data created is written to the shared container space. As a result it would not be persistent storage.
When the Pod is destroyed the directory would be deleted along with the container.
The YAML above would create a Pod with a single container with a volume named scratch-volume created, which would
create the /scratch directory inside the container.


9.2. VOLUMES

Volume Types

• Several types possible, more being added
– awsElasticBlockStore
– azureDisk
– azureFile
– cephfs
– csi
– downwardAPI
– emptyDir
– fc (fibre channel)
– flocker

– gcePersistentDisk
– gitRepo
– glusterfs
– hostPath
– iscsi
– local
– nfs
– projected
– portworxVolume

– quobyte
– rbd
– scaleIO
– secret
– storageos
– vsphereVolume
– persistentVolumeClaim
– CSIPersistentVolumeSource

There are several types that you can use to define volumes, each with their pros and cons. Some are local, many make use
of network-based resources.
In GCE or AWS, you can use Volumes of type GCEpersistentDisk or awsElasticBlockStore, which allows you to mount
GCE and EBS disks in your Pods, assuming you have already set up accounts and privileges.

emptyDir and hostPath volumes are easy to use. As mentioned, emptyDir is an empty directory that gets erased when
the Pod dies but is recreated when the container restarts. The hostPath volume mounts a resource from the host node
filesystem. The resource could be a directory, file socket, character, or block device. These resources must already exist on
the host to be used. There are two types, DirectoryOrCreate and FileOrCreate, which create the resources on the host
and use them if they don’t already exist.
NFS (Network File System) and iSCSI (Internet Small Computer System Interface) are straightforward choices for multiple
readers scenarios.
CSI allows for even more flexibility and decoupling plugins without the need to edit the core Kubernetes code. It was developed
as a standard for exposing arbitrary plugins in the future.
Note: Many in-tree storage drivers are deprecated and removed in the current Kubernetes version and all operations for the
in-tree deprecated volume type is redirected to the CSI driver.


Shared Volume Example
....
containers:
- name: alphacont
image: busybox
volumeMounts:
- mountPath: /alphadir
name: sharevol
- name: betacont
image: busybox
volumeMounts:
- mountPath: /betadir
name: sharevol
volumes:
- name: sharevol
emptyDir: {}

The above YAML creates a pod, exampleA, with two containers both with access to one shared volume. You could use
emptyDir or hostPath easily, since those types do not require any additional setup and will work in your Kubernetes cluster.
$ kubectl exec -ti exampleA -c betacont -- touch /betadir/foobar
$ kubectl exec -ti exampleA -c alphacont -- ls -l /alphadir
total 0
-rw-r--r-- 1 root root 0 Nov 19 16:26 foobar

Note that one container, betacont wrote, and the other container, alphacont had immediate access to the data. There is
nothing to keep the containers from overwriting the other’s data. Locking or versioning considerations must be part of the
containerized application to avoid corruption.


9.3. PERSISTENT VOLUMES

9.3

Persistent Volumes

Persistent Volumes and Claims

• Useful for porting data
• Resources managed via API
• Storage abstraction
• Several phases
$ kubectl get pv
$ kubectl get pvc

A persistent volume (pv) is a storage abstraction used to retain data longer then the Pod using it. Pod define a volume
of type persistentVolumeClaim with various parameters for size and possibly the type of back-end storage known as its
StorageClass. The cluster then attaches the persistentVolume.
Kubernetes will dynamically use volumes that are available irrespective of its storage type, allowing claims to any back-end
storage.
There are several phases to persistent storage:
• Provisioning can be from PVs created in advance by the cluster administrator, or requested from a dynamic source
such as the cloud provider.
• Binding occurs when a control loop on the cp notices the PVC, containing an amount of storage, access request, and
optionally a particular StorageClass. The watcher locates a matching PV or waits for the StorageClass provisioner to
create one. The PV must match at least the storage amount requested, but may provide more.
• The use phase begins when the bound volume is mounted for the Pod to use, which continues as long as the Pod
requires.
• Releasing happens when the Pod is done with the volume and an API request is sent deleting the PVC. The volume
remains in the state from when the claim is deleted until available to a new claim. The resident data remains depending
on the persistentVolumeReclaimPolicy.
• The reclaim phase has three options: Retain which keeps the data intact allowing for an admin to handle the storage
and data. Delete tells the volume plug-in to delete the API object as well as the storage behind it. The Recycle option
runs an rm -rf /mountpoint then makes it available to a new claim. With the stability of dynamic provisioning the
Recycle is planned to be deprecated.


Persistent Volume

kind: PersistentVolume
apiVersion: v1
metadata:
name: 10Gpv01
labels:
type: local
spec:
capacity:
storage: 10Gi
accessModes:
- ReadWriteOnce
hostPath:
path: "/somepath/data01"

This shows a basic declaration of a PersistentVolume using the hostPath type. Each type will have its own configuration
settings. For example an already created Ceph or GCE Persistent Disks would not need to be configured but could be claimed
from the provider.
Persistent volumes are not a namespaces object, but persistent volume claims are.
A beta feature of v1.13 allows for static provisioning of Raw Block Volumes, which currently supports Fibre Channel, AWS
EBS, Azure Disk, and RBD plugins among others.
The use of locally attached storage has been graduated to a stable feature. This feature is often used as part of distributed
file systems and databases.


9.3. PERSISTENT VOLUMES

Persistent Volume Claim
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
name: myclaim
spec:
accessModes:
- ReadWriteOnce
resources:
requests:
storage: 8Gi

In the Pod:
spec:
containers:
....
volumes:
- name: test-volume
persistentVolumeClaim:
claimName: myclaim
With a persistent volume created in your cluster, you can then write a manifest for a claim and use that claim in your pod
definition. In the Pod, the volume uses the persistentVolumeClaim.
The Pod configuration could also be as complex as this:
volumeMounts:
- name: Cephpd
mountPath: /data/rbd
volumes:
- name: rbdpd
rbd:
monitors:
- '10.19.14.22:6789'
- '10.19.14.23:6789'
- '10.19.14.24:6789'
pool: k8s
image: client
fsType: ext4
readOnly: true
user: admin
keyring: /etc/ceph/keyring
imageformat: "2"
imagefeatures: "layering"


Dynamic Provisioning

• Claim filled via auto-provisioning
• No need for admin to pre-create PVs
• Uses the StorageClass API object
• StorageClass defines volume plugin, or provisioner to use
• Single, default class possible via annotation

While handling volumes with a persistent volume definition and abstracting the storage provider using a claim is powerful, a
cluster administrator needed to create those volumes in the first place. Starting in v1.4 Dynamic Provisioning allowed for
the cluster to request storage from an exterior, pre-configured, source. API calls made by the appropriate plug-in allow for a
wide range of dynamic storage use.
The StorageClass API resource allows an administrator to define a persistent volume provisioner of a certain type, passing
storage-specific parameters.
With a StorageClass created, a user can request a claim which the API Server fills via auto-provisioning. The resource will
also be reclaimed as configured by the provider. AWS and GCE are common choices for dynamic storage, but other options
exist such as a Ceph cluster or iSCSI.
Here is an example of a StorageClass using GCE:
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
name: fast
provisioner: kubernetes.io/gce-pd
parameters:
type: pd-ssd


# Could be any name


9.4. ROOK

9.4

Rook

Using Rook for Storage Orchestration

• File, block and object storage
• Multiple storage providers
• Hyper-scale or hyper-converge
• Leverages CRDs and a rook operator

In keeping with the decouple and distributed nature of Cloud technology the Rook project https://rook.io allows orchestration
of storage using multiple storage providers.
As with other agents of the cluster Rook uses custom resource definitions (CRD) and a custom operator to provision storage
according to the back-end storage type, upon API call.
Several storage providers are supported:
• Ceph
• Cassandra
• Network File System (NFS)


9.5


Passing Data To Pods

Secrets

• Leverages base64 encoding
• Is not encryption unless further configured
• Conversion to generic data, acceptable everywhere
• Encoded manually or via kubectl create secret
$ kubectl create secret generic mysql --from-literal=password=root

Pods can access local data using volumes, but there is some data you don’t want readable to the naked eye. Passwords may
be an example. Using the Secret API resource the same password could be encoded or encrypted.
A secret is not encrypted, only base64-encoded, by default. One must create a EncryptionConfiguration with a key and
proper identity. Then the kube-apiserver needs the --encryption-provider-config flag set to a previously configured
provider such as aescbc or ksm. Once this is enabled you need to recreate every secret as they are encrypted upon write.
Multiple keys are possible. Each key for a provider is tried during decryption. The first key of the first provider is used for
encryption. To rotate keys first create a new key, restart (all) kube-apiserver processes, then recreate every secret.
You can see the encoded string inside the secrets with kubectl. The secret will be decoded and be presented as a string
saved to a file. The file can be used as an environmental variable or in a new directory, similar to the presentation of a volume.
A secret can be made manually as well, then inserted into a YAML file.


9.5. PASSING DATA TO PODS

Using Secrets via Environment Variables

...
spec:
containers:
- image: mysql:5.5
name: dbpod
env:
- name: MYSQL_ROOT_PASSWORD
valueFrom:
secretKeyRef:
name: mysql
key: password

A secret can be used as an environment variable in a Pod. Above we see one being configured.
There is not a limit to the number of Secrets used, but there is a 1MB limit to their size. Each secret occupies memory,
along with other API objects, so very large numbers of secrets could deplete memory on a host.
They are stored in the tmpfs storage on the host node, and are only sent only to the host running Pod. All volumes requested
by a Pod must be mounted before the containers within the Pod are started. So a secret must exist prior to being requested.


Mounting Secrets as Volumes
...
spec:
containers:
- image: busybox
command:
- sleep
- "3600"
volumeMounts:
- mountPath: /mysqlpassword
name: mysql
name: busy
volumes:
- name: mysql
secret:
secretName: mysql

You can also mount secrets as files using a volume definition in a Pod manifest. The mount path will contain a file whose name
will be the key of the secret created with the kubectl create secret step earlier.
Once the Pod is running, you can verify that the secret is indeed accessible in the container:
$ kubectl exec -ti busybox -- cat /mysqlpassword/password
LFTr@1n


9.6. CONFIGMAPS

9.6

ConfigMaps

Portable Data With ConfigMaps
• Decouple configuration data from container image
• Not encoded or encrypted
• Can be created from various sources
– Multiple files in same directory
– Individual files
– Literal values
• Can be consumed in various ways
– Container environmental variables
– Use ConfigMap values in Pod commands
– Populate Volume from ConfigMap
– Add ConfigMap data to specific path in Volume
– Set file names and access mode in Volume from ConfigMap data
– Can be used by system components and controllers
A similar API resource to Secrets is a ConfigMap, except the data is not encoded. In keeping with the concept of decoupling
in Kubernetes. Using a ConfigMap decouples the container image from configuration artifacts.
They store data as sets of key-value pairs or plain configuration files in any format. The data can come from a collection of
files or all files in a directory. It can also be populated from a literal value.
A ConfigMap can be used in several different ways. A container can use the data as environmental variables from one or
more sources. The values contained inside can be passed to commands inside the pod. A Volume or a file in a Volume can
be created, including different names and particular access modes. In addition, cluster components like controllers can use
the data.
Let’s say you have a file on your local filesystem called config.js. You can create a ConfigMap The configmap object will
have a data section containing the content of the file:
$ kubectl get configmap foobar -o yaml
kind: ConfigMap
apiVersion: v1
metadata:
name: foobar
data:
config.js: |
{
...}


Using ConfigMaps

env:
- name: SPECIAL_LEVEL_KEY
valueFrom:
configMapKeyRef:
name: special-config
key: special.how
volumes:
- name: config-volume
configMap:
name: special-config

Like secrets, you can use ConfigMaps as environment variables or using a volume mount. They must exist prior to being used
by a Pod, unless marked as optional. They also reside in a specific namespace.
In the case of environment variables, your pod manifest will use the valueFrom key and the configMapKeyRef value to read
the values.
With volumes, define a volume with the configMap type in your Pod and mount it where it needs to be used.


9.7

Labs


---


---

# Chapter 9: Volumes and Data
**Working directory: `~/lfs458/ch09-volumes/`**

## Exercise 9.1: Create a ConfigMap

```bash
cd ~/lfs458/ch09-volumes/
```

Overview
Container files are ephemeral, which can be problematic for some applications. Should a container be restarted the
files will be lost. In addition, we need a method to share files between containers inside a Pod.
A Volume is a directory accessible to containers in a Pod. Cloud providers offer volumes which persist further than the
life of the Pod, such that AWS or GCE volumes could be pre-populated and offered to Pods, or transferred from one
Pod to another. Ceph is also another popular solution for dynamic, persistent volumes.
Unlike current Docker volumes a Kubernetes volume has the lifetime of the Pod, not the containers within. You
can also use different types of volumes in the same Pod simultaneously, but Volumes cannot mount in a nested
fashion. Each must have their own mount point. Volumes are declared with spec.volumes and mount points with
spec.containers.volumeMounts parameters. Each particular volume type, may have other restrictions. https:
//kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes
We will also work with a ConfigMap, which is basically a set of key-value pairs. This data can be made available so that
a Pod can read the data as environment variables or configuration data. A ConfigMap is similar to a Secret, except
they are not base64 byte encoded arrays. They are stored as strings and can be read in serialized form.
There are three different ways a ConfigMap can ingest data, from a literal value, from a file or from a directory of files.
1. We will create a ConfigMap containing primary colors. We will create a series of files to ingest into the ConfigMap.
First, we create a directory primary and populate it with four files. Then we create a file in our home directory with our
favorite color.
mkdir primary
echo c > primary/cyan
echo m > primary/magenta
echo y > primary/yellow
echo k > primary/black
echo "known as key" >> primary/black
echo blue > favorite

2. Now we will create the ConfigMap and populate it with the files we created as well as a literal value from the command
line.
kubectl create configmap colors \
--from-literal=text=black \
--from-file=./favorite \
--from-file=./primary/
configmap/colors created

3. View how the data is organized inside the cluster. Use the yaml then the json output type to see the formatting.
kubectl get configmap colors
NAME
colors


DATA

AGE
30s


kubectl get configmap colors -o yaml
apiVersion: v1
data:
black: |
k
known as key
cyan: |
c
favorite: |
blue
magenta: |
m
text: black
yellow: |
y
kind: ConfigMap
<output_omitted>

4. Now we can create a Pod to use the ConfigMap. In this case a particular parameter is being defined as an environment
variable.
cp /home/student/LFS458/SOLUTIONS/s_09/simpleshell.yaml .
vim simpleshell.yaml

simpleshell.yaml
2
4
6
8
10
12
14

apiVersion: v1
kind: Pod
metadata:
name: shell-demo
spec:
containers:
- name: nginx
image: nginx
env:
- name: ilike
valueFrom:
configMapKeyRef:
name: colors
key: favorite

5. Create the Pod and view the environmental variable. After you view the parameter, exit out and delete the pod.
kubectl create -f simpleshell.yaml
pod/shell-demo created

kubectl exec shell-demo -- /bin/bash -c 'echo $ilike'
blue


kubectl delete pod shell-demo
pod "shell-demo" deleted

6. All variables from a file can be included as environment variables as well. Comment out the previous env: stanza
and add a slightly different envFrom to the file. Having new and old code at the same time can be helpful to see and
understand the differences. Recreate the Pod, check all variables and delete the pod again. They can be found spread
throughout the environment variable output.
vim simpleshell.yaml

simpleshell.yaml
2
4
6
8
10

<output_omitted>
image: nginx
#
env:
#
- name: ilike
#
valueFrom:
#
configMapKeyRef:
#
name: colors
#
key: favorite
envFrom:
- configMapRef:
name: colors

#<-- Same indent as image: line

kubectl create -f simpleshell.yaml
pod/shell-demo created

kubectl exec shell-demo -- /bin/bash -c 'env'
black=k
known as key
KUBERNETES_SERVICE_PORT_HTTPS=443
cyan=c
<output_omitted>

kubectl delete pod shell-demo
pod "shell-demo" deleted

7. A ConfigMap can also be created from a YAML file. Create one with a few parameters to describe a car.
cp /home/student/LFS458/SOLUTIONS/s_09/car-map.yaml .
vim car-map.yaml


car-map.yaml
2
4
6
8

apiVersion: v1
kind: ConfigMap
metadata:
name: fast-car
namespace: default
data:
car.make: Ford
car.model: Mustang
car.trim: Shelby

8. Create the ConfigMap and verify the settings.
kubectl create -f car-map.yaml
configmap/fast-car created

kubectl get configmap fast-car -o yaml

2
4
6

apiVersion: v1
data:
car.make: Ford
car.model: Mustang
car.trim: Shelby
kind: ConfigMap
<output_omitted>

9. We will now make the ConfigMap available to a Pod as a mounted volume. You can again comment out the previous
environmental settings and add the following new stanza. The containers: and volumes: entries are indented the
same number of spaces.
vim simpleshell.yaml

simpleshell.yaml
2
4
6
8
10
12

<output_omitted>
spec:
containers:
- name: nginx
image: nginx
volumeMounts:
- name: car-vol
mountPath: /etc/cars
volumes:
- name: car-vol
configMap:
name: fast-car
<comment out rest of file>

10. Create the Pod again. Verify the volume exists and the contents of a file within. Due to the lack of a carriage return in
the file your next prompt may be on the same line as the output, Shelby.


kubectl create -f simpleshell.yaml
pod "shell-demo" created


/dev/root

kubectl exec shell-demo -- /bin/bash -c 'df -ha |grep car'
9.6G

3.2G

6.4G

34% /etc/cars

kubectl exec shell-demo -- /bin/bash -c 'cat /etc/cars/car.trim'
Shelby

#<-- Then your prompt

11. Delete the Pod and ConfigMaps we were using.
kubectl delete pods shell-demo
pod "shell-demo" deleted

kubectl delete configmap fast-car colors
configmap "fast-car" deleted
configmap "colors" deleted


---

## Exercise 9.2: Creating a Persistent NFS Volume (PV)

```bash
cd ~/lfs458/ch09-volumes/
```


We will first deploy an NFS server. Once tested we will create a persistent NFS volume for containers to claim.
1. Install the software on your cp node.
sudo apt-get update && sudo \
apt-get install -y nfs-kernel-server
<output_omitted>

2. Make and populate a directory to be shared. Also give it similar permissions to /tmp/
sudo mkdir /opt/sfw
sudo chmod 1777 /opt/sfw/
sudo bash -c 'echo software > /opt/sfw/hello.txt'

3. Edit the NFS server file to share out the newly created directory. In this case we will share the directory with all. You can
always snoop to see the inbound request in a later step and update the file to be more narrow.
sudo vim /etc/exports
/opt/sfw/ *(rw,sync,no_root_squash,subtree_check)

4. Cause /etc/exports to be re-read:


sudo exportfs -ra

5. Test by mounting the resource from your second node.
sudo apt-get -y install nfs-common
<output_omitted>

showmount -e cp
Export list for cp:
/opt/sfw *

sudo mount cp:/opt/sfw /mnt
ls -l /mnt
total 4
-rw-r--r-- 1 root root 23 Aug 28 17:55 hello.txt

6. Return to the cp node and create a YAML file for the object with kind, PersistentVolume. Use the hostname of the cp
server and the directory you created in the previous step. Only syntax is checked, an incorrect name or directory will not
generate an error, but a Pod using the resource will not start. Note that the accessModes do not currently affect actual
access and are typically used as labels instead.
cp /home/student/LFS458/SOLUTIONS/s_09/PVol.yaml .
vim PVol.yaml

PVol.yaml
2
4
6
8
10
12
14

apiVersion: v1
kind: PersistentVolume
metadata:
name: pvvol-1
spec:
capacity:
storage: 1Gi
accessModes:
- ReadWriteMany
persistentVolumeReclaimPolicy: Retain
nfs:
path: /opt/sfw
server: cp
#<-- Edit to match cp node
readOnly: false

7. Create the persistent volume, then verify its creation.
kubectl create -f PVol.yaml
persistentvolume/pvvol-1 created

kubectl get pv


NAME

CAPACITY
ACCESS MODES
VOLUMEATTRIBUTESCLASS
REASON
pvvol-1
1Gi
RWX
6s
RECLAIM POLICY
AGE
Retain

STATUS

CLAIM

STORAGECLASS

Available

<unset>


---

## Exercise 9.3: Creating a Persistent Volume Claim (PVC)

```bash
cd ~/lfs458/ch09-volumes/
```

Before Pods can take advantage of the new PV we need to create a Persistent Volume Claim (PVC).
1. Begin by determining if any currently exist.
kubectl get pvc
No resources found in default namespace.

2. Create a YAML file for the new pvc.
cp /home/student/LFS458/SOLUTIONS/s_09/pvc.yaml .
vim pvc.yaml

pvc.yaml
2
4
6
8
10

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: pvc-one
spec:
accessModes:
- ReadWriteMany
resources:
requests:
storage: 200Mi

3. Create and verify the new pvc is bound. Note that the size is 1Gi, even though 200Mi was suggested. Only a volume of
at least that size could be used.
kubectl create -f pvc.yaml
persistentvolumeclaim/pvc-one created

kubectl get pvc
NAME
pvc-one

STATUS
Bound

VOLUME
pvvol-1

CAPACITY
1Gi

ACCESS MODES
RWX

STORAGECLASS

VOLUMEATTRIBUTESCLASS
<unset>

4. Look at the status of the pv again, to determine if it is in use. It should show a status of Bound.
kubectl get pv
NAME
CAPACITY
ACCESS MODES
VOLUMEATTRIBUTESCLASS
REASON

RECLAIM POLICY
AGE

STATUS

CLAIM


STORAGECLASS

AGE
7s

pvvol-1
1Gi
<unset>

RWX

Retain
3m45s

Bound

default/pvc-one

5. Create a new deployment to use the pvc. We will copy and edit an existing deployment yaml file. We will change the
deployment name then add a volumeMounts section under containers and a volumes section to the general spec. The
name used must match in both places, whatever name you use. The claimName must match an existing pvc. As shown
in the following example. The volumes line is the same indent as containers and dnsPolicy.
cp /home/student/LFS458/SOLUTIONS/s_09/nfs-pod.yaml .
vim nfs-pod.yaml

nfs-pod.yaml
2
4
6
8
10
12
14
16
18
20
22
24
26
28
30
32
34
36
38
40
42
44

apiVersion: apps/v1
kind: Deployment
metadata:
annotations:
deployment.kubernetes.io/revision: "1"
generation: 1
labels:
run: nginx
name: nginx-nfs
#<-- Edit name
namespace: default
spec:
replicas: 1
selector:
matchLabels:
run: nginx
strategy:
rollingUpdate:
maxSurge: 1
maxUnavailable: 1
type: RollingUpdate
template:
metadata:
creationTimestamp: null
labels:
run: nginx
spec:
containers:
- image: nginx
imagePullPolicy: Always
name: nginx
volumeMounts:
#<-- Add these three lines
- name: nfs-vol
mountPath: /opt
ports:
- containerPort: 80
protocol: TCP
resources: {}
terminationMessagePath: /dev/termination-log
terminationMessagePolicy: File
volumes:
#<-- Add these four lines
- name: nfs-vol
persistentVolumeClaim:
claimName: pvc-one
dnsPolicy: ClusterFirst


restartPolicy: Always
schedulerName: default-scheduler
securityContext: {}
terminationGracePeriodSeconds: 30

46
48

6. Create the pod using the newly edited file.
kubectl create -f nfs-pod.yaml
deployment.apps/nginx-nfs created

7. Look at the details of the pod. You may see the daemonset pods running as well.
kubectl get pods
NAME
nginx-nfs-1054709768-s8g28

READY
1/1

STATUS
Running

RESTARTS

AGE
3m

kubectl describe pod nginx-nfs-1054709768-s8g28
Name:
Namespace:
Priority:
Node:

nginx-nfs-1054709768-s8g28
default
worker/10.128.0.5

<output_omitted>
Mounts:
/opt from nfs-vol (rw)
<output_omitted>
Volumes:
nfs-vol:
Type:
PersistentVolumeClaim (a reference to a PersistentV...
ClaimName:
pvc-one
ReadOnly:
false
<output_omitted>


---

## Exercise 9.4: Using a ResourceQuota to Limit PVC Count and Usage

```bash
cd ~/lfs458/ch09-volumes/
```

The flexibility of cloud-based storage often requires limiting consumption among users. We will use the ResourceQuota object
to both limit the total consumption as well as the number of persistent volume claims.
1. Begin by deleting the deployment we had created to use NFS, the pv and the pvc.
kubectl delete deploy nginx-nfs
deployment.apps "nginx-nfs" deleted

kubectl delete pvc pvc-one


persistentvolumeclaim "pvc-one" deleted

kubectl delete pv pvvol-1
persistentvolume "pvvol-1" deleted

2. Create a yaml file for the ResourceQuota object. Set the storage limit to ten claims with a total usage of 500Mi.
cp /home/student/LFS458/SOLUTIONS/s_09/storage-quota.yaml .
vim storage-quota.yaml

storage-quota.yaml
2
4
6
8

apiVersion: v1
kind: ResourceQuota
metadata:
name: storagequota
spec:
hard:
persistentvolumeclaims: "10"
requests.storage: "500Mi"

3. Create a new namespace called small. View the namespace information prior to the new quota. Either the long name
with double dashes --namespace or the nickname ns work for the resource.
kubectl create namespace small
namespace/small created

kubectl describe ns small
Name:
Labels:
Annotations:
Status:

small
<none>
<none>
Active

No resource quota.
No resource limits.

4. Create a new pv and pvc in the small namespace.
kubectl -n small create -f PVol.yaml
persistentvolume/pvvol-1 created

kubectl -n small create -f pvc.yaml
persistentvolumeclaim/pvc-one created


5. Create the new resource quota, placing this object into the small namespace.
kubectl -n small create -f storage-quota.yaml
resourcequota/storagequota created

6. Verify the small namespace has quotas. Compare the output to the same command above.
kubectl describe ns small
Name:
Labels:
Annotations:
Status:

small
<none>
<none>
Active

Resource Quotas
Name:
Resource
-------persistentvolumeclaims
requests.storage

storagequota
Used
Hard
----1
200Mi 500Mi

No resource limits.

7. Remove the namespace line from the nfs-pod.yaml file. Should be around line 11 or so. This will allow us to pass
other namespaces on the command line.
vim nfs-pod.yaml

8. Create the container again.
kubectl -n small create -f nfs-pod.yaml
deployment.apps/nginx-nfs created

9. Determine if the deployment has a running pod.
kubectl -n small get deploy
NAME
nginx-nfs

READY
1/1

UP-TO-DATE

AVAILABLE

AGE
43s

kubectl -n small describe deploy nginx-nfs
<output_omitted>

10. Look to see if the pods are ready.
kubectl -n small get pod
NAME
nginx-nfs-2854978848-g3khf


READY
1/1

STATUS
Running

RESTARTS

AGE
37s


11. Ensure the Pod is running and is using the NFS mounted volume. If you pass the namespace first Tab will auto-complete
the pod name.
kubectl -n small describe pod \
nginx-nfs-2854978848-g3khf
Name:
nginx-nfs-2854978848-g3khf
Namespace:
small
<output_omitted>
Mounts:
/opt from nfs-vol (rw)
<output_omitted>

12. View the quota usage of the namespace
kubectl describe ns small
<output_omitted>
Resource Quotas
Name:
Resource
-------persistentvolumeclaims
requests.storage

storagequota
Used
Hard
----1
200Mi 500Mi

No resource limits.

13. Create a 300M file inside of the /opt/sfw directory on the host and view the quota usage again. Note that with NFS the
size of the share is not counted against the deployment.
sudo dd if=/dev/zero of=/opt/sfw/bigfile bs=1M count=300
300+0 records in
300+0 records out
314572800 bytes (315 MB, 300 MiB) copied, 0.196794 s, 1.6 GB/s

kubectl describe ns small
<output_omitted>
Resource Quotas
Name:
Resource
-------persistentvolumeclaims
requests.storage
<output_omitted>

storagequota
Hard
--1
200Mi
500Mi
Used
---

du -h /opt/
301M
41M
41M
341M


/opt/sfw
/opt/cni/bin
/opt/cni
/opt/


14. Now let us illustrate what happens when a deployment requests more than the quota. Begin by shutting down the
existing deployment.
kubectl -n small get deploy
NAME
nginx-nfs

READY

UP-TO-DATE

AVAILABLE

AGE
11m

kubectl -n small delete deploy nginx-nfs
deployment.apps "nginx-nfs" deleted

15. Once the Pod has shut down view the resource usage of the namespace again. Note the storage did not get cleaned
up when the pod was shut down.
kubectl describe ns small
<output_omitted>
Resource Quotas
Name:
storagequota
Resource
Used
Hard
-----------persistentvolumeclaims 1
requests.storage
200Mi
500Mi

16. Remove the pvc then view the pv it was using. Note the RECLAIM POLICY and STATUS.
kubectl -n small get pvc
NAME
pvc-one

STATUS
Bound

VOLUME
pvvol-1

CAPACITY
1Gi

ACCESSMODES
RWX

STORAGECLASS

AGE
19m

kubectl -n small delete pvc pvc-one
persistentvolumeclaim "pvc-one" deleted

kubectl -n small get pv
NAME
CAPACITY
ACCESSMODES
RECLAIMPOLICY
STATUS
STORAGECLASS
REASON
AGE
pvvol-1 1Gi
RWX
Retain
Released
small/pvc-one 44m

CLAIM

17. Dynamically provisioned storage uses the ReclaimPolicy of the StorageClass which could be Delete, Retain, or
some types allow Recycle. Manually created persistent volumes default to Retain unless set otherwise at creation.
The default storage policy is to retain the storage to allow recovery of any data. To change this begin by viewing the
yaml output.
kubectl get pv/pvvol-1 -o yaml

....


path: /opt/sfw


4
6

server: cp
persistentVolumeReclaimPolicy: Retain
status:
phase: Released

18. Currently we will need to delete and re-create the object. Future development on a deleter plugin is planned. We will
re-create the volume and allow it to use the Retain policy, then change it once running.
kubectl delete pv/pvvol-1
persistentvolume "pvvol-1" deleted

grep Retain PVol.yaml
persistentVolumeReclaimPolicy: Retain

kubectl create -f PVol.yaml
persistentvolume "pvvol-1" created

19. We will use kubectl patch to change the retention policy to Delete. The yaml output from before can be helpful in
getting the correct syntax.
kubectl patch pv pvvol-1 -p \
'{"spec":{"persistentVolumeReclaimPolicy":"Delete"}}'
persistentvolume/pvvol-1 patched

kubectl get pv/pvvol-1
NAME
CAPACITY
ACCESSMODES
STORAGECLASS
REASON
AGE
pvvol-1
1Gi
RWX

RECLAIMPOLICY

STATUS

CLAIM

Delete

Available

2m

20. View the current quota settings.
kubectl describe ns small
....
requests.storage

500Mi

21. Create the pvc again. Even with no pods running, note the resource usage.
kubectl -n small create -f pvc.yaml
persistentvolumeclaim/pvc-one created

kubectl describe ns small


....
requests.storage

200Mi

500Mi

22. Remove the existing quota from the namespace.
kubectl -n small get resourcequota
NAME
storagequota

CREATED AT
2024-08-25T04:10:02Z

kubectl -n small delete resourcequota storagequota
resourcequota "storagequota" deleted

23. Edit the storagequota.yaml file and lower the capacity to 100Mi.
vim storage-quota.yaml

2

....
requests.storage: "100Mi"

24. Create and verify the new storage quota. Note the hard limit has already been exceeded.
kubectl -n small create -f storage-quota.yaml
resourcequota/storagequota created

kubectl describe ns small
....
persistentvolumeclaims
requests.storage

200Mi

100Mi

No resource limits.

25. Create the deployment again. View the deployment. Note there are no errors seen.
kubectl -n small create -f nfs-pod.yaml
deployment.apps/nginx-nfs created

kubectl -n small describe deploy/nginx-nfs
Name:
Namespace:
<output_omitted>

nginx-nfs
small

26. Examine the pods to see if they are actually running.


kubectl -n small get po
NAME
nginx-nfs-2854978848-vb6bh

READY
1/1

STATUS
Running

RESTARTS

AGE
58s

27. As we were able to deploy more pods even with apparent hard quota set, let us test to see if the reclaim of storage takes
place. Remove the deployment and the persistent volume claim.
kubectl -n small delete deploy nginx-nfs
deployment.apps "nginx-nfs" deleted

kubectl -n small delete pvc/pvc-one
persistentvolumeclaim "pvc-one" deleted

28. View if the persistent volume exists. You will see it attempted a removal, but failed. If you look closer you will find the
error has to do with the lack of a deleter volume plugin for NFS. Other storage protocols have a plugin.
kubectl -n small get pv
NAME

CAPACITY
ACCESSMODES
STORAGECLASS
REASON
AGE
pvvol-1
1Gi
RWX
Delete

RECLAIMPOLICY
Failed

STATUS

small/pvc-one

CLAIM
20m

29. Ensure the deployment, pvc and pv are all removed.
kubectl delete pv/pvvol-1
persistentvolume "pvvol-1" deleted

30. Edit the persistent volume YAML file and change the persistentVolumeReclaimPolicy: to Recycle.
vim PVol.yaml

PVol.yaml
2

....
persistentVolumeReclaimPolicy: Recycle
....

31. Add a LimitRange to the namespace and attempt to create the persistent volume and persistent volume claim again.
We can use the LimitRange we used earlier.
kubectl -n small create -f low-resource-range.yaml
limitrange/low-resource-range created

32. View the settings for the namespace. Both quotas and resource limits should be seen.


kubectl describe ns small
<output_omitted>
Resource Limits
Type
Resource Min Max Default Request Default Limit ...
----------- --- --- --------------- ------------- -...
Container cpu
500m
Container memory
100Mi
500Mi
-

33. Create the persistent volume again. View the resource. Note the Reclaim Policy is Recycle.
kubectl -n small create -f PVol.yaml
persistentvolume/pvvol-1 created

kubectl get pv
NAME
pvvol-1

CAPACITY
1Gi

ACCESS MODES
RWX

RECLAIM POLICY
Recycle

STATUS
Available

...
...

34. Attempt to create the persistent volume claim again. The quota only takes effect if there is also a resource limit in effect.
kubectl -n small create -f pvc.yaml
Error from server (Forbidden): error when creating "pvc.yaml":
persistentvolumeclaims "pvc-one" is forbidden: exceeded quota:
storagequota, requested: requests.storage=200Mi, used:
requests.storage=0, limited: requests.storage=100Mi

35. Edit the resourcequota to increase the requests.storage to 500mi.
kubectl -n small edit resourcequota

2
4
6
8

....
spec:
hard:
persistentvolumeclaims: "10"
requests.storage: 500Mi
status:
hard:
persistentvolumeclaims: "10"
....

36. Create the pvc again. It should work this time. Then create the deployment again.
kubectl -n small create -f pvc.yaml
persistentvolumeclaim/pvc-one created

kubectl -n small create -f nfs-pod.yaml


deployment.apps/nginx-nfs created

37. View the namespace settings.
kubectl describe ns small
<output_omitted>

38. Delete the deployment. View the status of the pv and pvc.
kubectl -n small delete deploy nginx-nfs
deployment.apps "nginx-nfs" deleted

kubectl -n small get pvc
NAME
pvc-one

STATUS
Bound

VOLUME
pvvol-1

CAPACITY
1Gi

ACCESS MODES
RWX

STORAGECLASS

AGE
7m

kubectl -n small get pv
NAME
pvvol-1

CAPACITY
1Gi

ACCESS MODES
RWX

RECLAIM POLICY
Recycle

STATUS
Bound

CLAIM
...
small/pvc-one ...

39. Delete the pvc and check the status of the pv. It should show as Available.
kubectl -n small delete pvc pvc-one
persistentvolumeclaim "pvc-one" deleted

kubectl -n small get pv
NAME
CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORA...
pvvol-1 1Gi
RWX
Recycle
Available
...

40. Remove the pv and any other resources created during this lab.
kubectl delete pv pvvol-1
persistentvolume "pvvol-1" deleted


---

## Exercise 9.5: Using StorageClass to Dynamically provision a volume

```bash
cd ~/lfs458/ch09-volumes/
```

StorageClasses in Kubernetes simplify and automate the process of provisioning and managing storage resources, provide
users with the flexibility to choose appropriate storage types for their workloads, and help administrators enforce policies
and manage storage infrastructure more effectively. StorageClasses enables dynamic provisioning of storage resources.
Without StorageClasses, administrators have to manually create PersistentVolumes (PVs) for each PersistentVolumeClaim
(PVC) made by users. With StorageClasses, this process is automated. When a user creates a PVC and specifies a
StorageClasses, the system automatically creates a corresponding PV that meets the requirements.


1. Begin by listing to see if we have any storage class available on our cluster.
kubectl get sc
No resources found

2. We dont have any StorageClass created. Before we can create the sc, we need to deploy the provisioner. Kubernetes
doesn’t include an internal NFS provisioner. We need to use an external provisioner to create a StorageClass for NFS.
Let us deploy a nfs provisioner.
helm repo add nfs-subdir-external-provisioner \
https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
"nfs-subdir-external-provisioner" has been added to your repositories

helm install nfs-subdir-external-provisioner \
nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set nfs.server=cp \
--set nfs.path=/opt/sfw/
NAME: nfs-subdir-external-provisioner
LAST DEPLOYED: Mon Jan 8 12:11:39 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

3. The installation also created a StorageClass for us.
kubectl get sc
NAME

PROVISIONER
ALLOWVOLUMEEXPANSION
AGE
nfs-client
cluster.local/nfs-subdir-external-provisioner
true
12m

RECLAIMPOLICY

VOLUMEBINDINGMODE

Delete

Immediate

4. List to see if there are any PV and PVC available. Clean up in previous lab should have removed all of them.
kubectl get pv,pvc
No resources found

5. Create a YAML file for the new pvc.
cp /home/student/LFS458/SOLUTIONS/s_09/pvc-sc.yaml .
vim pvc-sc.yaml

pvc-sc.yaml
2

apiVersion: v1
kind: PersistentVolumeClaim


4
6
8
10

metadata:
name: pvc-two
spec:
storageClassName: nfs-client
accessModes:
- ReadWriteMany
resources:
requests:
storage: 200Mi

6. Create and verify when the new pvc is created, a dynamic volume is provisioned.
kubectl create -f pvc-sc.yaml
persistentvolumeclaim/pvc-one created

kubectl get pv,pvc
NAME

CAPACITY
POLICY
STATUS
CLAIM
STORAGECLASS
REASON
AGE
persistentvolume/pvc-71149612-33f1-4b18-916d-c67f79aca797
200Mi
Bound
default/pvc-two
nfs-client
28s

ACCESS MODES

RECLAIM

RWX

Delete

NAME

STATUS
ACCESS MODES
STORAGECLASS
AGE
persistentvolumeclaim/pvc-two
Bound
nfs-client
28s

VOLUME

CAPACITY

pvc-71149612-33f1-4b18-916d-c67f79aca797

200Mi

7. Create a new pod to use the pvc.
cp /home/student/LFS458/SOLUTIONS/s_09/pod-sc.yaml .
vim pod-sc.yaml

pod-sc.yaml
2
4
6
8
10
12
14

apiVersion: v1
kind: Pod
metadata:
name: web-server
spec:
containers:
- image: nginx
name: web-container
volumeMounts:
- name: nfs-volume
mountPath: /usr/share/nginx/html
volumes:
- name: nfs-volume
persistentVolumeClaim:
claimName: pvc-two


RWX

8. Create the pod using the file.
kubectl create -f pod-sc.yaml
pod/web-server created

9. Create a new file and copy it inside the pod.
echo "Welcome to the demo of storage class" > index.html
kubectl cp index.html web-server:/usr/share/nginx/html

10. The file was copied on to the default location of the nginx server. Instead of the ephemeral read-write layer of the
container, the file is saved on the NFS server as we have made use of the PV.
ls -l /opt/sfw/default-pvc-two-pvc-<Hit the Tab key>
-rw-rw-r-- 1 student student 37 Jan

8 13:08 index.html

11. Cleanup by deleting the pod,volume claim.
kubectl delete pod/web-server pvc/pvc-two
pod "web-server" deleted
persistentvolumeclaim "pvc-two" deleted


10.1

Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 230

10.2

Accessing Services

10.3

DNS

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 238

10.4

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 240

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 232


10.1


Overview

Overview

• Essential to micro-service architecture
• Connect Pods together, or outside cluster
• Service abstraction
• Load balancing
• Service types
• DNS resolution

As touched on previously Kubernetes architecture is built on the concept of transient, decoupled objects connected together.
Services are the agents which connect Pods together, or provide access outside of the cluster with the idea that any particular
Pod could be terminated and rebuilt. Typically using Labels, the refreshed Pod is connected and the micro-service continues
to provide the expected resource via an Endpoint object. Google has been working on Extensible Service Proxy (ESP),
based off the nginx HTTP reverse proxy server, to provide a more flexible and powerful object than Endpoints, but ESP has
not been adopted much outside of Google App Engine or GKE environments.
There are several different service types, with the flexibility to add more as necessary. Each service can be exposed internally
or externally to the cluster. A service can also connect internal resources to an external resource such as a third-party
database.
The kube-proxy agent watches the Kubernetes API for new services and endpoints being created on each node. It opens
random ports and listens for traffic to the ClusterIPPort:, and redirects the traffic to the randomly generated service endpoints.
Services provide automatic load-balancing, matching a label-query. While there is no configuration of this option there is the
possibility of session affinity via IP. As well a headless service, one without a fixed IP nor load-balancing, can be configured.
Unique IP addresses are assigned, and configured via the etcd database, so that Services implement iptables to route
traffic, but could leverage other technologies to provide access to resources in the future.


10.1. OVERVIEW

Service Update Pattern

• Uses labels to match target Pods
• Rolling Deployments
• Traffic Shift

Labels are used to determine which Pods should receive traffic from a service. As we have learned labels can be dynamically updated for an object, which may affect which Pods continue to connect to a service.
The default update pattern is for a rolling deployment, where new Pods are added, with different versions of an application,
and due to automatic load balancing receive traffic along with previous versions of the application.
Should there be a difference in applications deployed, such that clients would have issues communicating with different
versions you may consider a more specific label for the deployment which includes a version number. When the deployment
creates a new replicaSet for the update the label would not match. Once the new Pods have been created, and perhaps
allowed to fully initialize, we would edit the labels for which the Service connects. Traffic would shift to the new and ready
version, minimizing client version confusion.


10.2


Accessing Services

Accessing an Application With A Service
$ kubectl expose deployment/nginx --port=80 --type=NodePort
$ kubectl get svc
NAME
kubernetes
nginx

TYPE
ClusterIP
NodePort

CLUSTER-IP
10.0.0.1
10.0.0.112

EXTERNAL-IP
<none>
<none>

PORT(S)
443/TCP
80:31230/TCP

AGE
18h
5s

$ kubectl get svc nginx -o yaml
apiVersion: v1
kind: Service
...
spec:
clusterIP: 10.0.0.112
ports:
- nodePort: 31230
...

Open browser http://Public-IP:31230
The basic steps to use kubectl to access a new service. The kubectl expose command created a service for the nginx
deployment. This service used port 80 and generated a random port on all the nodes. A particular port and targetPort can
also be passed during object creation to avoid random values. The targetPort defaults to the port, but could be set to any
value including a string referring to a port on a back-end Pod. Each Pod could have a different port, but traffic is still passed
via the name. Switching traffic to a different port would maintain a client connection, while changing versions of software, for
example.
The kubectl get svc command gave you a list of all the existing services, and we saw the nginx service, which was created
with an internal cluster IP.
The range of cluster IPs and the range of ports used for the random NodePort are configurable in the API server startup
options.
Services can also be used to point to a service in a different Namespace or even a resource outside the cluster such as a
legacy application not yet in Kubernetes.


10.2. ACCESSING SERVICES

Service Types

• ClusterIP
• NodePort
• LoadBalancer
• ExternalName

The ClusterIP service type is the default and only provides access internally (except if manually creating an external endpoint).
The range of ClusterIP used is defined via an API server startup option.
The NodePort type is great for debugging or when a static IP address is necessary, such as opening a particular address
through a firewall. NodePort range is defined in the Cluster configuration).
The LoadBalancer service was created to pass requests to a cloud provider like GKE or AWS. Private cloud solutions also
may implement this service type if there is a Cloud provider plugin such as with CloudStack and OpenStack. Even without
a cloud provider the address is made available to public traffic and packets are spread among the Pods in the deployment
automatically.
A newer service is ExternalName, which is a bit different. It has no selectors, nor does it define ports or endpoints. It allows
the return of an alias to an external service. The redirection happens at the DNS level, not via a proxy or forward. This object
can be useful for services not yet brought into the Kubernetes cluster. A simple change of the type in the future would redirect
traffic to the internal objects.
The kubectl proxy command creates a local service to access a ClusterIP. This can be useful for troubleshooting or
development work.


Service Types (cont)

Figure 10.1: Built in services

While we have talked about three services, some build upon others. A Service is an operator running inside the
kube-controller-manager, which sends API calls via the kube-apiserver to the Network Plugin (such as Cilium) and
the kube-proxy pods running all nodes. The Service operator also creates an Endpoint operator, which queries for the
ephemeral IP addresses of pods with a particular label. These agents work together to manage firewall rules using iptables or
ipvs.
The ClusterIP service configure a persistent IP address and directs traffic sent to that address to the existing pod ephemeral
addresses. This only handles inside the cluster traffic.
When a request for a NodePort is made the operator first creates a ClusterIP. After the ClusterIP has been a high numbered
port is determined and a firewall rules is sent out so that traffic to the high numbered port on any node will be sent to the
persistent IP, which then will be sent to the pod(s).
A LoadBalancer does not create a load balancer. Instead a NodePort and makes an async request to use a load balancer.
If a listener sees the request, as found when using public cloud providers, one would be created. Otherwise the status will
remain Pending as no load balancer has responded to the API call.
An ingress controller a microservice running in a pod, listening to a high port on whichever node the pod may be running,
which will send traffic to a Service based on the URL requested. It is not a built-in service, but is often used with services to
centralize traffic to services. More on an ingress controller is found in a future chapter.


10.2. ACCESSING SERVICES

Services Diagram

Figure 10.2: Service Traffic

The controllers of services and endpoints run inside the kube-controller-manager and send API calls to the kube-apiserver.
API calls are then sent to the network plugin, such as cilium-controller which then communicates with agents on each node,
such as cilium-node. Every kube-proxy is also sent an API call so that it can manage the firewall locally. The firewall is often
iptables or ipvs. The kube-proxy mode is configured via a flag sent during initialization such as mode=iptables and could
also be IPVS or userspace.
In the iptables proxy mode kube-proxy continues to get updates from the API server for changes in Service and Endpoint
objects and updates rules for each object when created or removed.
The graphic above shows two workers each with a replica of MyApp running. A NodePort has been configured which will direct
traffic from port 35001 to the ClusterIP and on to the ephemeral IP of the pod. All nodes use the same firewall rule. As a
result you can connect to any node and Cilium will get the traffic to a node which is running the pod.


Overall Network View

Figure 10.3: Example of Cluster Networking

An example of a multi-container pod with two services sending traffic to its ephemeral IP. The diagram also shows an ingress
controller, which would typically be represented as a pod but has a different shape to show that it is listening to a high number
port of an interface and is sending traffic to a service. Typically the service the ingress controller sends traffic to would be a
ClusterIP, but the diagram shows that it would be possible to send traffic to a NodePort or a LoadBalancer.


10.2. ACCESSING SERVICES

Local Proxy For Development

• Quick way to check service
• Available on localhost
• Not exposed to Internet
$ kubectl proxy
Starting to serve on 127.0.0.1:8001

When developing an application or service, a quick way to check your service is to run a local proxy with kubectl. It will capture
the shell unless you place it in the background. When running you can make calls to the Kubernetes API on localhost and
also reach the verb:ClusterIP: services on their API URL. The IP and port where the proxy listens can be configured with
command arguments.
To access a ghost service using the local proxy we could use this URL: http://localhost:8001/api/v1/namespaces/default/
services/ghost.
If the service port has a name, the path will be http://localhost:8001/api/v1/namespaces/default/services/ghost:⟨port name⟩.


10.3


DNS

DNS

• DNS provided using CoreDNS by default as of v1.13
• Exposed via the kube-dns service.
• Server started for zones served
• Configured via a configmap
• Each loads plugin chains to provide services
– tls
– prometheus
– health
– errors

The use of CoreDNS allows for a great amount of flexibility. Once the container starts it will run a Server for the zones it has
been configured to serve. Then each server can load one or more plugins chains to provide other functionality. As with other
microservices clients would access using a service, kube-dns.
The thirty or so in-tree plugins provide most common functionality, with an easy process to write and enable other plugins as
necessary.
Common plugins can provide metrics for consumption by Prometheus, error logging, health reporting, and tls to configure
certificates for TLS and gRPC servers.
More can be found here: https://coredns.io/plugins/.


10.3. DNS

Verifying DNS Registration

• Verify the service and the pod
• netstat
• dig
• nc
• Network Policy

To make sure that your DNS setup works well and that services get registered, the easiest way to do it is to run pod with a
shell and network tools in the cluster, create a service to connect to the pod, then exec in it to do a DNS lookup.
Troubleshooting of DNS uses typical tools such as nslookup, dig, nc, wireshark and more. The difference is that we leverage
a service to access the DNS server, so we need to check labels and selectors in addition to standard network concerns.
Other steps, similar to any DNS troubleshooting, would be to check the /etc/resolv.conf file of the container as well as
Network Policies and firewalls. We will cover more on Network Policies in the Security chapter.


10.4

Labs


---


---

# Chapter 10: Services
**Working directory: `~/lfs458/ch10-services/`**

## Exercise 10.1: Deploy A New Service

```bash
cd ~/lfs458/ch10-services/
```

Overview
Services (also called microservices) are objects which declare a policy to access a logical set of Pods. They are typically assigned with labels to allow persistent access to a resource, when front or back end containers are terminated
and replaced.
Native applications can use the Endpoints API for access. Non-native applications can use a Virtual IP-based bridge
to access back end pods. ServiceTypes Type could be:
• ClusterIP default - exposes on a cluster-internal IP. Only reachable within cluster
• NodePort Exposes node IP at a static port. A ClusterIP is also automatically created.
• LoadBalancer Exposes service externally using cloud providers load balancer. NodePort and ClusterIP automatically created.
• ExternalName Maps service to contents of externalName using a CNAME record.
We use services as part of decoupling such that any agent or object can be replaced without interruption to access
from client to back end application.
1. Deploy two nginx servers using kubectl and a new .yaml file. The kind should be Deployment and label it with nginx.
Create two replicas and expose port 8080. What follows is a well documented file. There is no need to include the
comments when you create the file. This file can also be found among the other examples in the tarball.
cp /home/student/LFS458/SOLUTIONS/s_10/nginx-one.yaml .
vim nginx-one.yaml

nginx-one.yaml
2
4
6
8
10
12
14
16
18
20

apiVersion: apps/v1
# Determines YAML versioned schema.
kind: Deployment
# Describes the resource defined in this file.
metadata:
name: nginx-one
labels:
system: secondary
# Required string which defines object within namespace.
namespace: accounting
# Existing namespace resource will be deployed into.
spec:
selector:
matchLabels:
system: secondary
# Declaration of the label for the deployment to manage
replicas: 2
# How many Pods of following containers to deploy
template:
metadata:
labels:


23
25
27
29
31
33
35
37
39
41

system: secondary
# Some string meaningful to users, not cluster. Keys
# must be unique for each object. Allows for mapping
# to customer needs.
spec:
containers:
# Array of objects describing containerized application with a Pod.
# Referenced with shorthand spec.template.spec.containers
- image: nginx:1.20.1
# The Docker image to deploy
imagePullPolicy: Always
name: nginx
# Unique name for each container, use local or Docker repo image
ports:
- containerPort: 8080
protocol: TCP
# Optional resources this container may need to function.
nodeSelector:
system: secondOne
# One method of node affinity.

2. View the existing labels on the nodes in the cluster.
kubectl get nodes --show-labels
<output_omitted>

3. Run the following command and look for the errors. Assuming there is no typo, you should have gotten an error about
about the accounting namespace.
kubectl create -f nginx-one.yaml
Error from server (NotFound): error when creating
"nginx-one.yaml": namespaces "accounting" not found

4. Create the namespace and try to create the deployment again. There should be no errors this time.
kubectl create ns accounting
namespace/accounting" created

kubectl create -f nginx-one.yaml
deployment.apps/nginx-one created

5. View the status of the new pods. Note they do not show a Running status.
kubectl -n accounting get pods
NAME
nginx-one-74dd9d578d-fcpmv
nginx-one-74dd9d578d-r2d67


READY
0/1
0/1

STATUS
Pending
Pending

RESTARTS
0

AGE
4m
4m


6. View the node each has been assigned to (or not) and the reason, which shows under events at the end of the output.
kubectl -n accounting describe pod nginx-one-74dd9d578d-fcpmv
Name:
Namespace:
Node:

nginx-one-74dd9d578d-fcpmv
accounting
<none>

<output_omitted>
Events:
Type
Reason
Age
From
....
--------------Warning FailedScheduling <unknown>
default-scheduler
0/2 nodes are available: 2 node(s) didn't match node selector.

7. Label the secondary node. Note the value is case sensitive. Verify the labels.
kubectl label node <worker_node_name> system=secondOne
node/worker labeled

kubectl get nodes --show-labels
NAME
STATUS
ROLES
AGE
VERSION
LABELS
cp
Ready
control-plane
14h
v1.34.1
beta.kubernetes.io/arch=amd64,
beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=cp,
kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,
node.kubernetes.io/exclude-from-external-load-balancers=
worker
Ready
<none>
14h
v1.34.1
beta.kubernetes.io/arch=amd64,
beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker,
kubernetes.io/os=linux

8. View the pods in the accounting namespace. They may still show as Pending. Depending on how long it has been
since you attempted deployment the system may not have checked for the label. If the Pods show Pending after a
minute delete one of the pods. They should both show as Running after a deletion. A change in state will cause the
Deployment controller to check the status of both Pods.


kubectl -n accounting get pods

NAME
nginx-one-74dd9d578d-fcpmv
nginx-one-74dd9d578d-sts5l

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
10m
3s

9. View Pods by the label we set in the YAML file. If you look back the Pods were given a label of app=nginx.
kubectl get pods -l system=secondary --all-namespaces
NAMESPACE
accounting
accounting

NAME
nginx-one-74dd9d578d-fcpmv
nginx-one-74dd9d578d-sts5l

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
20m
9m

10. Recall that we exposed port 8080 in the YAML file. Expose the new deployment.
kubectl -n accounting expose deployment nginx-one


service/nginx-one exposed

11. View the newly exposed endpoints. Note that port 8080 has been exposed on each Pod.
kubectl -n accounting get ep nginx-one
NAME
nginx-one

ENDPOINTS
AGE
192.168.1.72:8080,192.168.1.73:8080
47s

12. Attempt to access the Pod on port 8080, then on port 80. Even though we exposed port 8080 of the container the
application within has not been configured to listen on this port. The nginx server listens on port 80 by default. A curl
command to that port should return the typical welcome page.
curl 192.168.1.72:8080
curl: (7) Failed to connect to 192.168.1.72 port 8080: Connection refused

curl 192.168.1.72:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<output_omitted>

13. Delete the deployment. Edit the YAML file to expose port 80 and create the deployment again.
kubectl -n accounting delete deploy nginx-one
deployment.apps "nginx-one" deleted

vim nginx-one.yaml

nginx-one.yaml

....

3
5

....

ports:
- containerPort: 8080
protocol: TCP

#<-- Edit this line

kubectl create -f nginx-one.yaml
deployment.apps/nginx-one created


---

## Exercise 10.2: Configure a NodePort

```bash
cd ~/lfs458/ch10-services/
```

In a previous exercise we deployed a LoadBalancer which deployed a ClusterIP andNodePort automatically. In this exercise
we will deploy a NodePort. While you can access a container from within the cluster, one can use a NodePort to NAT traffic


from outside the cluster. One reason to deploy a NodePort instead, is that a LoadBalancer is also a load balancer resource
from cloud providers like GKE and AWS.
1. In a previous step we were able to view the nginx page using the internal Pod IP address. Now expose the deployment
using the --type=NodePort. We will also give it an easy to remember name and place it in the accounting namespace.
We could pass the port as well, which could help with opening ports in the firewall.
kubectl -n accounting expose deployment nginx-one --type=NodePort --name=service-lab
service/service-lab exposed

2. View the details of the services in the accounting namespace. We are looking for the autogenerated port.
kubectl -n accounting describe services
....
NodePort:
....

<unset>

32103/TCP

3. Locate the exterior facing hostname or IP address of the cluster. The lab assumes use of GCP nodes, which we access
via a FloatingIP, we will first check the internal only public IP address. Look for the Kubernetes cp URL. Whichever
way you access check access using both the internal and possible external IP address
kubectl cluster-info
Kubernetes control plane is running at https://k8scp:6443
CoreDNS is running at https://k8scp:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

4. Test access to the nginx web server using the combination of cp URL and NodePort.
curl http://k8scp:32103
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>

5. Using the browser on your local system, use the public IP address you use to SSH into your node and the port. You
should still see the nginx default page. You may be able to use curl to locate your public IP address.
curl ifconfig.io
104.198.192.84


---

## Exercise 10.3: Working with CoreDNS

```bash
cd ~/lfs458/ch10-services/
```

1. We can leverage CoreDNS and predictable hostnames instead of IP addresses. A few steps back we created the
service-lab NodePort in the Accounting namespace. We will create a new pod for testing using Ubuntu. The pod
name will be named ubuntu.
cp /home/student/LFS458/SOLUTIONS/s_10/nettool.yaml .


vim nettool.yaml

nettool.yaml
2
4
6
8
10

apiVersion: v1
kind: Pod
metadata:
name: ubuntu
spec:
containers:
- name: ubuntu
image: ubuntu:latest
command: [ "sleep" ]
args: [ "infinity" ]

2. Create the pod and then log into it.
kubectl create -f nettool.yaml
pod/ubuntu created

kubectl exec -it ubuntu -- /bin/bash

On Container
(a) Add some tools for investigating DNS and the network. The installation will ask you the geographic area
and timezone information. Someone in Austin would first answer 2. America, then 37 for Chicago, which
would be central time


apt-get update ; apt-get install curl dnsutils -y

(b) Use the dig command with no options. You should see root name servers, and then information about the
DNS server responding, such as the IP address.
sudo dig
; <<>> DiG 9.16.1-Ubuntu <<>>
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3394
;; flags: qr rd ra; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 1
<output_omitted>
;; Query time: 4 msec
;; SERVER: 10.96.0.10#53(10.96.0.10)
;; WHEN: Thu Aug 27 22:06:18 CDT 2024
;; MSG SIZE rcvd: 431

(c) Also take a look at the /etc/resolv.conf file, which will indicate nameservers and default domains to search
if no using a Fully Qualified Distinguished Name (FQDN). From the output we can see the first entry is
default.svc.cluster.local..
sudo cat /etc/resolv.conf


nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
c.endless-station-188822.internal google.internal
options ndots:5

(d) Use the dig command to view more information about the DNS server. Us the -x argument to get the
FQDN using the IP we know. Notice the domain name, which uses .kube-system.svc.cluster.local.,
to match the pod namespaces instead of default. Also note the name, kube-dns, is the name of a service
not a pod.
sudo dig @10.96.0.10 -x 10.96.0.10
...
;; QUESTION SECTION:
;10.0.96.10.in-addr.arpa.
;; ANSWER SECTION:
10.0.96.10.in-addr.arpa.
IN
PTR

IN

PTR

kube-dns.kube-system.svc.cluster.local.

;; Query time: 0 msec
;; SERVER: 10.96.0.10#53(10.96.0.10)
;; WHEN: Thu Aug 27 23:39:14 CDT 2024
;; MSG SIZE rcvd: 139

(e) Recall the name of the service-lab service we made and the namespaces it was created in. Use this
information to create a FQDN and view the exposed pod.
sudo curl service-lab.accounting.svc.cluster.local.
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
body {
width: 35em;
margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif;
}
...

(f) Attempt to view the default page using just the service name. It should fail as nettool is in the default
namespace.
sudo curl service-lab
curl: (6) Could not resolve host: service-lab

(g) Add the accounting namespaces to the name and try again. Traffic can access a service using a name,
even across different namespaces.
sudo curl service-lab.accounting


<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<output_omitted>

(h) Exit out of the container and look at the services running inside of the kube-system namespace. From the
output we see that the kube-dns service has the DNS serverIP, and exposed ports DNS uses.
sudo exit
kubectl -n kube-system get svc
NAME
kube-dns

TYPE
ClusterIP

CLUSTER-IP
10.96.0.10

EXTERNAL-IP
<none>

PORT(S)
53/UDP,53/TCP,9153/TCP

AGE
42h

3. Examine the service in detail. Among other information notice the selector in use to determine the pods the service
communicates with.
kubectl -n kube-system get svc kube-dns -o yaml
...

...

...

labels:
k8s-app: kube-dns
kubernetes.io/cluster-service: "true"
kubernetes.io/name: CoreDNS
selector:
k8s-app: kube-dns
sessionAffinity: None
type: ClusterIP

4. Find pods with the same labels in all namespaces. We see that infrastructure pods all have this label, including coredns.
kubectl get pod -l k8s-app --all-namespaces
NAMESPACE
kube-system
kube-system
kube-system
kube-system
kube-system
kube-system

NAME
cilium-5tv9d
cilium-gzdk6
coredns-5d78c9869d-44qvq
coredns-5d78c9869d-j6tqx
kube-proxy-lpsmq
kube-proxy-pvl8w

READY
1/1
1/1
1/1
1/1
1/1
1/1

STATUS
Running
Running
Running
Running
Running
Running

RESTARTS
0
0
0

AGE
136m
54m
31m
31m
35m
34m

5. Look at the details of one of the coredns pods. Read through the pod spec and find the image in use as well as any
configuration information. You should find that configuration comes from a configmap.
kubectl -n kube-system get pod coredns-f9fd979d6-4dxpl -o yaml
...
spec:
containers:
- args:
- -conf


...

- /etc/coredns/Corefile
image: k8s.gcr.io/coredns:1.7.0
volumeMounts:
- mountPath: /etc/coredns
name: config-volume
readOnly: true

...
volumes:
- configMap:
defaultMode: 420
items:
- key: Corefile
path: Corefile
name: coredns
name: config-volume
...

6. View the configmaps in the kube-system namespace.
kubectl -n kube-system get configmaps
NAME
cilium-config
coredns
extension-apiserver-authentication
kube-proxy
kubeadm-config
kubelet-config

DATA
1
2
1

AGE
43h
43h
43h
43h
43h
43h

7. View the details of the coredns configmap. Note the cluster.local domain is listed.
kubectl -n kube-system get configmaps coredns -o yaml
apiVersion: v1
data:
Corefile: |
.:53 {
errors
health {
lameduck 5s
}
ready
kubernetes cluster.local in-addr.arpa ip6.arpa {
pods insecure
fallthrough in-addr.arpa ip6.arpa
ttl 30
}
prometheus :9153
forward . /etc/resolv.conf {
max_concurrent 1000
}
cache 30
loop
reload
loadbalance
}
kind: ConfigMap
...


8. It is very important to backup our resources before we make changes to it.
kubectl -n kube-system get configmaps coredns -o yaml > coredns-backup.yaml

9. While there are many options and zone files we could configure, lets start with simple edit. Add a rewrite statement such
that test.io will redirect to cluster.local More about each line can be found at coredns.io.
kubectl -n kube-system edit configmaps coredns
apiVersion: v1
data:
Corefile: |
.:53 {
rewrite name regex (.*)\.test\.io {1}.default.svc.cluster.local
errors
health {
lameduck 5s
}
ready
kubernetes cluster.local in-addr.arpa ip6.arpa {
pods insecure
fallthrough in-addr.arpa ip6.arpa
ttl 30
}
prometheus :9153
forward . /etc/resolv.conf {
max_concurrent 1000
}
cache 30
loop
reload
loadbalance
}

#<-- Add this line

10. Delete the coredns pods causing them to re-read the updated configmap.
kubectl -n kube-system delete pod coredns-f9fd979d6-s4j98 coredns-f9fd979d6-xlpzf
pod "coredns-f9fd979d6-s4j98" deleted
pod "coredns-f9fd979d6-xlpzf" deleted

11. Create a new web server and create a ClusterIP service to verify the address works. Note the new service IP to start
with a reverse lookup.
kubectl create deployment nginx --image=nginx
deployment.apps/nginx created

kubectl expose deployment nginx --type=ClusterIP --port=80
service/nginx expose

kubectl get svc
NAME
kubernetes
nginx


TYPE
ClusterIP
ClusterIP

CLUSTER-IP
10.96.0.1
10.104.248.141

EXTERNAL-IP
<none>
<none>

PORT(S)
443/TCP
80/TCP

AGE
3d15h
7s


12. Log into the ubuntu container and test the URL rewrite starting with the reverse IP resolution.
kubectl exec -it ubuntu -- /bin/bash

On Container
(a) Use the dig command. Note that the service name becomes part of the FQDN.
sudo dig -x 10.104.248.141
....
;; QUESTION SECTION:
;141.248.104.10.in-addr.arpa.
;; ANSWER SECTION:
141.248.104.10.in-addr.arpa.
IN
PTR
....

IN

PTR

nginx.default.svc.cluster.local.

(b) Now that we have the reverse lookup test the forward lookup. The IP should match the one we used in the
previous step.
sudo dig nginx.default.svc.cluster.local.
....
;; QUESTION SECTION:
;nginx.default.svc.cluster.local. IN

A

;; ANSWER SECTION:
nginx.default.svc.cluster.local. 30 IN
....

A

10.104.248.141

(c) Now test to see if the rewrite rule for the test.io domain we added resolves the IP. Note the response
uses the original name, not the requested FQDN.
sudo dig nginx.test.io
....
;; QUESTION SECTION:
;nginx.test.io.
;; ANSWER SECTION:
nginx.default.svc.cluster.local. 30 IN
....

IN

A
A

10.104.248.141

13. Exit out of the container then edit the configmap to add an answer section.
kubectl -n kube-system edit configmaps coredns
....
data:
Corefile: |
.:53 {
rewrite stop {
#<-- Edit this and following two lines
name regex (.*)\.test\.io {1}.default.svc.cluster.local
answer name (.*)\.default\.svc\.cluster\.local {1}.test.io
}


errors
health {
....

14. Delete the coredns pods again to ensure they re-read the updated configmap.
kubectl -n kube-system delete pod coredns-f9fd979d6-fv9qn coredns-f9fd979d6-lnxn5
pod "coredns-f9fd979d6-fv9qn" deleted
pod "coredns-f9fd979d6-lnxn5" deleted

15. Log into the ubuntu container again. This time the response should show the FQDN with the requested FQDN.
kubectl exec -it ubuntu -- /bin/bash

On Container
sudo dig nginx.test.io
....
;; QUESTION SECTION:
;nginx.test.io.
;; ANSWER SECTION:
nginx.test.io.
....

IN

A

IN

A

10.104.248.141

16. Exit then delete the DNS test tools container to recover the resources.
kubectl delete -f nettool.yaml


---

## Exercise 10.4: Use Labels to Manage Resources

```bash
cd ~/lfs458/ch10-services/
```

1. Try to delete all Pods with the system=secondary label, in all namespaces.
kubectl delete pods -l system=secondary \
--all-namespaces
pod "nginx-one-74dd9d578d-fcpmv" deleted
pod "nginx-one-74dd9d578d-sts5l" deleted

2. View the Pods again. New versions of the Pods should be running as the controller responsible for them continues.
kubectl -n accounting get pods
NAME
nginx-one-74dd9d578d-ddt5r
nginx-one-74dd9d578d-hfzml

READY
1/1
1/1

STATUS
Running
Running

RESTARTS
0

AGE
1m
1m

3. We also gave a label to the deployment. View the deployment in the accounting namespace.
kubectl -n accounting get deploy --show-labels


NAME
nginx-one

READY
2/2

UP-TO-DATE

AVAILABLE

AGE
10m

LABELS
system=secondary

4. Delete the deployment using its label.
kubectl -n accounting delete deploy -l system=secondary
deployment.apps "nginx-one" deleted

5. Remove the label from the secondary node. Note that the syntax is a minus sign directly after the key you want to
remove, or system in this case.
kubectl label node worker systemnode/worker unlabeled


11.1

Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 254

11.2

Ingress Controller

11.3

Ingress Rules . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 260

11.4

Service Mesh . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 262

11.5

Limitations . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 263

11.6

Gateway API . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 264

11.7

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 255

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 269


11.1


Overview

Ingress Overview

• Single entrypoint to cluster
• Manage external access to services in cluster
– HTTP
– Load-balancing
– SSL termination
– Name-based virtual hosting

In an earlier chapter we learned about using a Service to expose a containerized application outside of the cluster. We use
Ingress Controllers and Rules to do the same function. The difference is efficiency. Instead of lots of services, such as
LoadBalancer, you can route traffic based on request host or path. This allows for centralization of many services to a single
point.
An Ingress Controller is different than most controllers as it does not run as part of kube-controller-manager binary. You
can deploy multiple controllers, each with unique configurations. A controller uses Ingress Rules to handle traffic to and
from outside the cluster.
There are many ingress controllers such as GKE, nginx, Traefik, Contour, Envoy to name a few. Any tool capable of reverse
proxy should work. These agents consume rules and listen for associated traffic. An Ingress Rule is an API resource that
you can create with kubectl. When you create that resource, it re-programs and re-configures your Ingress Controller to allow
traffic to flow from the outside to an internal service. You can leave a service as a ClusterIP type and define how the traffic
gets routed to that internal service using an Ingress Rule.


11.2. INGRESS CONTROLLER

11.2

Ingress Controller

Ingress Controller
• Allow inbound traffic access to services
• Used instead of NodePort, or other services
• Proxy reconfigured according to rules

Figure 11.1: Ingress Controller for inbound connections

An Ingress Controller is a daemon running in a Pod which watches the /ingresses endpoint on the API Server, which is
found under networking.k8s.io/v1 group, for new objects. When a new endpoint is created the daemon uses the configured
set of rules to allow inbound connection to a service, most often HTTP traffic. This allows easy access to a service through an
edge router to Pods regardless of where the Pod is deployed.
Multiple Ingress controllers can be deployed. Traffic should use annotations to select the proper controller. The lack of a
matching annotation will cause every controller to attempt to satisfy the ingress traffic.


nginx

• Easy integration with RBAC
• Uses the annotation
kubernetes.io/ingress.class: "nginx"

• L7 traffic requires the proxy-real-ip-cidr setting
• Bypasses kube-proxy to allow session affinity
• Does not use conntrack entries for iptables DNAT
• TLS requires host field to be defined

Deployment of an nginx controller has been made easy through the use of provided YAML files which can be found here:
https://github.com/kubernetes/ingress-nginx/tree/master/deploy
This page has configuration files to configure nginx on several platforms such as AWS, GKE, Azure, and bare-metal among
others.
As with any Ingress Controller there are some configuration requirements for proper deployment. Customization can be done
via a ConfigMap, Annotations, or for detailed configuration a Custom template.


11.2. INGRESS CONTROLLER

Google Load Balancer Controller (GLBC)
• Controller called glbc, must be created and started first
• Also create
– Replication Controller with single replica
– Three services for application Pod
– Ingress with two hostnames and three endpoints for each service.
• Backend is group of virtual machine instances, Instance Group
• Multi-pool path:

Global Forwarding Rule -> Target HTTP Proxy -> URL map
-> Backend Service -> Instance Group
• Each pool checks next hop pool
• Not aware of GCE quota settings

There are several objects which need to be created to deploy the GCE Ingress Controller. YAML files are available to make
the process easy. Be aware that several objects would be created for each service, and currently quotas are not evaluated
prior to creation.
Each path for traffic uses a group of like objects referred to as a pool. Each pool regularly checks the next hop up to ensure
connectivity.
Currently the TLS Ingress only supports port 443 and assumes TLS termination. It does not support SNI, only using the first
cert. The TLS secret must contain keys named tls.crt and tls.key


Ingress API Resources

• Now part of networking.k8s.io API
• POST to API server
• Manage like other resources
• Able to expose low-number ports
$ kubectl get ingress
$ kubectl delete ingress <ingress_name>
$ kubectl edit ingress <ingress_name>

Ingress objects now part of the networking.k8s.io API . A typical Ingress object that you can POST to the API server is:

2
4
6
8
10
12
14
16

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: ghost
spec:
rules:
- host: ghost.192.168.99.100.nip.io
http:
paths:
- backend:
service
name: ghost
port:
number: 2368
path: /
pathType: ImplementationSpecific

You can manage Ingress resources like you do pods, deployments, services, etc


11.2. INGRESS CONTROLLER

Deploy Ingress Controller

• Several options available
• Firewall rules may stop traffic
• Default backend serves 404 pages
$ kubectl create -f backend.yaml

To deploy an Ingress Controller, it can be as simple as creating it with kubectl. The source for a sample controller deployment
is available on GitHub: https://github.com/kubernetes/ingress-nginx/tree/master/deploy
The result will be a set of pods managed by a replication controller and some internal services. You will notice a default HTTP
backend which serves 404 pages.
$ kubectl get pods,rc,svc
NAME
READY STATUS
po/default-http-backend-xvep8
1/1
Running
po/nginx-ingress-controller-fkshm 1/1
Running
NAME
DESIRED CURRENT READY
rc/default-http-backend
1
NAME
CLUSTER-IP EXTERNAL-IP
svc/default-http-backend 10.0.0.212 <none>
svc/kubernetes
10.0.0.1
<none>


RESTARTS AGE
4m
4m
AGE
4m
PORT(S) AGE
80/TCP
4m
443/TCP 77d


11.3


Ingress Rules

Creating an Ingress Rule
• Run deployment and expose port
• POST rules entry to controller
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
rules:
- host: ghost.192.168.99.100.nip.io
http:
paths:
- backend:
service
name: ghost
port:
number: 2368
path: /
pathType: ImplementationSpecific

To get exposed with ingress quickly, you can go ahead and try to create a similar rule as mentioned on the previous page.
First, start a Ghost deployment and expose it with an internal ClusterIP service.
$ kubectl run ghost --image=ghost
$ kubectl expose deployments ghost --port=2368

With the deployment exposed and the Ingress rules in place you should be able to access the application from outside the
cluster.


11.3. INGRESS RULES

Multiple Rules
• Multiple Services can use multiple rules
- host: ghost.192.168.99.100.nip.io
http:
paths:
- backend:
service
name: external
port:
number: 80
....
- host: ghost.192.168.99.100.nip.io
http:
paths:
- backend:
service
name: internal
port:
number: 8080
....
On the previous page we defined a single rule. If you have multiple services, you can define multiple rules in the same ingress,
each rule forwarding traffic to a specific service.


11.4

Service Mesh

Intelligent Connected Proxies

Figure 11.2: Istio Service Mesh

For more complex connections or resources such as service discovery, rate limiting, traffic management and advanced metrics
you may want to implement a service mesh.
A service mesh consists of edge and embedded proxies communicating with each other and handling traffic based on rules
from a control plane. Various options are available including Envoy, Istio, and linkerd.
• Envoy - a modular and extensible proxy favored due to modular construction, open architecture and dedication to
remaining un-monetized. Often used as a data plane under other tools of a service mesh. envoyproxy.io
• Istio - a powerful tool set which leverages Envoy proxies via a multi-component control plane. Built to be platform
independent it can be used to make the service mesh flexible and feature filled.
• linkerd - Another service mesh purpose built to be easy to deploy, fast, and ultralight. https://linkerd.io/
a
a Image downloaded from https://istio.io/latest/docs/ops/deployment/architecture/ 05-mar-2025


11.5. LIMITATIONS

11.5

Limitations

Ingress Limitations
• Limited Feature Set - No Native Support to
– TCP/UDP
– gRPC
– Traffic Splitting
• Heavy Reliance on Vendor Specific Annotations
– Authentication
– Rate limiting
– Traffic management
• No True Role Separation
• Minimal Spec
• Frozen Development

The Ingress spec only supports HTTP(S) routing on host/path and TLS termination – no native support for TCP, gRPC, headerbased routing, canary traffic splitting, etc. Many advanced traffic management needs cannot be met without vendor-specific
extensions.
To implement anything beyond the basics, you have to use annotations which are non-standard and vary between controllers.
While you can restrict access to Ingress via RBAC, there’s no built-in way for, say, an admin to manage TLS and a dev to
manage routing in the same ingress – the dev would need access to the whole Ingress spec. Misconfigurations by one team
member can impact others sharing the controller in a multi tenant environment.
The spec is minimal, each controller has to implement additional features its own way. This means you might be locked into a
particular ingress controller if you rely on its unique features.
The Ingress API is considered feature-frozen now; Kubernetes is not expanding it. All new ingress-like capabilities are being developed in Gateway API instead. This means over time Ingress will not keep up with evolving requirements, and its
stagnation could become a liability.


11.6


Gateway API

Gateway API

• Modular Design
– GatewayClass
– Gateway
– HTTPRoute
• Native Support to Advanced Routing
• Enhanced Security
• Extensible Framework
• Future-Proof and Portable

Gateway API has three stable API kinds and belongs to gateway.networking.k8s.io/v1 group.
GatewayClass: Defines a set of gateways with common configuration and managed by a controller that implements the class.
Gateway: Defines an instance of traffic handling infrastructure, such as cloud load balancer.
HTTPRoute: Defines HTTP-specific rules for mapping traffic from a Gateway listener to a representation of backend network
endpoints. These endpoints are often represented as a Service.
Gateway API was built to address Ingress’s shortcomings, so it includes support for L4/L7 protocols, advanced routing (headers, queries), traffic splitting, mirroring, and more, in a standard way. Many things that required custom tweaks in Ingress are
first-class features in Gateway API.
Gateway API is highly extensible through well-defined mechanisms (custom route filters, policies, new route types) without
breaking the API’s consistency. It can evolve with new requirements while maintaining portability.
The separation of Gateway and Routes enables safe multi-tenant usage. Platform admins can maintain control over entry
points and security, while developers can independently manage their routing rules. This delineation reduces the chance of
conflict and aligns with enterprise team structures. It’s effectively a built-in RBAC model for networking.
By codifying what was previously done via annotations, Gateway API provides a vendor-neutral language for ingress and traffic
policy. If your configuration uses Gateway API resources, you should be able to switch between different implementations
(NGINX, HAProxy, Istio, etc.) with minimal changes, since they all adhere to the same spec.
Gateway API introduces security improvements like cross-namespace permission checks (Route binding and ReferenceGrants) and the ability to enforce policies consistently. These give administrators confidence in delegating route control
without compromising on security.


11.6. GATEWAY API

GatewayClass

• Acts as a template
• Implemented by different controllers
• Defines Controller Specification
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
name: example-class
spec:
controllerName: example.com/gateway-controller

The GatewayClass is a key resource in Kubernetes Gateway API, designed to standardize and streamline how gateways
are deployed and managed across a cluster. Unlike the traditional Ingress model, Gateway API introduces a more modular
approach to network traffic management, and GatewayClass is central to that design.
GatewayClass is a cluster-scoped resource that defines a template or blueprint for a category of gateways. It specifies a set
of common properties and behaviors that any Gateway referring to it will inherit. This resource is intended to be created and
maintained by cluster administrators or infrastructure teams, ensuring that gateways are set up according to organizational
standards.
The most critical field in a GatewayClass is spec.controller. This string field indicates the controller (such as an NGINX-based
controller, Istio, or another vendor-specific implementation) responsible for managing the lifecycle and configuration of all
Gateway resources that reference this GatewayClass.
The list of currently supported gateway controllers can be found at https://gateway-api.sigs.k8s.io/implementations/
#gateway-controller-implementation-status


Gateway
• Network Entry Point
• Listener Configuration
• GatewayClass Association
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
name: example-gateway
spec:
gatewayClassName: example-class
listeners:
- name: http
protocol: HTTP
port: 80

A Gateway in Kubernetes serves as the designated entry point for external traffic, orchestrating how requests are received and
processed by defining listener configurations for various protocols and ports. It works in tandem with a GatewayClass, which
specifies the controller that implements the actual networking infrastructure, ensuring a clear separation between infrastructure
management and application-specific routing. This design not only enhances security through features like TLS termination
and cross-namespace policies but also offers extensibility and scalability to meet the complex demands of modern cloud-native
environments.


11.6. GATEWAY API

HTTPRoute

• HTTP-Specific Routing
• Granular Matching
• Advanced Traffic Management

HTTPRoute is a dedicated resource that specifies how HTTP traffic should be handled and directed within a Kubernetes
cluster.
It allows developers to set up precise routing rules based on various HTTP attributes such as hostnames, paths, headers, and
query parameters, enabling advanced traffic management techniques like weighted routing and canary deployments.
By associating HTTPRoutes with one or more Gateways, Kubernetes achieves a clear separation between infrastructure
management and application-level routing, making the overall system more flexible, scalable, and easier to maintain.


HTTPRoute
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
name: example-httproute
spec:
parentRefs:
- name: example-gateway
hostnames:
- "www.example.com"
rules:
- matches:
- path:
type: PathPrefix
value: /login
backendRefs:
- name: example-svc
port: 8080


11.7

Labs


---


---

# Chapter 11: Ingress
**Working directory: `~/lfs458/ch11-ingress/`**

## Exercise 11.1: Service Mesh

```bash
cd ~/lfs458/ch11-ingress/
```


If you have a large number of services to expose outside of the cluster, or to expose a low-number port on the host
node you can deploy an ingress controller. While nginx and GCE have controllers mentioned a lot in Kubernetes.io,
there are many to chose from. Even more functionality and metrics come from the use of a service mesh, such as Istio,
Linkerd, Contour, Aspen, or several others.
1. We will install linkerd using their own scripts. There is quite a bit of output. Instead of showing all of it the output
has been omitted. Look through the output and ensure that everything gets a green check mark. Some steps may
take a few minutes to complete. Each command is listed here to make install easier. As well these steps are in the
setupLinkerd.txt file.
curl -sL run.linkerd.io/install-edge | sh
export PATH=$PATH:/home/student/.linkerd2/bin
echo "export PATH=$PATH:/home/student/.linkerd2/bin" >> $HOME/.bashrc
linkerd check --pre
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/\
releases/download/v1.2.1/standard-install.yaml
linkerd install --crds | kubectl apply -f linkerd install | kubectl apply -f linkerd check
linkerd viz install | kubectl apply -f linkerd viz check
linkerd viz dashboard &

2. By default the GUI is on available on the localhost. We will need to edit the service and the deployment to allow
outside access, in case you are using a cloud provider for the nodes. Edit to remove all characters after equal sign for
-enforced-host, which is around line 59.
kubectl -n linkerd-viz edit deploy web

spec:

3
5
7
9
11

#

13

containers:
- args:
- -linkerd-controller-api-addr=linkerd-controller-api.linkerd.svc.cluster.local:8085
- -linkerd-metrics-api-addr=metrics-api.linkerd-viz.svc.cluster.local:8085
- -cluster-domain=cluster.local
- -grafana-addr=grafana.linkerd-viz.svc.cluster.local:3000
- -controller-namespace=linkerd
- -viz-namespace=linkerd-viz
- -log-level=info
- -enforced-host=
#<-- Comment the line by adding #
image: cr.l5d.io/linkerd/web:stable-2.11.1
imagePullPolicy: IfNotPresent


3. Now edit the http nodePort and type to be a NodePort.
kubectl edit svc web -n linkerd-viz

2
4
6
8
10

....
ports:
- name: http
nodePort: 31500
port: 8084
....
sessionAffinity: None
type: NodePort
status:
loadBalancer: {}
....

#<-- Add line with an easy to remember port

#<-- Edit type to be NodePort

4. Test access using a local browser to your public IP. Your IP will be different than the one shown below.
curl ifconfig.io
104.197.159.20

5. From you local system open a browser and go to the public IP and the high-number nodePort. Be aware the look of
the web page may look slightly different as the software is regularly updated, for example Grafana is not longer fully
integrated.


Figure 11.3: Main Linkerd Page

6. In order for linkerd to pay attention to an object we need to add an annotation. The linkerd inject command will do
this for us. Generate YAML and pipe it to linkerd then pipe again to kubectl. Expect an error about how the object
was created, but the process will work. The command can run on one line if you omit the back-slash. Recreate the
nginx-one deployment we worked with in a previous lab exercise.
kubectl get ns accounting

## Verify namespace exists

kubectl label node worker<TAB> system=secondOne
vim nginx-one.yaml
kubectl apply -f nginx-one.yaml

## Re-label the node

## Validate or correct containerPort: 80 (not 8080)
## Re-deploy nginx-one application

kubectl -n accounting get deploy nginx-one -o yaml | \
linkerd inject - | kubectl apply -f <output_omitted>

7. Check the GUI, you should see that the accounting namespaces and pods are now meshed, and the name is a link.
8. Generate some traffic to the pods, and watch the traffic via the GUI. Use the service-lab service.
kubectl -n accounting get svc
NAME
nginx-one
service-lab


TYPE
ClusterIP
NodePort

CLUSTER-IP
10.107.141.227
10.102.8.205

EXTERNAL-IP
<none>
<none>

PORT(S)
8080/TCP
80:30759/TCP

AGE
5h15m
5h14m


curl 10.102.8.205
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<output_omitted>

Figure 11.4: Now shows meshed

9. Scale up the nginx-one deployment. Generate traffic to get metrics for all the pods.
kubectl -n accounting scale deploy nginx-one --replicas=5
deployment.apps/nginx-one scaled

curl 10.102.8.205

#Several times

10. Explore some of the other information provided by the GUI. Note that the initial view is of the default namespaces.
Change to accounting to see details of the nginx-one deployment.

Figure 11.5: Five meshed pods


---

## Exercise 11.2: Ingress Controller

```bash
cd ~/lfs458/ch11-ingress/
```

We will use the Helm tool we learned about earlier to install an ingress controller.


1. Create two deployments, web-one and web-two, one running nginx. Expose both as ClusterIP services. Use previous
content to determine the steps if you are unfamiliar. Test that both ClusterIPs work before continuing to the next
step.
2. Linkerd does not come with an ingress controller, so we will add one to help manage traffic. We will leverage a Helm
chart to install an ingress controller. Search the hub to find that there are many available.
helm search hub ingress
URL
CHART VERSION
APP VERSION
DESCRIPTION
https://artifacthub.io/packages/helm/k8s-as-hel...
1.0.2
v1.0.0
Helm Chart representing a single Ingress Kubern...
https://artifacthub.io/packages/helm/openstack-...
0.2.1
v0.32.0
OpenStack-Helm Ingress Controller
<output_omitted>
https://artifacthub.io/packages/helm/api/ingres...
3.29.1
0.45.0
Ingress controller for Kubernetes using NGINX a...
https://artifacthub.io/packages/helm/wener/ingr...
3.31.0
0.46.0
Ingress controller for Kubernetes using NGINX a...
https://artifacthub.io/packages/helm/nginx/ngin...
0.9.2
1.11.2
NGINX Ingress Controller
<output_omitted>

3. We will use a popular ingress controller provided by NGINX.
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
"ingress-nginx" has been added to your repositories

helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "ingress-nginx" chart repository
Update Complete. -Happy Helming!-

4. Download and edit the values.yaml file and change it to use a DaemonSet instead of a Deployment. This way there
will be a pod on every node to handle traffic.
helm fetch ingress-nginx/ingress-nginx --untar
cd ingress-nginx
ls
CHANGELOG.md

Chart.yaml

OWNERS

README.md

ci

templates

values.yaml

vim values.yaml

values.yaml
2
4

....
## DaemonSet or Deployment
##
kind: DaemonSet

#<-- Change to DaemonSet, around line 204


7

## Annotations to be added to the controller Deployment or DaemonSet
....

5. Now install the controller using the chart. Note the use of the dot (.) to look in the current directory.
helm install myingress .
NAME: myingress
LAST DEPLOYED: Thu Aug 23 13:47:16 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running
'kubectl --namespace default get services -o wide -w myingress-ingress-nginx-controller'
An example Ingress that makes use of the controller:
<output_omitted>

6. We now have an ingress controller running, but no rules yet. View the resources that exist. Use the -w option to watch
the ingress controller service show up. After it is available use ctrl-c to quit and move to the next command.
kubectl get ingress --all-namespaces
No resources found

kubectl --namespace default get services -o wide

myingress-ingress-nginx-controller

NAME
TYPE
CLUSTER-IP
EXTERNAL-IP
PORT(S)
AGE
SELECTOR
myingress-ingress-nginx-controller
LoadBalancer
10.104.227.79
<pending>
80:32558/TCP,443:30219/TCP
47s
app.kubernetes.io/component=controller,
app.kubernetes.io/instance=myingress,app.kubernetes.io/name=ingress-nginx

kubectl get pod --all-namespaces |grep nginx
default
default
default

myingress-ingress-nginx-controller-mrqt5
myingress-ingress-nginx-controller-pkdxm
nginx-b68dd9f75-h6ww7

1/1
1/1
1/1

Running
Running
Running

7. Now we can add rules which match HTTP headers to services.
cp /home/student/LFS458/SOLUTIONS/s_11/ingress.yaml .
vim ingress.yaml


0

20s
62s
21h


ingress.yaml
2
4
6
8
10
12
14
16
18
20

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: ingress-test
annotations:
nginx.ingress.kubernetes.io/service-upstream: "true"
namespace: default
spec:
ingressClassName: nginx
rules:
- host: www.external.com
http:
paths:
- backend:
service:
name: web-one
port:
number: 80
path: /
pathType: ImplementationSpecific

8. Create then verify the ingress is working. If you don’t pass a matching header you should get a 404 error.
kubectl create -f ingress.yaml
ingress.networking.k8s.io/ingress-test created

kubectl get ingress
NAME
ingress-test

CLASS
nginx

HOSTS
www.external.com

ADDRESS

PORTS

AGE
5s

1/1

Running

8m9s

192.168.219.118

1/1

Running

8m9s

192.168.0.250

kubectl get pod -o wide |grep myingress
myingress-ingress-nginx-controller-mrqt5
cp
<none>
<none>
myingress-ingress-nginx-controller-pkdxm
worker
<none>
<none>

curl 192.168.219.118
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>

9. Check the ingress service and expect another 404 error, don’t use the admission controller.
kubectl get svc |grep ingress


myingress-ingress-nginx-controller
80:32558/TCP,443:30219/TCP
10m
myingress-ingress-nginx-controller-admission
443/TCP
10m

LoadBalancer

10.104.227.79

<pending>

ClusterIP

10.97.132.127

<none>

curl 10.104.227.79
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>

10. Now pass a header which matches a URL to one of the services we exposed in an earlier step. You should see the
default nginx or httpd web server page.
curl -H "Host: www.external.com" http://10.104.227.79
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
<output_omitted>

11. We can add an annotation to the ingress pods for Linkerd. You will get some warnings, but the command will work.
kubectl get ds myingress-ingress-nginx-controller -o yaml |\
linkerd inject --ingress - | kubectl apply -f daemonset "myingress-ingress-nginx-controller" injected
Warning: resource daemonsets/myingress-ingress-nginx-controller is missing the
kubectl.kubernetes.io/last-applied-configuration annotation which is required
by kubectl apply. kubectl apply should only be used on resources created
declaratively by either kubectl create --save-config or kubectl apply. The
missing annotation will be patched automatically.
daemonset.apps/myingress-ingress-nginx-controller configured

to
the
Top
page,
change
the
namespace
to
default
and
the
resource
to
daemonset/myingress-ingress-nginx-controller. Press start then pass more traffic to the ingress controller and
view traffic metrics via the GUI. Let top run so we can see another page added in an upcoming step.

12. Go


Figure 11.6: Ingress Traffic

13. At this point we would keep adding more and more servers. We’ll configure one more, which would then could be a
process continued as many times as desired.
Customize the web-two (or whichever deployment is running nginx) welcome page. Run a bash shell inside the web-two
pod. Your pod name will end differently. Install vim or an editor inside the container then edit the index.html file of
nginx so that the title of the web page will be Internal Welcome Page. Much of the command output is not shown
below.
kubectl exec -it web-two-<Tab> -- /bin/bash

On Container
root@web-two-...-:/# apt-get update
root@web-two-...-:/# apt-get install vim -y
root@web-two-...-:/# vim /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<head>
<title>Internal Welcome Page</title>
<style>
<output_omitted>

#<-- Edit this line

exit

Edit the ingress rules to point the thirdpage service. It may be easiest to copy the existing host stanza and edit the host
and name.
14. kubectl edit ingress ingress-test


ingress-test
2
4
6
8
10
12
14
16
18
20
22
24

....
spec:
rules:
- host: internal.org
http:
paths:
- backend:
service:
name: web-two
port:
number: 80
path: /
pathType: ImplementationSpecific
- host: www.external.com
http:
paths:
- backend:
service:
name: web-one
port:
number: 80
path: /
pathType: ImplementationSpecific
status:
....

15. Test the second Host: setting using curl locally as well as from a remote system, be sure the <title> shows the
non-default page. Use the main IP of either node. The Linkerd GUI should show a new TO line, if you select the small
blue box with an arrow you will see the traffic is going to internal.org.
curl -H "Host: internal.org" http://10.128.0.7/
<!DOCTYPE html>
<html>
<head>
<title>Internal Welcome Page</title>
<style>
<output_omitted>


Figure 11.7: Linkerd Top Metrics


---

## Exercise 11.3: Gateway API

```bash
cd ~/lfs458/ch11-ingress/
```

1. First step is to Deploy NGINX Gateway Fabric, To install the Gateway API resources, execute the following command
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/\
config/crd/gateway-api/standard?ref=v1.6.1" | kubectl apply -f customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/grpcroutes.gateway.networking.k8s.io configured
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io configured
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created

2. Deploy the NGINX Gateway Fabric CRDs by executing the following command
kubectl apply -f https://raw.githubusercontent.com/nginx/\
nginx-gateway-fabric/v1.6.1/deploy/crds.yaml
customresourcedefinition.apiextensions.k8s.io/clientsettingspolicies.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/nginxgateways.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/nginxproxies.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/observabilitypolicies.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/snippetsfilters.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/upstreamsettingspolicies.gateway.nginx.org created

3. Deploy the NGINX Gateway Fabric by executing the following command
kubectl apply -f https://raw.githubusercontent.com/nginx/\
nginx-gateway-fabric/v1.6.1/deploy/default/deploy.yaml
namespace/nginx-gateway created
serviceaccount/nginx-gateway created
clusterrole.rbac.authorization.k8s.io/nginx-gateway created
clusterrolebinding.rbac.authorization.k8s.io/nginx-gateway created


configmap/nginx-includes-bootstrap created
service/nginx-gateway created
deployment.apps/nginx-gateway created
gatewayclass.gateway.networking.k8s.io/nginx created
nginxgateway.gateway.nginx.org/nginx-gateway-config created

4. Verify the NGINX Gateway Fabric is running by executing the following command
kubectl get all -n nginx-gateway
NAME
pod/nginx-gateway-7c59cd4cc6-5m84b
NAME

AGE
service/nginx-gateway
92s

READY
2/2

STATUS
Running

RESTARTS

AGE
92s

TYPE

CLUSTER-IP

EXTERNAL-IP

PORT(S)

LoadBalancer

10.111.70.106

<pending>

80:32500/TCP,443:32730/TCP

NAME
deployment.apps/nginx-gateway

READY
1/1

NAME
replicaset.apps/nginx-gateway-7c59cd4cc6

UP-TO-DATE
DESIRED

AVAILABLE

AGE
92s

CURRENT

READY

AGE
92s

5. The NGINX Gateway Fabric service is of the type of LoadBalancer service. The external IP is pending, Kubernetes will
randomly allocate two ports on every node of the cluster. To access the NGINX Gateway Fabric, use an IP address
of any node of the cluster along with the two allocated ports. For simplicty sake, we can patch the service to be of
NodePort.
kubectl patch service/nginx-gateway -n nginx-gateway -p '{"spec": {"type": "NodePort"}}'
kubectl get service/nginx-gateway -n nginx-gateway
service/nginx-gateway patched
NAME
TYPE
CLUSTER-IP
nginx-gateway
NodePort
10.111.70.106

EXTERNAL-IP
<none>

PORT(S)
80:32500/TCP,443:32730/TCP

AGE
11m

6. Deploy an application and create a service named books. Then, define two Gateway API resources—a gateway and
an HTTPRoute. These components work together to establish a routing rule that captures all HTTP requests for the
hostname shop.example.com and directs them to the books service.
cp /home/student/LFS458/SOLUTIONS/s_11/books.yaml .
vim books.yaml

books.yaml
2
4

apiVersion: apps/v1
kind: Deployment
metadata:
name: books
spec:


7
9
11
13
15
17
19
21
23
25
27
29
31

replicas: 2
selector:
matchLabels:
app: books
template:
metadata:
labels:
app: books
spec:
containers:
- name: books
image: nginx
ports:
- containerPort: 80
--apiVersion: v1
kind: Service
metadata:
name: books
spec:
ports:
- port: 80
targetPort: 80
protocol: TCP
name: http
selector:
app: books

kubectl create -f books.yaml
deployment.apps/books created
service/books created

7. Verify the books delployment and service are created and running fine
kubectl get svc/books deploy/books
NAME
service/books

TYPE
ClusterIP

NAME
deployment.apps/books

CLUSTER-IP
10.111.164.248

READY
2/2

UP-TO-DATE

EXTERNAL-IP
<none>
AVAILABLE

PORT(S)
80/TCP

AGE
2m57s

AGE
2m57s

8. To route traffic to the books application, we will create a gateway and HTTPRoute. We need a gateway to create an
entry point for HTTP traffic coming into the cluster. The shop gateway we are going to create will open an entry point to
the cluster on port 80 for HTTP traffic.
cp /home/student/LFS458/SOLUTIONS/s_11/gateway.yaml .
vim gateway.yaml


gateway.yaml
2
4
6
8
10

apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
name: shop
spec:
gatewayClassName: nginx
listeners:
- name: http
port: 80
protocol: HTTP

kubectl create -f gateway.yaml
gateway.gateway.networking.k8s.io/shop created

9. Verify the shop Gateway API resource has been deployed.
kubectl get gateway
NAME
shop

CLASS
nginx

ADDRESS

PROGRAMMED
True

AGE
2m9s

10. To route HTTP traffic from the gateway to the books service, we need to create an HTTPRoute named books and
attach it to the gateway. This HTTPRoute will have a single routing rule that routes all traffic to the hostname
“shop.example.com” from the gateway to the books service. Once NGINX Gateway Fabric processes the shop gateway
and books HTTPRoute, it will configure its data plane (NGINX) to route all HTTP requests sent to “shop.example.com”
to the pods that the books service targets.
cp /home/student/LFS458/SOLUTIONS/s_11/httproute.yaml .
vim httproute.yaml

httproute.yaml
2
4
6
8
10
12
14
16

apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
name: books
spec:
parentRefs:
- name: shop
hostnames:
- "shop.example.com"
rules:
- matches:
- path:
type: PathPrefix
value: /
backendRefs:
- name: books
port: 80


kubectl create -f httproute.yaml
httproute.gateway.networking.k8s.io/books created

11. Verify the httproute has been deployed.
kubectl get httproute
NAME
books

HOSTNAMES
["shop.example.com"]

AGE
35s

12. The Gateway API and HTTPRoute have been deployed succesfully, test the configuration by sending a request to the
Node IP - i.e. private ip, which can be obtained by command hostname -i and node port of the NGINX gateway
Fabric. First, Lets send a request to the path ’/’. Since the shop HTTPRoute routes all traffic on any path to the books
application, the following requests should also be handled by the books pods.
curl --resolve shop.example.com:32500:10.2.0.30 http://shop.example.com:32500/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
...
<output_omitted>

13. Requests to hostnames other than “shop.example.com” should not be routed to the books application, since the books
HTTPRoute only matches requests with the “shop.example.com” hostname. To verify this, send a request to the hostname “test.example.com”:
curl --resolve test.example.com:32500:10.2.0.30 http://test.example.com:32500/
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>


12.1

Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 286

12.2

Scheduler Settings . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 287

12.3

Pod Specification . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 292

12.4

Affinity Rules . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 294

12.5

Taints and Tolerations . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 299

12.6

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 302


12.1


Overview

kube-scheduler
• Topology-aware algorithm to determine Pod placement
• Pod priority and preemption
• Labels used for each method
• podAffinity label encourage Pod on specific nodes
• Avoid via podAntiAffinity
• Node taints to repel specific Pods
• Pod tolerations of node taints
• nodeSelector to force specific Pods
• Require, prefer and evict

The larger and more diverse a Kubernetes deployment becomes the more administration of scheduling can be important. The
kube-scheduler determines which nodes will run a Pod.
Users can set the priority of a pod, which will allow preemption of lower priority pods. The eviction of lower priority pods would
then allow the higher priority pod to be scheduled.
The scheduler tracks the set of nodes in your cluster, filters and scores to determine on which node each Pod should be
scheduled. The Pod spec as part of a request is sent to the kubelet on the node for creation.
The default scheduling decision can be affected through the use of Labels on nodes or Pods. Labels of podAffinity, taints,
and pod bindings allow for configuration from the Pod or the node perspective. Some like tolerations allow a Pod to work
with a Node, even when the Node has a taint that would otherwise preclude a Pod being scheduled.
Not all labels are drastic. Affinity settings may encourage a Pod to be deployed on a node, but would deploy the Pod elsewhere
if the node was not available. Sometimes documentation may use the term require, but practice shows the setting to be more
of a request. As beta features expect the specifics to change. Some settings will evict Pods from a node should the required
condition no longer be true such as requiredDuringSchedulingRequiredDuringExecution.
Others options, like a custom scheduler, need to be programmed and deployed into your Kubernetes cluster.


12.2. SCHEDULER SETTINGS

12.2

Scheduler Settings

Node selection in kube-scheduler

• Filtering
• Scoring

The Filtering stage, identifies the set of Nodes where the Pod can be scheduled.
For example, the PodFitsResources filter determines whether a prospective Node has sufficient resources available to satisfy
a Pod’s particular resource requirements (requests & limits). Any appropriate Nodes that were found after this phase are
included in the node list. It is not possible to schedule that pod if the list is empty, it remains unscheduled.
The Scoring stage, the scheduler rates the remaining nodes to determine the best Pod placement. Each Node that made it
through filtering is given a score by the scheduler, which is based on the default scheduler configuration.
The Pod is given to the Node with the highest ranking by kube-scheduler.
Note: The filtering and scoring behavior of the scheduler can be configured using scheduling configuration profiles.


Scheduling Configuration

• Customize the behavior
• Scheduling Profiles
– Extension points
– Scheduling plugins
– Multiple profiles

You can customize the behavior of the kube-scheduler by writing a configuration file and passing its path as a command
line argument. A scheduling Profile allows you to configure the different stages of scheduling in the kube-scheduler. Each
stage is exposed in an extension point. Plugins provide scheduling behaviors by implementing one or more of these extension
points.
We can configure the scheduler via the use of scheduling profiles. These profiles allow the configuration of extension points
at which plugins can be used.
An extension point is one of the twelve stages of scheduling, at which point a plugin can be used to modify how that state of
a scheduler works


12.2. SCHEDULER SETTINGS

Extension Points

• queueSort

• reserve

• preFilter

• permit

• filter

• preBind

• postFilter

• bind

• preScore

• postBind

• score

• multiPoint

queueSort: These plugins provide an ordering function that is used to sort pending Pods in the scheduling queue. Exactly
one queue sort plugin may be enabled at a time.
preFilter: These plugins are used to pre-process or check information about a Pod or the cluster before filtering. They can
mark a pod as unschedulable.
filter: These plugins are the equivalent of Predicates in a scheduling Policy and are used to filter out nodes that can not run
the Pod. Filters are called in the configured order. A pod is marked as unschedulable if no nodes pass all the filters.
postFilter: These plugins are called in their configured order when no feasible nodes were found for the pod. If any postFilter
plugin marks the Pod schedulable, the remaining plugins are not called.
preScore: This is an informational extension point that can be used for doing pre-scoring work.
score: These plugins provide a score to each node that has passed the filtering phase. The scheduler will then select the
node with the highest weighted scores sum.
More information can be found here: https://kubernetes.io/docs/reference/scheduling/config/#profiles


Scheduling plugins
• ImageLocality

• VolumeRestrictions

• TaintToleration

• VolumeZone

• NodeName

• NodeVolumeLimits

• NodePorts

• EBSLimits

• NodeAffinity

• GCEPDLimits

• PodTopologySpread

• AzureDiskLimits

• NodeUnschedulable

• InterPodAffinity

• NodeResourcesFit

• PrioritySort

• NodeResourcesBalancedAllocation • DefaultBinder
• VolumeBinding

• DefaultPreemption

The following plugins, enabled by default, implement one or more of these extension points
ImageLocality: Favors nodes that already have the container images that the Pod runs. Extension points: score.
TaintToleration: Implements taints and tolerations. Implements extension points: filter, preScore, score.
NodeName: Checks if a Pod spec node name matches the current node. Extension points: filter.
NodePorts: Checks if a node has free ports for the requested Pod ports. Extension points: preFilter, filter.
NodeAffinity: Implements node selectors and node affinity. Extension points: filter, score.
PodTopologySpread: Implements Pod topology spread. Extension points: preFilter, filter, preScore, score.
NodeUnschedulable: Filters out nodes that have .spec.unschedulable set to true. Extension points: filter.
NodeResourcesFit: Checks if the node has all the resources that the Pod is requesting.
NodeResourcesBalancedAllocation: Favors nodes that would obtain a more balanced resource usage if the Pod is scheduled there. Extension points: score.
More information can be found here: https://kubernetes.io/docs/reference/scheduling/config/#scheduling-plugins


12.2. SCHEDULER SETTINGS

Multiple profiles
• Configure more than one profile
– Example config
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: default-scheduler
- schedulerName: custom-scheduler
plugins:
preFilter:
disabled:
- name: '*'
filter:
disabled:
- name: '*'
postFilter:
disabled:
- name: '*'

kube-scheduler can be configured to run more than one profile. Each profile should have an associated scheduler name and
also can run a different set of plugins.
In the above example, the scheduler will run with two profiles: one with the default plugins and one with all filtering plugins
disabled.
Pods that want to be scheduled according to a specific profile can include the corresponding scheduler name in its
.spec.schedulerName.
By default, one profile with the scheduler name default-scheduler is created. This profile includes the default plugins described above. When declaring more than one profile, a unique scheduler name for each of them is required.
Note: If a scheduler name is not specified in the pod spec, kube-apiserver will set it to default-scheduler. Therefore, a profile
with this scheduler name should exist to get those pods scheduled.


12.3


Pod Specification

Pod Specification

• Pod Specification fields
– nodeName
– nodeSelector
– affinity
– tolerations
– schedulerName

Most scheduling decisions can be made as part of the Podspec. The nodeName and nodeSelector options allow a Pod to be
assigned to a single node or a group of nodes with particular labels.
Affinity and anti-affinity can be used to require or prefer which node is used by the scheduler. If using a preference instead a
matching node is chosen first, but other nodes would be used if no match is present.
The use of taints allows a node to be labeled such that Pods would not be scheduled for some reason, such as the cp node
after initialization. A toleration allows a Pod to ignore the taint and be scheduled assuming other requirements are met.
Should none of these options meet the needs of the cluster there is also the ability to deploy a custom scheduler. Each Pod
could then include a schedulerName to choose which schedule to use.


12.3. POD SPECIFICATION

Specifying Node Label

• Match a label with nodeSelector
• Pod remains in Pending state until a suitable Node is found
spec:
containers:
- name: redis
image: redis
nodeSelector:
net: fast

The nodeSelector field in a pod specification provides a straightforward way to target a node or set of nodes, using one or
more key-value pairs.
Setting the nodeSelector tells the scheduler to place the pod on a node that matches the labels. All listed selectors must be
met, but the node could have more labels. In the above example any node with a key of net set to fast would be a candidate
for scheduling. Remember that labels are admin created tags, with no tie to actual resources. This node could have a slow
network.
The pod would remain Pending until a node is found with the matching labels.
The use of affinity/anti-affinity should be able to express every feature as nodeSelector.


12.4


Affinity Rules

Pod Affinity Rules

• Uses In, NotIn, Exists, and DoesNotExist operators
• requiredDuringSchedulingIgnoredDuringExecution
• preferredDuringSchedulingIgnoredDuringExecution
• affinity
– podAffinity
– podAntiAffinity

Pods which may communicate a lot or share data may operate best if co-located, which would be a form of affinity. For greater
fault tolerance you may want Pods to be as separate as possible, which would be anti-affinity. These settings are used by the
scheduler based on labels of Pods already running. As a result the scheduler must interrogate each node and track the labels
of running Pods. Clusters larger than several hundred nodes may see significant performance loss.
The use of requiredDuringSchedulingIgnoredDuringExecution means that the Pod will not be scheduled on a node
unless the following operator is true. If the operator changes to become false in the future the Pod will continue to run. This
could be seen as a hard rule.
Similar is preferredDuringSchedulingIgnoredDuringExecution which will choose a node with the desired setting before
those without. Should no properly labeled nodes be available the Pod will execute anyway. This is more of a soft settings
which declares a preference instead of a requirement.
With the use of podAffinity the scheduler will try to schedule Pods together. The use of podAntiAffinity would cause the
scheduler to keep Pods on different nodes.


12.4. AFFINITY RULES

podAffinity Example

spec:
affinity:
podAffinity:
requiredDuringSchedulingIgnoredDuringExecution:
- labelSelector:
matchExpressions:
- key: security
operator: In
values:
- S1

An example of affinity and podAffinity settings. This also requires a particular label to be matched when the Pod starts,
but not required if the label is later removed.
The Pod can be scheduled on a node running a Pod with a key label of security and a value of S1. If this requirement is not
met the Pod will remain in a Pending state.


podAntiAffinity Example

podAntiAffinity:
preferredDuringSchedulingIgnoredDuringExecution:
- weight: 100
podAffinityTerm:
labelSelector:
matchExpressions:
- key: security
operator: In
values:
- S2

With podAntiAffinity we can prefer to avoid nodes with a particular label. In this case the scheduler will prefer to avoid a
node running a pod that has a key label of security and value of S2.
In a large, varied, environment there may be multiple situations to be avoided. As a preference this settings tries to avoid
certain labels, but will still schedule the Pod on some node. As the Pod will still run we can provide a weight to a particular
rule. The weights can be declared in a value from 1 to 100. The scheduler then tries to choose, or avoid, the node with the
greatest combined value.


12.4. AFFINITY RULES

Node Affinity Rules

• Similar to Pod affinity rules, declared with affinity
• Uses In, NotIn, Exists, and DoesNotExist operators
• requiredDuringSchedulingIgnoredDuringExecution
• preferredDuringSchedulingIgnoredDuringExecution
• Planned for future
– requiredDuringSchedulingRequiredDuringExecution

Where Pod affinity/anti-affinity has to do with other Pods, the use of nodeAffinity allows Pod scheduling based of node
labels. This is similar, and will some day replace, use of the nodeSelector setting. The scheduler will not look at other Pods
on the system, but the labels of the nodes. This should have much less performance impact on the cluster, even with a large
number of nodes.
Until nodeSelector has been fully deprecated both the selector and required labels must be met for a Pod to be scheduled.


Node Affinity Example

spec:
affinity:
nodeAffinity:
preferredDuringSchedulingIgnoredDuringExecution:
- weight: 1
preference:
matchExpressions:
- key: diskspeed
operator: In
values:
- quick
- fast

The nodeAffinity prefers a node with the following rule, but the pod would be scheduled even if there were no matching
nodes. The rule gives extra weight to nodes with a key of diskspeed with a value of fast or quick.


12.5. TAINTS AND TOLERATIONS

12.5

Taints and Tolerations

Taint

• Expressed as key=value:effect
• key and value value created by admin
• Effect must be NoSchedule, PreferNoSchedule, or NoExecute
• Only nodes can be tainted currently
• Multiple taints possible on a node

A node with a particular taint will repel Pods without tolerations for that taint.
The key and value used can be any legal string, while allows flexibility to prevent Pods from running on nodes based off of
any need. If a Pod does not have an existing toleration the scheduler will not consider the tainted node.
There are three effects, or ways to handle Pod scheduling.
• NoSchedule The scheduler will not schedule a Pod on this node unless the Pod has this toleration. Existing Pods
continue to run, regardless of toleration.
• PreferNoSchedule The scheduler will avoid using this node unless there are no untainted nodes for the Pods
toleration. Existing Pods are unaffected.
• NoExecute This taint will cause existing Pods to be evacuated and no future Pods scheduled. Should an existing Pod
have a toleration it will continue to run. If the Pod tolerationSeconds value is set the Pod will remain for that many
seconds then be evicted. Certain node issues will cause kubelet to add 300 second tolerations to avoid unnecessary
evictions.
If a node has multiple taints the scheduler ignores those with matching tolerations. The remaining un-ignored taints
have their typical effect.
The use of TaintBasedEvictions is still an alpha feature. The kubelet uses taints to rate-limit evictions when the node
has problems.


Tolerations
• Pod setting to run on tainted nodes
• Same effects as node taints
• Two operators
– Exists
– Equal which requires value
tolerations:
- key: "server"
operator: "Equal"
value: "ap-east"
effect: "NoExecute"
tolerationSeconds: 3600

Setting tolerations on a node are used to schedule on tainted nodes. This provides an easy way to avoid Pods using the
node. Only those with a particular toleration would be scheduled.
An operator can be included in a Pod spec, defaulting to Equal if not declared. The use of operator Equal requires a value to
match. The Exists operator should not be specified. If an empty key uses the Exists operator it will tolerate every taint. If
there is no effect, but a key and operator are declared all effects are matched with the declared key.
In the above example the Pod will remain on the server with a key of server and value of ap-east for 3600 seconds after
the node has been tainted with NoExecute. When the time runs out the Pod will be evicted.


12.5. TAINTS AND TOLERATIONS

Custom Scheduler

• Create and deploy custom scheduler as a container
• Run multiple schedulers simultaneously
• Pods can declare which scheduler to use
• Scheduler must be running or Pod remains Pending
• View scheduler and other information with kubectl get events

If the default scheduling mechanisms are not flexible enough for your needs you can write your own scheduler. The programming of a custom scheduler is outside the scope of this course, but you may want to start with the existing scheduler code
which can be found here: https://github.com/kubernetes/kubernetes/tree/master/pkg/scheduler
If a Pod spec does not declare which scheduler to use the standard scheduler is used by default. If the Pod declares a
scheduler and that container is not running the Pod would remain in Pending state forever.
The end result of the scheduling process is that a pod gets a binding that specifies which node it should run on. A binding is
a Kubernetes API primitive in the api/v1 group. Technically without any scheduler running, you could still schedule a pod on a
node, by specifying a binding for that pod.


12.6

Labs


---


---

# Chapter 12: Scheduling
**Working directory: `~/lfs458/ch12-scheduling/`**

## Exercise 12.1: Assign Pods Using Labels

```bash
cd ~/lfs458/ch12-scheduling/
```

Overview
While allowing the system to distribute Pods on your behalf is typically the best route, you may want to determine which
nodes a Pod will use. For example you may have particular hardware requirements to meet for the workload. You may
want to assign VIP Pods to new, faster hardware and everyone else to older hardware.
In this exercise we will use labels to schedule Pods to a particular node. Then we will explore taints to have more
flexible deployment in a large environment.
1. Begin by getting a list of the nodes. They should be in the ready state and without added labels or taints.
kubectl get nodes
NAME
cp
worker

STATUS
Ready
Ready

ROLES
control-plane
<none>

AGE
44h
43h

VERSION
v1.34.1
v1.34.1

2. View the current labels and taints for the nodes.
kubectl describe nodes |grep -A5 -i label
Labels:

-Labels:

beta.kubernetes.io/arch=amd64
beta.kubernetes.io/os=linux
kubernetes.io/arch=amd64
kubernetes.io/hostname=scp
kubernetes.io/os=linux
node-role.kubernetes.io/control-plane=
beta.kubernetes.io/arch=amd64
beta.kubernetes.io/os=linux
kubernetes.io/arch=amd64
kubernetes.io/hostname=worker
kubernetes.io/os=linux
system=secondOne

kubectl describe nodes |grep -i taint
Taints:
Taints:

<none>
<none>

3. Get a count of how many containers are running on both the cp and worker nodes. There are about 24 containers
running on the cp in the following example, and eight running on the worker. There are status lines which increase
the wc count. You may have more or less, depending on previous labs and cleaning up of resources. Take note of
the number of containers, and then notice the numbers change due to scheduling. The change between nodes is the
important information, not the particular number. If you are using cri-o you can view containers using crictl ps.
kubectl get deployments --all-namespaces
NAMESPACE
accounting
default
default
default


NAME
nginx-one
anotherweb-apache
web-one
web-two

READY
1/1
1/1
1/1
1/1

UP-TO-DATE
1
1

AVAILABLE
1
1

AGE
19h
8h
45m
45m


kube-system
cilium-operator
<output_omitted>

1/1


35h

sudo crictl ps | wc -l

sudo crictl ps | wc -l

4. For the purpose of the exercise we will assign the cp node to be VIP hardware and the secondary node to be for others.
kubectl label nodes cp status=vip
node/cp labeled

kubectl label nodes worker status=other
node/worker labeled

5. Verify your settings. You will also find there are some built in labels such as hostname, os and architecture type. The
output below appears on multiple lines for readability.
kubectl get nodes --show-labels
NAME
STATUS
ROLES
AGE
VERSION
LABELS
cp
Ready
control-plane
35h
v1.34.1
beta.kubernetes.io/arch=amd64,
beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=cp,
kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,
node.kubernetes.io/exclude-from-external-load-balancers=,status=vip
worker
Ready
<none>
35h
v1.34.1
beta.kubernetes.io/arch=amd64,
beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker,
kubernetes.io/os=linux,status=other,system=secondOne

6. Create vip.yaml to spawn four busybox containers which sleep the whole time. Include the nodeSelector entry.
cp /home/student/LFS458/SOLUTIONS/s_12/vip.yaml .
vim vip.yaml

vip.yaml
2
4
6
8

apiVersion: v1
kind: Pod
metadata:
name: vip
spec:
containers:
- name: vip1
image: busybox


args:
- sleep
- "1000000"
- name: vip2
image: busybox
args:
- sleep
- "1000000"
- name: vip3
image: busybox
args:
- sleep
- "1000000"
- name: vip4
image: busybox
args:
- sleep
- "1000000"
nodeSelector:
status: vip

10
12
14
16
18
20
22
24
26
28

7. Deploy the new pod. Verify the containers have been created on the cp node. It may take a few seconds for all the
containers to spawn. Check both the cp and the secondary nodes. From this point forward use crictl where the step
lists docker if you have deployed your cluster with cri-o.
kubectl create -f vip.yaml
pod/vip created

sudo crictl ps |wc -l

sudo crictl ps |wc -l

8. Delete the pod then edit the file, commenting out the nodeSelector lines. It may take a while for the containers to fully
terminate.
kubectl delete pod vip
pod "vip" deleted

vim vip.yaml
....
# nodeSelector:
#
status: vip

9. Create the pod again. Containers can now be spawning on either of the node. You may see pods for the daemonsets
as well.


kubectl get pods
<output_omitted>

kubectl create -f vip.yaml
pod/vip created

10. Determine where the new containers have been deployed. They should be more evenly spread this time. Again, the
numbers may be different, the change in numbers is what we are looking for. Due to lack of nodeSelector they could
go to either node.
sudo crictl ps |wc -l

sudo crictl ps |wc -l

11. Create another file for other users. Change the names from vip to others, and uncomment the nodeSelector lines.
cp vip.yaml other.yaml
sed -i s/vip/other/g other.yaml
vim other.yaml

other.yaml
2

....
nodeSelector:
status: other

12. Create the other containers. Determine where they deploy.
kubectl create -f other.yaml
pod/other created

sudo crictl ps |wc -l

sudo crictl ps |wc -l

13. Shut down both pods and verify they terminated. Only our previous pods should be found.


kubectl delete pods vip other
pod "vip" deleted
pod "other" deleted

kubectl get pods
<output_omitted>


---

## Exercise 12.2: Using Taints to Control Pod Deployment

```bash
cd ~/lfs458/ch12-scheduling/
```

Use taints to manage where Pods are deployed or allowed to run. In addition to assigning a Pod to a group of nodes,
you may also want to limit usage on a node or fully evacuate Pods. Using taints is one way to achieve this. You may
remember that the cp node begins with a NoSchedule taint. We will work with three taints to limit or remove running
pods.
1. Create a deployment which will deploy eight nginx containers. Begin by creating a YAML file.
cp /home/student/LFS458/SOLUTIONS/s_12/taint.yaml .
vim taint.yaml

taint.yaml
2
4
6
8
10
12
14
16
18

apiVersion: apps/v1
kind: Deployment
metadata:
name: taint-deployment
spec:
replicas: 8
selector:
matchLabels:
app: nginx
template:
metadata:
labels:
app: nginx
spec:
containers:
- name: nginx
image: nginx:1.20.1
ports:
- containerPort: 80

2. Apply the file to create the deployment.
kubectl apply -f taint.yaml
deployment.apps/taint-deployment created

3. Determine where the containers are running. In the following example three have been deployed on the cp node and
five on the secondary node. Remember there will be other housekeeping containers created as well. Your numbers may
be different, the actual number is not important, we are tracking the change in numbers.


sudo crictl ps |grep nginx
00c1be5df1e7
<output_omitted>

nginx@sha256:e3456c851a152494c3e.....

sudo crictl ps |wc -l

sudo crictl ps |wc -l

4. Delete the deployment. Verify the containers are gone.
kubectl delete deployment taint-deployment
deployment.apps "taint-deployment" deleted

sudo crictl ps |wc -l

5. Now we will use a taint to affect the deployment of new containers. There are three taints, NoSchedule,
PreferNoSchedule and NoExecute. The taints having to do with schedules will be used to determine newly deployed
containers, but will not affect running containers. The use of NoExecute will cause running containers to move.
Taint the secondary node, verify it has the taint then create the deployment again. We will use the key of bubba to
illustrate the key name is just some string an admin can use to track Pods.
kubectl taint nodes worker \
bubba=value:PreferNoSchedule
node/worker tainted

kubectl describe node |grep Taint
Taints:
Taints:

bubba=value:PreferNoSchedule
<none>

kubectl apply -f taint.yaml
deployment.apps/taint-deployment created

6. Locate where the containers are running. We can see that more containers are on the cp, but there still were some
created on the secondary. Delete the deployment when you have gathered the numbers.
sudo crictl ps |wc -l


sudo crictl

ps |wc -l

kubectl delete deployment taint-deployment
deployment.apps "taint-deployment" deleted

7. Remove the taint, verify it has been removed. Note that the key is used with a minus sign appended to the end.
kubectl taint nodes worker bubbanode/worker untainted

kubectl describe node |grep Taint
Taints:
Taints:

<none>
<none>

8. This time use the NoSchedule taint, then create the deployment again. The secondary node should not have any new
containers, with only daemonsets and other essential pods running.
kubectl taint nodes worker \
bubba=value:NoSchedule
node/worker tainted

kubectl apply -f taint.yaml
deployment.apps/taint-deployment created

sudo crictl ps |wc -l

sudo crictl ps |wc -l

9. Remove the taint and delete the deployment. When you have determined that all the containers are terminated create
the deployment again. Without any taint the containers should be spread across both nodes.
kubectl delete deployment taint-deployment
deployment.apps "taint-deployment" deleted

kubectl taint nodes worker bubbanode/worker untainted


kubectl apply -f taint.yaml
deployment.apps/taint-deployment created

sudo crictl ps |wc -l

sudo crictl ps |wc -l

10. Now use the NoExecute to taint the secondary (worker) node. Wait a minute then determine if the containers have
moved. The DNS containers can take a while to shutdown. Some containers will remain on the worker node to continue
communication from the cluster.
kubectl taint nodes worker \
bubba=value:NoExecute
node "worker" tainted

sudo crictl ps |wc -l

sudo crictl ps |wc -l

11. Remove the taint. Wait a minute. Note that all of the containers did not return to their previous placement.
kubectl taint nodes worker bubbanode/worker untainted

sudo crictl ps |wc -l

sudo crictl ps |wc -l

12. Remove the deployment a final time to free up resources.
kubectl delete deployment taint-deployment
deployment.apps "taint-deployment" deleted


13. CHALLENGE STEP Use your knowledge of deployments and scaling items to deploy multiple httpd pods across the
nodes and examine the typical spread and spread after using taints.


13.1

Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 312

13.2

Troubleshooting Flow . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 313

13.3

Basic Start Sequence . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 315

13.4

Monitoring . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 316

13.5

Plugins . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 317

13.6

Logging . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 321

13.7

Troubleshooting Resources . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 322

13.8

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 323


13.1


Overview

Overview

• Linux troubleshooting via shell
• Turn on basic monitoring
• Set up cluster-wide logging
• External products Fluentd, Prometheus helpful.
• Internal Metrics Server and API

Kubernetes relies on API calls and is sensitive to network issues. Standard Linux tools and processes are the best method for
troubleshooting your cluster. If a shell, such as bash is not available in an affected Pod, consider deploying another similar pod
with a shell, like busybox. DNS configuration files and tools like dig are a good place to start. For more difficult challenges
you may need to install other tools like tcpdump.
Large and diverse workloads can be difficult to track, so monitoring of usage is essential. Monitoring is about collecting key
metrics such as CPU, memory, and disk usage, and network bandwidth on your nodes, as well as monitoring key metrics in
your applications. These features are being been ingested into Kubernetes with the Metric Server, which a cut-down version
of the now deprecated Heapster. Once installed the Metrics Server exposes a standard API which can be consumed by
other agents such as autoscalers. Once installed this endpoint can be found here on the cp server: /apis/metrics/k8s.io/
Logging activity across all the nodes is another feature not part of Kubernetes. Using Fluentd can be a useful data collector
for a unified logging layer. Having aggregated logs can help visualize the issue and provides the ability to search all logs.
A good place to start when local network troubleshooting does not expose the root cause. It can be downloaded from
http://www.fluentd.org
Another project from cncf.io combines logging, monitoring, and alerting called Prometheus can be found here https:
//prometheus.io. It provides a time-series database as well as integration with Grafana for visualization and dashboards.
We are going to review some of the basic kubectl commands that you can use to debug what is happening, and we will
walk you through the basic steps to be able to debug your containers, your pending containers, and also the systems
in Kubernetes.


13.2. TROUBLESHOOTING FLOW

13.2

Troubleshooting Flow

Basic Steps
• Errors from command line
• Pod logs and state of the Pod
• Use shell to troubleshoot Pod DNS and network
• Check node logs for errors. Enough resources
• RBAC, SELinux or AppArmor security settings
• API calls to and from controllers to kube-apiserver
• Enable auditing
• Inter-node network issues, DNS and firewall
• Control Plane server controllers.
– Control Pods in pending or error state
– Errors in log files
– Enough resources

The flow of troubleshooting should start with the obvious. If there are errors from the command line investigate them first. The
symptoms of the issue will probably determine the next step to check. Working from the application running inside a container
to the cluster as a whole may be a good idea. The application may have a shell you can use, for example:
$ kubectl create deploy busybox --image=busybox --command sleep 3600
$ kubectl exec -ti <busybox_pod> -- /bin/sh

If the Pod is running use kubectl logs pod-name to view standard out of the container. Without logs you may consider
deploying a sidecar container in the Pod to generate and handle logging. The next place to check is networking including
DNS, firewalls and general connectivity using standard Linux commands and tools.
Security settings can also be a challenge. RBAC, covered in the security chapter, provides mandatory or discretionary access
control in a granular manner. SELinux and AppArmor are also common issues, especially with network-centric applications.
A newer feature of Kubernetes is the ability to enable auditing for the kube-apiserver, which can allow a view into actions
after the API call has been accepted.
The issues found with a decoupled system like Kubernetes are similar to those of a traditional datacenter, plus the added
layers of Kubernetes controllers.


Ephemeral Containers

• Add a tool-filled container to a running pod
• Alpha with v1.16 release
kubectl debug buggypod --image debian --attach

A feature new to the 1.16 version is the ability to add a container to a running pod. This would allow a feature-filled container
to be added to an existing pod without having to terminate and re-create. Intermittent and difficult to determine problems may
take a while to reproduce, or not exist with the addition of another container.
As an Alpha stability feature, it may change or be removed at any time. As well they will not be restarted automatically, and
several resources such as ports or resources are not allowed.
These containers are added via the ephemeralcontainers handler via an API call, not via the podSpec. As a result the use
of kubectl edit is not possible.
You may be able to use the kubectl attach command to join an existing process within the container. This can be helpful
instead of kubectl exec which executes a new process. The functionality of the attached process depends entirely on what
you are attaching to.


13.3. BASIC START SEQUENCE

13.3

Basic Start Sequence

Cluster Start Sequence
• systemd starts kubelet.service
– Uses /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
– Uses /var/lib/kubelet/config.yaml config file
– staticPodPath set to /etc/kubernetes/manifests/
• kubelet creates all pods from *.yaml in directory
– kube-apiserver
– etcd
– kube-controller-manager
– kube-scheduler
• kube-controller-manager control loops use etcd data to start rest

The cluster startup sequence begins with systemd if you built the cluster using kubeadm. Other tools may leverage a different
method. Use systemctl status kubelet.service to see the current state and configuration files used to run the kubelet binary.
Inside of the config.yaml file you will find several settings for the binary including the staticPodPath which indicates the
directory where kubelet will read every yaml file and start every pod. If you put a yaml file in this directory it is a way to
troubleshoot the scheduler, as the pod is created with any requests to the scheduler.
The four default yaml files will start the base pods necessary to run the cluster. Once the watch loops and controllers from
kube-controller-manager run using etcd data the rest of the configured objects will be created.


13.4


Monitoring

Monitoring

• Enable the add-ons
• Metrics Server and API
• Prometheus
• Jaeger
• OpenTelemetry
• API Server Tracing

Monitoring is about collecting metrics from the infrastructure, as well as applications.
The long used and now deprecated Heapster has been replaced with an integrated Metrics Server. Once installed and
configured the server exposes a standard API which other agents can use to determine usage. It can also be configured to
expose custom metrics, which then could also be used by autoscalers to determine if an action should take place.

Prometheus
Prometheus is part of the Cloud Native Computing Foundation (CNCF). As a Kubernetes plugin, it allows one to
scrape resource usage metrics from Kubernetes objects across the entire cluster. It also has several client libraries
which allow you to instrument your application code in order to collect application level metrics.
Other CNCF projects such as OpenTelemetry which allows for adding instrumentation to code, and Jaeger for consuming
the telemetry are popular to use when tracking distributed issues. An alpha feature is API Server Tracing, which leverages
OpenTelemetry. More can be found here: https://kubernetes.io/blog/2021/09/03/api-server-tracing/.


13.5. PLUGINS

13.5

Plugins

Using Krew
• Install the software using steps at https://krew.dev
$ kubectl krew help
krew is the kubectl plugin manager.
You can invoke krew through kubectl: "kubectl krew [command]..."
Usage:
krew [command]
Available Commands:
help
Help about any command
info
Show information about a kubectl plugin
install
Install kubectl plugins
list
List installed kubectl plugins
search
Discover kubectl plugins
uninstall
Uninstall plugins
update
Update the local copy of the plugin index
upgrade
Upgrade installed plugins to newer versions
version
Show krew version and diagnostics

We have been using the kubectl command throughout the course. The basic commands can be used together in a more complex manner extending what can be done. There are over seventy and growing plugins available to interact with Kubernetes
objects and components.
At the moment plugins cannot overwrite existing kubectl commands. Nor can it add sub-commands to existing commands.
Writing new plugins should take into account the command line runtime package and a Go library for plugin authors.
As a plugin the declaration of options such as namespace or container to use must come after the command.
$ kubectl sniff bigpod-abcd-123 -c mainapp -n accounting

Plugins can be distributed in many ways. The use of krew allows for cross-platform packaging and a helpful plugin index,
which makes finding new plugins easy.
More information can be found here: https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/


Managing Plugins

• Add paths to plugins to $PATH variable
• View current plugins with kubectl plugin list
• Find new plugins with kubectl krew search
• Installing using kubectl krew install new-plugin
• Once installed use as kubectl sub-command
• upgrade and uninstall also available


13.5. PLUGINS

The help option explains basic operation. After installation ensure the $PATH includes the plugins. krew should allow easy
installation and use after that.
$ export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
$ kubectl krew search
NAME
access-matrix
advise-psp
....

DESCRIPTION
Show an RBAC access matrix for server resources
Suggests PodSecurityPolicies for cluster.

INSTALLED
no
no

$ kubectl krew install tail
Updated the local copy of plugin index.
Installing plugin: tail
Installed plugin: tail
\
| Use this plugin:
....
| | Usage:
| |
| |
# match all pods
| |
$ kubectl tail
| |
| |
# match pods in the 'frontend' namespace
| |
$ kubectl tail --ns staging
....


Sniffing Traffic With Wireshark

• Read installation output for basic usage
• Note some require extra packages and configuration.
• sniff requires Wireshark and ability to export graphical display
• Pass the command the pod and container to use
$ kubectl krew install sniff nginx-123456-abcd -c webcont

Cluster network traffic is encrypted making troubleshooting of possible network issues more complex. Using the sniff plugin
you can view the traffic from within.
The sniff command will use the first found container unless you pass the -c option to declare which container in the pod to
use for traffic monitoring.


13.6. LOGGING

13.6

Logging

Logging Tools

• No cluster-wide logging in Kubernetes
• Often aggregated and digested by outside tools like ElasticSearch
• Fluentd
• Kibana

Logging, like monitoring, is a vast subject in IT. It has many tools that you can use as part of your arsenal.
Typically, logs are collected locally and aggregated before being ingested by a search engine and displayed via a dashboard
which can use the search syntax. While there are many software stacks that you can use for logging, the Elasticsearch,
Logstash, and Kibana Stack (ELK) has become quite common.
In Kubernetes, the kubelet writes container logs to local files (via the Docker logging driver). The command kubectl logs
allows you to retrieve these logs.
Cluster-wide, you can use Fluentd to aggregate logs: http://www.fluentd.org/. Check the cluster administration logging concepts for a detailed description: https://kubernetes.io/docs/concepts/cluster-administration/logging/.
Fluentd is part of the Cloud Native Computing Foundation (CNCF) and, together with Prometheus, makes a nice
combination for monitoring and logging. You can find a detailed walk-through of running Fluentd on Kubernetes here:
https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana.
Setting up Fluentd for Kubernetes logging can be a good exercise in understanding DaemonSets. Fluentd agents
run on each node via DaemonSet, they aggregate the logs, and feed them to an Elasticsearch instance prior to
visualization in a Kibana dashboard.


13.7

Troubleshooting Resources

More Resources

• Official documentation
• Major vendor pages
• Github pages
• Kubernetes Slack channel

Other URLs to view for more troubleshooting information:
• General guidelines and instructions:
https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/
• Troubleshooting applications:
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application
• Troubleshooting clusters:
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster
• Debugging pods:
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller
• Debugging services:
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/
• Github Site for issue and bug tracking
https://github.com/kubernetes/kubernetes/issues
• Kubernetes Slack channel
kubernetes.slack.com


13.8

Labs


---


---

# Chapter 13: Logging and Troubleshooting
**Working directory: `~/lfs458/ch13-troubleshoot/`**

## Exercise 13.1: Review Log File Locations

```bash
cd ~/lfs458/ch13-troubleshoot/
```

Overview
In addition to various logs files and command output, you can use journalctl to view logs from the node perspective.
We will view common locations of log files, then a command to view container logs. There are other logging options,
such as the use of a sidecar container dedicated to loading the logs of another container in a pod.
Whole cluster logging is not yet available with Kubernetes. Outside software is typically used, such as Fluentd, part of
http://fluentd.org/, which is another member project of CNCF.io, like Kubernetes.
Take a quick look at the following log files and web sites. As server processes move from node level to running in containers
the logging also moves.
1. If using a systemd.based Kubernetes cluster, view the node level logs for kubelet, the local Kubernetes agent. Each
node will have different contents as this is node specific.
journalctl -u kubelet |less
<output_omitted>

2. Major Kubernetes processes now run in containers. You can view them from the container or the pod perspective. Use
the find command to locate the kube-apiserver log. Your output will be different, but will be very long.
sudo find / -name "*apiserver*log"
/var/log/containers/kube-apiserver-cp_kube-system_kube-apiserver-423
d25701998f68b503e64d41dd786e657fc09504f13278044934d79a4019e3c.log

3. Take a look at the log file.
sudo less /var/log/containers/kube-apiserver-cp_kube-system_kubeapiserver-423d25701998f68b503e64d41dd786e657fc09504f13278044934d79a4019e3c.log
<output_omitted>

4. Search for and review other log files for coredns, kube-proxy, and other cluster agents.
5. If not on a Kubernetes cluster using systemd which collects logs via journalctl you can view the text files on the cp
node.
(a) /var/log/kube-apiserver.log
Responsible for serving the API
(b) /var/log/kube-scheduler.log
Responsible for making scheduling decisions
(c) /var/log/kube-controller-manager.log
Controller that manages replication controllers
6. /var/log/containers
Various container logs


7. /var/log/pods/
More log files for current Pods.
8. Worker Nodes Files (on non-systemd systems)
(a) /var/log/kubelet.log
Responsible for running containers on the node
(b) /var/log/kube-proxy.log
Responsible for service load balancing
9. More reading: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/ and https://kubernetes.io/
docs/tasks/debug-application-cluster/determine-reason-pod-failure/


---

## Exercise 13.2: Viewing Logs Output

```bash
cd ~/lfs458/ch13-troubleshoot/
```

Container standard out can be seen via the kubectl logs command. If there is no standard out, you would not see any
output. In addition, the logs would be destroyed if the container is destroyed.
1. View the current Pods in the cluster. Be sure to view Pods in all namespaces.
kubectl get po --all-namespaces
NAMESPACE
kube-system
kube-system

NAME
cilium-operator-788c7d7585-jgn6s
cilium-5tv9d

....
kube-system
kube-system
kube-system
kube-system
....

etcd-cp
kube-apiserver-cp
kube-controller-manager-cp
kube-scheduler-cp

1/1
1/1
1/1
1/1

READY
1/1
1/1
Running
Running
Running
Running

STATUS
Running
Running
2
2

0

RESTARTS

AGE
13m
6d1h

44h
44h
44h
44h

2. View the logs associated with various infrastructure pods. Using the Tab key you can get a list and choose a container.
Then you can start typing the name of a pod and use Tab to complete the name.
kubectl -n kube-system logs <Tab><Tab>
cilium-operator-788c7d7585-jgn6s
cilium-5tv9d
coredns-5644d7b6d9-k7kts
coredns-5644d7b6d9-rnr2v
etcd-cp
kube-apiserver-cp
kube-controller-manager-cp
kube-proxy-qhc4f
kube-proxy-s56hl
kube-scheduler-f-cp
traefik-ingress-controller-hw5tv
traefik-ingress-controller-mcn47

kubectl -n kube-system logs \
kube-apiserver-cp
Flag --insecure-port has been deprecated, This flag will be removed in a future version.
I1119 02:31:14.933023
1 server.go:623] external host was not specified, using 10.128.0.3


I1119 02:31:14.933356
1 server.go:149] Version: v1.29.1
I1119 02:31:15.595131
1 plugins.go:158] Loaded 11 mutating admission controller(s)
successfully in the following order: NamespaceLifecycle,LimitRanger,ServiceAccount,
NodeRestriction,TaintNodesByCondition,Priority,DefaultTolerationSeconds,DefaultStorageClass,
StorageObjectInUseProtection,MutatingAdmissionWebhook,RuntimeClass.
I1119 02:31:15.595357
1 plugins.go:161] Loaded 7 validating admission controller(s)
successfully in the following order: LimitRanger,ServiceAccount,Priority,
PersistentVolumeClaimResize,ValidatingAdmissionWebhook,RuntimeClass,
ResourceQuota.
<output_omitted>

3. View the logs of other Pods in your cluster.


---

## Exercise 13.3: Adding tools for monitoring and metrics

```bash
cd ~/lfs458/ch13-troubleshoot/
```

With the deprecation of Heapster the new, integrated Metrics Server has been further developed and deployed. The
Prometheus project of CNCF.io has matured from incubation to graduation, is commonly used for collecting metrics,
and should be considered as well.

Configure Metrics
1. Create the necessary objects. Be aware as new versions are released there may be some changes to the process and
the created objects. Use the components.yaml to create the objects. The backslash is not necessary if you type it all on
one line.
kubectl create -f \
https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created

2. View the current objects, which are created in the kube-system namespace. All should show a Running status. You
will notice the metrics server pod is in not ready state. Allow the deployment to run insecure TLS and pod will start
accepting the traffic.
kubectl -n kube-system get pods
<output_omitted>
kube-proxy-ld2hb
kube-scheduler-u16-1-13-1-2f8c
metrics-server-fc6d4999b-b9rjj

1/1
1/1
0/1

Running
Running
Running

0

2d21h
2d21h
42s

3. Edit the metrics-server deployment to allow insecure TLS. The default certificate is x509 self-signed and not trusted
by default. In production you may want to configure and replace the certificate. You may encounter other issues as
this software is fast-changing. The need for the kubelet-preferred-address-types line has been reported on some
platforms.


kubectl -n kube-system edit deployment metrics-server

....

3
5
7
9

....

spec:
containers:
- args:
- --cert-dir=/tmp
- --secure-port=4443
- --kubelet-insecure-tls
#<-- Add this line
- --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname #<--May be needed
image: k8s.gcr.io/metrics-server/metrics-server:v0.3.7

4. Test that the metrics server pod is running and does not show errors. At first you should see a few lines showing the
container is listening. As the software changes these messages may be slightly different.
kubectl -n kube-system logs metrics-server<TAB>
I0207 14:08:13.383209
1 serving.go:312] Generated self-signed cert
(/tmp/apiserver.crt, /tmp/apiserver.key)
I0207 14:08:14.078360
1 secure_serving.go:116] Serving securely on
[::]:4443

5. Test that the metrics working by viewing pod and node metrics. Your output may have different pods. It can take an
minute or so for the metrics to populate and not return an error.
sleep 120 ; kubectl top pod --all-namespaces
NAMESPACE
NAME
kube-system
cilium-kube-controllers-7b9dcdcc5-qg6zd
kube-system
cilium-node-dr279
kube-system
cilium-node-xtvfd
kube-system
coredns-5644d7b6d9-k7kts
kube-system
coredns-5644d7b6d9-rnr2v
<output_omitted>

CPU(cores)
2m
23m
21m
2m
3m

MEMORY(bytes)
6Mi
22Mi
22Mi
6Mi
6Mi

kubectl top nodes
NAME
cp
228m
worker

76m

CPU(cores)
CPU%
MEMORY(bytes)
11%
2357Mi
31%
3%
1385Mi
18%

MEMORY%

6. Using keys we generated in an earlier lab we can also interrogate the API server. Your server IP address will be different.
curl --cert ./client.pem \
--key ./client-key.pem --cacert ./ca.pem \
https://k8scp:6443/apis/metrics.k8s.io/v1beta1/nodes
{

"kind": "NodeMetricsList",
"apiVersion": "metrics.k8s.io/v1beta1",
"metadata": {
"selfLink": "/apis/metrics.k8s.io/v1beta1/nodes"
},
"items": [


{

"metadata": {
"name": "u16-1-13-1-2f8c",
"selfLink": "/apis/metrics.k8s.io/v1beta1/nodes/u16-1-13-1-2f8c",
"creationTimestamp": "2024-08-10T20:27:00Z"
},
"timestamp": "2024-08-10T20:26:18Z",
"window": "30s",
"usage": {
"cpu": "215675721n",
"memory": "2414744Ki"
}

},
<output_omitted>

Configure the Dashboard
While the dashboard looks nice it has not been a common tool in use. Those that could best develop the tool tend to only use
the CLI, so it may lack full wanted functionality.
The first commands do not have the details. Refer to earlier content as necessary.
1. Copy the dashboard yaml from the tarball and deploy the dashboard.
cp /home/student/LFS458/SOLUTIONS/s_13/dashboard.yaml .
kubectl create -f dashboard.yaml

2. We will give the dashboard full admin rights, which may be more than one would in production. The dashboard is running
in the kubernetes-dashboard namespace. kubernetes-dashboard is the name of the service account.
There is more on service account in the Security chapter.
kubectl get sa -n kubernetes-dashboard
NAME
default
kubernetes-dashboard

SECRETS
0

AGE
4m25s
4m25s

kubectl create clusterrolebinding dashaccess \
--clusterrole=cluster-admin \
--serviceaccount=kubernetes-dashboard:kubernetes-dashboard
clusterrolebinding.rbac.authorization.k8s.io/dashaccess created

3. On your local system open a browser and navigate to an HTTPS URL made of the Public IP and the high-numbered
port. You will get a message about an insecure connection. Select the Advanced button, then Add Exception..., then
Confirm Security Exception. Some browsers won’t even give you to option. If nothing shows up try a different browser.
The page should then show the Kubernetes Dashboard. You may be able to find the public IP address using curl.
curl ifconfig.io
35.231.8.178


Figure 13.1: External Access via Browser

4. We will use the Token method to access the dashboard. With RBAC we need to use the proper token, the
kubernetes-dashboard-token in this case. Find the token, copy it then paste into the login page. The Tab key
can be helpful to complete the secret name instead of finding the hash.
kubectl create token kubernetes-dashboard -n kubernetes-dashboard
eyJlxvezoLAilithbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZX
JuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY
291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi1wbW04NCIsImt1YmVybmV0ZXMuaW8vc2Vydmlj
ZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2Vydml
jZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjE5MDY4ZDIzLTE1MTctMTFlOS1hZmMyLTQyMDEwYThlMDAwMyIsInN1Yi
I6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.aYTUMWr290pjt5i32rb8
qXpq4onn3hLhvz6yLSYexgRd6NYsygVUyqnkRsFE1trg9i1ftNXKJdzkY5kQzN3AcpUTvyj_BvJgzNh3JM9p7QMjI8LHTz4TrRZ
rvwJVWitrEn4VnTQuFVcADFD_rKB9FyI_gvT_QiW5fQm24ygTIgfOYd44263oakG8sL64q7UfQNW2wt5SOorMUtybOmX4CXNUYM8
G44ejEtv9GW5OsVjEmLIGaoEMX7fctwUN_XCyPdzcCg2WOxRHahBJmbCuLz2SSWL52q4nXQmhTq_L8VDDpt6LjEqXW6LtDJZGjVC
s2MnBLerQz-ZAgsVaubbQ


Figure 13.2: External Access via Browser

5. Navigate around the various sections and use the menu to the left as time allows. As the pod view is of the default
namespace, you may want to switch over to the kube-system namespace or create a new deployment to view the
resources via the GUI. Scale the deployment up and down and watch the responsiveness of the GUI.

Figure 13.3: External Access via Browser


14.1

Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 332

14.2

Custom Resource Definitions . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 333

14.3

Aggregated APIs . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 337

14.4

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 338


14.1


Overview

Custom Resources

• Flexible to meet changing needs
• Create your own API objects
• Manage your API objects via kubectl
• Custom Resource Definitions (CRD)
• Aggregated APIs (AA)

We have been working with built-in resources, or API endpoints. The flexibility of Kubernetes allows for dynamic addition of
new resources as well. Once these Custom Resources have been added the objects can be created and accessed using
standard calls and commands like kubectl. The creation of a new object stores new structured data in the etcd database and
allows access via kube-apiserver.
To make a new, custom resource part of a declarative API there needs to be a controller to retrieve the structured data
continually and act to meet and maintain the declared state. This controller, or operator, is an agent to create and mange
one or more instances of a specific stateful application. We have worked with built-in controllers such for Deployments,
DaemonSets and other resources.
The functions encoded into a custom operator should be all the tasks a human would need to perform if deploying the
application outside of Kubernetes. The details of building a custom controller are outside the scope of this course and not
included.
There are two ways to add custom resources to your Kubernetes cluster. The easiest, but less flexible, way is by adding
a Custom Resource Definition to the cluster. The second which is more flexible is the use of Aggregated APIs which
requires a new API server to be written and added to the cluster.
Either way of a new object to the cluster, as distinct from a built-in resource, is called a Custom Resource.
If you are using RBAC for authorization you probably will need to grant access to the new CRD resource and controller. If using
an Aggregated API, you can use the same or different authentication process.


14.2. CUSTOM RESOURCE DEFINITIONS

14.2

Custom Resource Definitions

Custom Resource Definitions

• Easy to deploy
• Typically does not require programming
• Does not require another API server
• Namespaced or cluster-scoped

As we have learned decoupled nature of Kubernetes depends on a collection of watcher loops, or controllers, interrogating
the kube-apiserver to determine if a particular configuration is true. If the current state does not match the declared state the
controller makes API calls to modify the state until they do match. If you add a new API object and controller you can use the
existing kube-apiserver to monitor and control the object. The addition of a Custom Resource Definition will be added to
the cluster API path, currently under apiextensions.k8s.io/v1.
While this is the easiest way to add a new object to the cluster it may not be flexible enough for your needs. Only the existing
API functionality can be used. Objects must respond to REST requests and have their configuration state validated and stored
in the same manner as built-in objects. They would also need to exist with the protection rules of built-in objects.
A CRD allows the resource to be deployed in a namespace or available in the entire cluster. The YAML file sets this with the
scope: parameter which can be set to Namespaced or Cluster.
Prior to v1.8 there was a resource type called ThirdPartyResource (TPR). This has been deprecated and is no longer
available. All resources will need to be rebuilt as CRD. After upgrade existing TPRs will need to be removed and replaced by
CRDs such that the API URL points to functional object.


Configuration Example
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
name: backups.stable.linux.com
spec:
group: stable.linux.com
versions: v1
scope: Namespaced
names:
plural: backups
singular: backup
shortNames:
- bks
kind: BackUp

apiVersion:
Should match the current level of stability, currently apiextensions.k8s.io/v1
kind: CustomResourceDefinition
The object type being inserted by the kube-apiserver.
name: backups.stable.linux.com
The name must match the spec field declared later. The syntax must be <plural name>.<group>.
group: stable.linux.com
The group name will become part of the REST API under /apis/<group>/<version> or /apis/stable/v1 in this case with
the versions set to v1.
scope
Determines if the object exists in a single namespace or is cluster-wide.
plural
Defines the last part of the API URL such as apis/stable/v1/backups.
singular and shortNames
represent the name with displayed and make CLI usage easier.
kind
A CamelCased singular type used in resource manifests.


14.2. CUSTOM RESOURCE DEFINITIONS

New Object Configuration

apiVersion: "stable.linux.com/v1"
kind: BackUp
metadata:
name: a-backup-object
spec:
timeSpec: "* * * * */5"
image: linux-backup-image
replicas: 5

Note that the apiVersion and kind match the CRD we created in a previous step. The spec parameters depend on the
controller.
The object will be evaluated by the controller. If the syntax, such as timeSpec does not match the expected value you
will receive and error, should validation be configured. Without validation only the existence of the variable is checked, not its
details.


Optional Hooks
• Finalizer
metadata:
finalizers:
- finalizer.stable.linux.com

• Validation
validation:
openAPIV3Schema:
properties:
spec:
properties:
timeSpec:
type: string
pattern: 'ˆ(\d+|\*)(/\d+)?(\s+(\d+|\*)(/\d+)?){4}$'
replicas:
type: integer
minimum: 1
maximum: 10

Just as with built-in objects you can use an asynchronous pre-delete hook known as a Finalizer. If an API delete request is
received the object metadata field metadata.deletionTimestamp is updated. The controller then triggers whichever finalizer
has been configured. When the finalizer completes it is removed from the list. The controller continues to compete and remove
finalizers until the string is empty. Then the object itself is deleted.
A feature in beta starting with v1.9 allows for validation of custom objects via the OpenAPI v3 schema. This will check various
properties of the object configuration being passed the API server. In the example above the timeSpec must be a string
matching a particular pattern and the number of allowed replicas is between one and 10. If the validation does not match the
error returned is the failed line of validation.


14.3. AGGREGATED APIS

14.3

Aggregated APIs

Understanding Aggregated APIs (AA)

• Usually requires non-trivial programming
• More control over API behavior
• Subordinate API server behind primary
• Primary acts as proxy
• Leverages extension resource
• Mutual TLS auth between API servers
• RBAC rule to allow addition of API service objects

The use of Aggregated APIs allows adding additional Kubernetes-stype API servers to the cluster. The added server acts
as a subordinate to kube-apiserver which as of v1.7 runs the aggregation layer in-process. When an extension resource is
registered the aggregation layer watches a passed URL path and proxy any requests to the newly registered API service.
The aggregation layer is easy to enable. Edit the flags passed during startup of the kube-apiserver to include
--enable-aggregator-routing=true. Some vendors enable this feature by default.
The creation of the exterior can be done via YAML configuration files or APIs. Configuring TLS auth between components
and RBAC rules for various new objects is also required. A sample API server is available on github here: https://github.com/
kubernetes/sample-apiserver. A project currently in incubation stage is an API server builder which should handle much of
the security and connection configuration. It can be found here: https://github.com/kubernetes-incubator/apiserver-builder


14.4

Labs


---


---

# Chapter 14: Custom Resource Definitions
**Working directory: `~/lfs458/ch14-crd/`**

## Exercise 14.1: Create a Custom Resource Definition

```bash
cd ~/lfs458/ch14-crd/
```

Overview
The use of CustomResourceDefinitions (CRD), has become a common manner to deploy new objects and operators. Creation of a new operator is beyond the scope of this course, basically it is a watch-loop comparing a spec to
the current status, and making changes until the states match. A good discussion of creating a operators can be found
here: https://operatorframework.io/.
First we will examine an existing CRD, then make a simple CRD, but without any particular action. It will be enough to find the
object ingested into the API and responding to commands.
1. View the existing CRDs.
kubectl get crd --all-namespaces
NAME
NAME
authorizationpolicies.policy.linkerd.io
ciliumcidrgroups.cilium.io
ciliumclusterwidenetworkpolicies.cilium.io
<output_omitted>

CREATED AT
CREATED AT
2024-08-28T11:30:34Z
2024-08-28T08:58:54Z
2024-08-28T08:58:57Z

2. We can see from the names that these CRDs are all working on Cilium, our network plugin. View the cilium-cni.yaml
file we used when we initialized the cluster to see how these objects were created, and some CRD templates to review.
cp /home/student/LFS458/SOLUTIONS/s_03/cilium-cni.yaml .
less cilium-cni.yaml
kubectl describe crd ciliumcidrgroups.cilium.io
<output_omitted>
--Name:
ciliumcidrgroups.cilium.io
Namespace:
Labels:
io.cilium.k8s.crd.schema.version=1.26.10
Annotations: <none>
API Version: apiextensions.k8s.io/v1
Kind:
CustomResourceDefinition
Metadata:
<output_omitted>

3. Now that we have seen some examples, we will create a new YAML file.
cp /home/student/LFS458/SOLUTIONS/s_14/crd.yaml .
vim crd.yaml


crd.yaml
2
4
6
8
10
12
14
16
18
20
22
24
26
28
30
32
34
36
38
40

apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
# name must match the spec fields below, and be in the form: <plural>.<group>
name: crontabs.stable.example.com
spec:
# group name to use for REST API: /apis/<group>/<version>
group: stable.example.com
# list of versions supported by this CustomResourceDefinition
versions:
- name: v1
# Each version can be enabled/disabled by Served flag.
served: true
# One and only one version must be marked as the storage version.
storage: true
schema:
openAPIV3Schema:
type: object
properties:
spec:
type: object
properties:
cronSpec:
type: string
image:
type: string
replicas:
type: integer
# either Namespaced or Cluster
scope: Namespaced
names:
# plural name to be used in the URL: /apis/<group>/<version>/<plural>
plural: crontabs
# singular name to be used as an alias on the CLI and for display
singular: crontab
# kind is normally the CamelCased singular type. Your resource manifests use this.
kind: CronTab
# shortNames allow shorter string to match your resource on the CLI
shortNames:
- ct

4. Add the new resource to the cluster.
kubectl create -f crd.yaml
customresourcedefinition.apiextensions.k8s.io/crontabs.stable.example.com created

5. View and describe the resource. The new line may be in the middle of the output. You’ll note the describe output is
unlike other objects we have seen so far.
kubectl get crd
NAME
<output_omitted>
crontabs.stable.example.com
<output_omitted>


CREATED AT
2024-08-13T03:18:07Z


kubectl describe crd crontab<Tab>
Name:
crontabs.stable.example.com
Namespace:
Labels:
<none>
Annotations: <none>
API Version: apiextensions.k8s.io/v1
Kind:
CustomResourceDefinition
<output_omitted>

6. Now that we have a new API resource we can create a new object of that type. In this case it will be a crontab-like
image, which does not actually exist, but is being used for demonstration.
cp /home/student/LFS458/SOLUTIONS/s_14/new-crontab.yaml .
vim new-crontab.yaml

new-crontab.yaml
2
4
6
8
10

apiVersion: "stable.example.com/v1"
# This is from the group and version of new CRD
kind: CronTab
# The kind from the new CRD
metadata:
name: new-cron-object
spec:
cronSpec: "*/5 * * * *"
image: some-cron-image
#Does not exist

7. Create the new object and view the resource using short and long name.
kubectl create -f new-crontab.yaml
crontab.example.com/new-cron-object created

kubectl get CronTab
NAME
new-cron-object

AGE
22s

kubectl get ct
NAME
new-cron-object

AGE
29s

kubectl describe ct
Name:
Namespace:
Labels:
Annotations:
API Version:
Kind:


new-cron-object
default
<none>
<none>
stable.example.com/v1
CronTab


<output_omitted>
Spec:
Cron Spec:
Image:
Events:

*/5 * * * *
some-cron-image
<none>

8. To clean up the resources we will delete the CRD. This should delete all of the endpoints and objects using it as well.
kubectl delete -f crd.yaml
customresourcedefinition.apiextensions.k8s.io "crontabs.stable.example.com" deleted


15.1

Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 344

15.2

Accessing the API . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 346

15.3

Authentication and Authorization . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 347

15.4

Admission Controller . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 351

15.5

Network Policies . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 353

15.6

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 357


15.1


Overview

Overview

• Explain how API requests go through the system.
• Compare different types of authentication.
• Configure authorization rules.
• Configure pod policies to control what containers are allowed to do.
• Restrict network traffic using network policies.

Security is a big and complex topic, especially in a distributed system like Kubernetes. Thus, we are just going to cover
some of the concepts that deal with security in the context of Kubernetes. In-depth Cloud and Kubernetes security
is covered in detail in the Kubernetes Security Fundamentals (LFS460) course https://training.linuxfoundation.org/training/
kubernetes-security-fundamentals-lfs460/
Then we are going to focus on the authentication aspect of the API server and will dive into authorization, looking at things
like RBAC, which is now the default configuration when you bootstrap a Kubernetes cluster with kubeadm.
We are going to look at the admission control system, which lets you look at and possibly modify the requests that are
coming in, and do a final deny or accept those requests.
Following that we’re going to look at a few other concepts, including how you can secure your Pods more tightly using security
contexts and pod security policies, which are full-fledged API objects in Kubernetes.
Finally, we will look at network policies. By default, we tend to not turn on network policies, which lets any traffic flow through
all our pods, in all the different namespaces. Using network policies, we can actually define Ingress rules so that we can
restrict the Ingress traffic between the different namespaces. The network tool in use, such as flannel or Calico will determine if a network policy can be implemented. As Kubernetes becomes more mature, this will become a strongly suggested
configuration.


15.1. OVERVIEW

Cloud Security Considerations

• Ongoing cycle of security
• Image and binary supply chain
• Hardening the operating system
• Securing the kube-apiserver
• Network considerations
• Static and runtime workload analysis
• Issue detection

Keeping a cloud environment secure is an ongoing, and wide ranging task. As more moves to the cloud we must look at more
than just Kubernetes towards the hardware, software, and configuration options for the entire environment. Starting in the
design phase care must be taken to secure safe hardware, firmware and operating system binaries.
Once the platform is hardened the kube-apiserver has a list of considerations, tools, and settings to limit access and formalized
access in an easy to understand manner.
As a network intensive environment it becomes important to secure the network both inside Kubernetes as done with a
NetworkPolicy as well as traditional firewall tools and pod to pod encryption.
Minimizing base images, insisting on container immutability, and static and runtime analysis of tools is also an important part
of security which often begins with developers and is implemented in the CI/CD pipeline prior to an image being used in a
production cluster. Tools like AppArmor and SELinux should also be used to further protect the environment from malicious
containers.
Security is more than just a settings and configuration. It is an ongoing process of issue detection using intrusion detection
tools and behavioral analytics. There needs to be an ongoing process of assessment, prevention, detection, and reaction
following written and often updated policies.


15.2

Accessing the API

Accessing the API

Figure 15.1: Accessing the API from kubernetes.io

To perform any action in a Kubernetes cluster, you need to access the API and go through three main steps:
• Authentication (token)
• Authorization (RBAC)
• Admission Controllers
These steps are described in more detail in the official documentation about controlling access to the API at https://kubernetes.
io/docs/admin/accessing-the-api/, and illustrated by the picture.
Once a request reaches the API server securely, it will first go through any authentication module that has been configured.
The request can be rejected if authentication fails or it gets authenticated and passed to the authorization step.
At the authorization step, the request will be checked against existing policies. It will be authorized if the user has the
permissions to perform the requested actions. Then, the requests will go through the last step of admission. In general,
admission controllers will check the actual content of the objects being created and validate them before admitting the request.
In addition to these steps, the requests reaching the API server over the network are encrypted using TLS. This needs to be
properly configured using SSL certificates. If you use kubeadm this configuration is done for you; otherwise, follow this guide:
https://github.com/kelseyhightower/kubernetes-the-hard-way or the API server configuration options: https://kubernetes.io/
docs/admin/kube-apiserver/.


15.3. AUTHENTICATION AND AUTHORIZATION

15.3

Authentication and Authorization

Authentication
• One or more Authenticator Modules used
– x509 Client Certs
– Static Token, Bearer or Bootstrap Token
– Static Password File
– Service Account and OpenID Connect Tokens
• Each tried until success, order not guaranteed
• Anonymous access can be enabled. Otherwise 401 response.
• Users are not created by the API, should be managed by an external
system.
• Use of proxy or webhook to LDAP, SAML, Kerberos, alternate x509 etc.
instead.
• System accounts are used by processes to access the API

There are three main points to remember with authentication in Kubernetes. In its straightforward form, authentication is done
with certificates, tokens or basic authentication (i.e. username and password). Users are not created by the API, but should
be managed by an external system. System accounts are used by processes to access the API: https://kubernetes.io/docs/
tasks/configure-pod-container/configure-service-account/.
There are two more advanced authentication mechanisms: Webhooks which can be used to verify bearer tokens; and
connection with an external OpenID provider.
The type of authentication used is defined in the kube-apiserver startup options. Below are four examples of a subset of
configuration options that would need to be set depending on what choice of authentication mechanism you choose. For
example:
• --basic-auth-file
• --oidc-issuer-url
• --token-auth-file
• --authorization-webhook-config-file
To learn more about authentication,
access-authn-authz/authentication/.


see the official documentation at:

https://kubernetes.io/docs/reference/


Authorization

• RBAC
• WebHook

Once a request is authenticated, it needs to be authorized to be able to proceed through the Kubernetes system and perform
its intended action.
There are three main authorization modes and two global Deny/Allow settings.
They can be configured as kube-apiserver startup options:
• --authorization-mode=RBAC
• --authorization-mode=Webhook
• --authorization-mode=AlwaysDeny
• --authorization-mode=AlwaysAllow
The authorization modes implement policies to allow requests. Attributes of the requests are checked against the policies (e.g.
user, group, namespace, verb).


15.3. AUTHENTICATION AND AUTHORIZATION

RBAC

• Resources and Operations (verbs)
• Rules
• Roles and ClusterRoles
• Subjects
• RoleBindings and ClusterRoleBindings

RBAC stands for Role Based Access Control: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
All resources are modeled API objects in Kubernetes from Pods to Namespaces. They also belong to API Groups such
as core and apps. These resources allow operations such as Create, Read, Update, and Delete (CRUD) which are called
operations, which we have been working with so far. Operations are called verbs inside YAML files. Adding to these basic
components we will add more elements of the API, which can then be managed via RBAC.
Rules are operations which can act upon an API group. Roles are a group of rules which affect, or scope, a single namespace,
whereas ClusterRoles have a scope of the entire cluster.
Each operation can act upon one of three subjects which are User Accounts which don’t exist as API objects,
Service Accounts, and Groups which are known as clusterrolebinding when using kubectl.
RBAC is then writing rules to allow or deny operations by users, roles or groups upon resources.


RBAC Process Overview

• Determine or create namespace
• Create certificate credentials for user
• Set the credentials for the user to the namespace using a context
• Create a role for the expected task set
• Bind the user to the role
• Verify the user has limited access

While RBAC can be complex the basic flow is to create a certificate for a user. As a user is not a API object of Kubernetes,
requiring outside authentication such as OpenSSL certificates. After generation of the certificate against the cluster certificate
authority we can set that credential for the user using a context.
Roles can then be used to configure an association of apiGroups, resources, and the verbs allowed to them. The user can
then be bound to a role limiting what and where they can work the the cluster.


15.4. ADMISSION CONTROLLER

15.4

Admission Controller

Admission Controller

• Access content of objects created
• Modify or validate content

The last step in letting an API request into Kubernetes is Admission Control.
Admission controllers are pieces of software that can access the content of the objects being created by the requests. They
can modify the content or validate it, and potentially deny the request.
Admission controllers are needed for certain features to work properly. Controllers have been added as Kubernetes has
matured. As of the v1.12 release the kube-apiserver uses a compiled in set of controllers. Instead of passing a list we can
enable or disable particular controllers. If you want to use a controller not available by default you would need to download
source and compile.
The first controller is Initializers which will allow dynamic modification of the API request, providing great flexibility. Each
admission controller functionality is explained in the documentation. For example, the ResourceQuota controller will ensure
that the object created does not violate any of the existing quotas.


Security Contexts

apiVersion: v1
kind: Pod
metadata:
name: nginx
spec:
securityContext:
runAsNonRoot: true
containers:
- image: nginx
name: nginx

Pods and containers within Pods can be given specific security constraints to limit what processes running in containers can
do. For example, the UID of the process, the Linux capabilities, and the file system group can be limited.
This security limitation is called a security context. It can be defined for the entire Pod or per container and is represented
as additional sections in the resources manifests. The notable difference is that Linux capabilities are set at the container
level.
For example, if you want to enforce a policy that containers cannot run their process as the root user, you can add a Pod
security context like the one above.
Then, when you create this pod, you will see a warning that the container is trying to run as root and that it is not allowed.
Hence, the Pod will never run:
$ kubectl get pods
NAME
nginx

READY
0/1

STATUS
container has runAsNonRoot and image will run as root

RESTARTS

Read more about security contexts to give proper constraints to your containers at:
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/.


AGE
10s

15.5. NETWORK POLICIES

15.5

Network Policies

Network Security Policies

• All traffic allowed by default
• Networking add-ons must support network policies
• Can be limited to a namespace
• Egress policy type now available
• Can match PodSelector, ingress labels and egress labels.

By default all Pods can reach each other; all ingress and egress traffic is allowed. This has been a high-level networking
requirement in Kubernetes. However, network isolation can be configured and traffic to pods can be blocked. In newer
versions of Kubernetes egress traffic can also be blocked. This is done by configuration of a NetworkPolicy. As all traffic is
allowed you may want to implement a policy that drops all traffic, then other policies which allow desired ingress and egress
traffic.
The spec of the policy can narrow down effect to a particular namespace, which can be handy. Further settings include a
podSelector, or label, to narrow down which Pods are affected. Further ingress and egress settings declare to and from
IP addresses and ports.
Not all network providers support the NetworkPolicies kind. A non-exhaustive list of providers with support includes Calico,
Romana, Cilium, Kube-router, and WeaveNet.
In previous versions of Kubernetes there was a requirement to annotate a namespace as part of network isolation, specifically
the net.beta.kubernetes.io/network-policy= value. Some network plugins may still require this setting.
Following is an example of a NetworkPolicy recipe.
kubernetes-network-policy-recipes


More recipes can be found here: https://github.com/ahmetb/


Network Security Policy Example
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
name: ingress-egress-policy
namespace: default
spec:
podSelector:
matchLabels:
role: db
policyTypes:
- Ingress
- Egress
ingress:
- from:
- ipBlock:
cidr: 172.17.0.0/16
except:
- 172.17.1.0/24
<continued_on_next_slide>
The use of policies has become stable, noted with the v1 apiVersion. The example above narrows down the policy to affect
the default namespace.
Only Pods with the label of role db: will be affected by this policy, and has both Ingress and Egress settings.
The ingress setting includes a 172.17 network, with a smaller range of 172.17.1.0 IPs being excluded from this traffic.


15.5. NETWORK POLICIES

Network Security Policy Example Cont.
- namespaceSelector:
matchLabels:
project: myproject
- podSelector:
matchLabels:
role: frontend
ports:
- protocol: TCP
port: 6379
egress:
- to:
- ipBlock:
cidr: 10.0.0.0/24
ports:
- protocol: TCP
port: 5978

These rules change the namespace for the following settings to be labeled project myproject:. The affected Pods also would
need to match the label role frontend:. Finally TCP traffic on port 6379 would be allowed from these Pods.
The egress rules have to settings, in this case the 10.0.0.0/24 range TCP traffic to port 5978.
The use of empty ingress or egress rules denies all of type of traffic for the included Pods, though not suggested. Use another
dedicated NetworkPolicy instead.
There can also be complex matchExpressions statements in the spec, but this may change as NetworkPolicy
matures.
podSelector:
matchExpressions:
- {key: inns, operator: In, values: ["yes"]}


Default Policy Example

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
name: default-deny
spec:
podSelector: {}
policyTypes:
- Ingress

The empty braces will match all Pods not selected by other NetworkPolicy will not allow ingress traffic. Egress traffic would
be unaffected by this policy.
With the potential for complex ingress and egress rules it may be helpful to create multiple objects which include simple
isolation rules and use easy to understand names and labels.
Some network plugins, such as WeaveNet may require annotation of the Namespace. The following shows the setting of a
DefaultDeny for the myns namespace:
kind: Namespace
apiVersion: v1
metadata:
name: myns
annotations:
net.beta.kubernetes.io/network-policy: |
{
"ingress": {
"isolation": "DefaultDeny"
}
}


15.6

Labs


---


---

# Chapter 15: Security
**Working directory: `~/lfs458/ch15-security/`**

## Exercise 15.1: Working with TLS

```bash
cd ~/lfs458/ch15-security/
```

Overview
We have learned that the flow of access to a cluster begins with TLS connectivity, then authentication followed by
authorization, finally an admission control plug-in allows advanced features prior to the request being fulfilled. The use
of Initializers allows the flexibility of a shell-script to dynamically modify the request. As security is an important,
ongoing concern, there may be multiple configurations used depending on the needs of the cluster.
Every process making API requests to the cluster must authenticate or be treated as an anonymous user.
While one can have multiple cluster root Certificate Authorities (CA) by default each cluster uses their own, intended for intracluster communication. The CA certificate bundle is distributed to each node and as a secret to default service accounts. The
kubelet is a local agent which ensures local containers are running and healthy.
1. View the kubelet on both the cp and secondary nodes. The kube-apiserver also shows security information such as
certificates and authorization mode. As kubelet is a systemd service we will start looking at that output.
systemctl status kubelet.service
kubelet.service - kubelet: The Kubernetes Node Agent
Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: en
Drop-In: /etc/systemd/system/kubelet.service.d
|__10-kubeadm.conf
<output_omitted>

2. Look at the status output. Follow the CGroup and kubelet information, which is a long line where configuration settings
are drawn from, to find where the configuration file can be found.
CGroup: /system.slice/kubelet.service
|--19523 /usr/bin/kubelet .... --config=/var/lib/kubelet/config.yaml ..

3. Take a look at the settings in the /var/lib/kubelet/config.yaml file. Among other information we can see the
/etc/kubernetes/pki/ directory is used for accessing the kube-apiserver. Near the end of the output it also sets the
directory to find other pod spec files.
sudo less /var/lib/kubelet/config.yaml

config.yaml
2
4
6
8

<output_omitted>
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s

4. Other agents on the cp node interact with the kube-apiserver. View the configuration files where these settings are
made. This was set in the previous YAML file. Look at one of the files for cert information.


sudo ls /etc/kubernetes/manifests/
etcd.yaml
kube-apiserver.yaml

kube-controller-manager.yaml
kube-scheduler.yaml

sudo less /etc/kubernetes/manifests/kube-controller-manager.yaml
<output_omitted>

5. The use of tokens has become central to authorizing component communication. The tokens are kept as secrets. Take
a look at the current secrets in the kube-system namespace. Note: Token is valid only for 24 hours and then the secret
is removed, If output does not show ’bootstrap-token-xxxxxx’ secret, create a new token
sudo kubeadm token create


kubectl -n kube-system get secrets

NAME
DATA
AGE
bootstrap-token-i3r13t
5d
<output_omitted>

TYPE
bootstrap.kubernetes.io/token

6. Take a closer look at one of the secrets and the token within. The bootstrap-token could be one to look at. The use
of the Tab key can help with long names. Long lines have been truncated in the output below.
kubectl -n kube-system get secrets bootstrap-token<Tab> -o yaml

2
4
6
8
10
12
14
16

apiVersion: v1
data:
auth-extra-groups: c3lzdGVtOmJvb3RzdHJhcHBlcnM6a3ViZWFkbTpkZWZhdWx0LW5vZGUtdG9rZW4=
expiration: MjAyMy0wNi0wMlQxMjowOTowMlo=
token-id: NXBvMGo3
token-secret: MXhzMDgxOG1rcTFyeDQxbg==
usage-bootstrap-authentication: dHJ1ZQ==
usage-bootstrap-signing: dHJ1ZQ==
kind: Secret
metadata:
creationTimestamp: "2024-08-01T12:09:02Z"
name: bootstrap-token-5po0j7
namespace: kube-system
resourceVersion: "209"
uid: 98219199-4876-4cfa-a4be-586cda27cc6b
type: bootstrap.kubernetes.io/token

7. The kubectl config command can also be used to view and update parameters. When making updates this could avoid
a typo removing access to the cluster. View the current configuration settings. The keys and certs are redacted from the
output automatically.
kubectl config view
apiVersion: v1
clusters:
- cluster:
certificate-authority-data: REDACTED


<output_omitted>

8. View the options, such as setting a password for the admin instead of a key. Read through the examples and options.
kubectl config set-credentials -h
Sets a user entry in kubeconfig
<output_omitted>

9. Make a copy of your access configuration file. Later steps will update this file and we can view the differences.
cp $HOME/.kube/config $HOME/cluster-api-config

10. Explore working with cluster and security configurations both using kubectl and kubeadm. Among other values, find
the name of your cluster. You will need to become root to work with kubeadm.
kubectl config <Tab><Tab>
current-context
delete-cluster
delete-context
get-clusters

get-contexts
rename-context
set
set-cluster

set-context
set-credentials
unset
use-context

view

sudo kubeadm token -h
<output_omitted>

sudo kubeadm config -h
<output_omitted>

11. Review the cluster default configuration settings. There may be some interesting tidbits to the security and infrastructure
of the cluster.
sudo kubeadm config print init-defaults
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
- system:bootstrappers:kubeadm:default-node-token
token: abcdef.0123456789abcdef
ttl: 24h0m0s
usages:
<output_omitted>


---

## Exercise 15.2: Authentication and Authorization

```bash
cd ~/lfs458/ch15-security/
```

Kubernetes clusters have two types of users service accounts and normal users, but normal users are assumed
to be managed by an outside service. There are no objects to represent them and they cannot be added via an API
call, but service accounts can be added.
We will use RBAC to configure access to actions within a namespace for a new contractor, Developer Dan who will be


working on a new project.
1. Create two namespaces, one for production and the other for development.
kubectl create ns development
namespace/development created

kubectl create ns production
namespace/production created

2. View the current clusters and context available. The context allows you to configure the cluster to use, namespace and
user for kubectl commands in an easy and consistent manner.
kubectl config get-contexts
CURRENT
*

NAME
CLUSTER
kubernetes-admin@kubernetes

AUTHINFO
kubernetes

NAMESPACE
kubernetes-admin

3. Create a new user DevDan and assign a password of lftr@in.
sudo useradd -s /bin/bash DevDan
sudo passwd DevDan
Enter new UNIX password: lftr@in
Retype new UNIX password: lftr@in
passwd: password updated successfully

4. Generate a private key then Certificate Signing Request (CSR) for DevDan. On some Ubuntu 18.04 nodes a missing file
may cause an error with random number generation. The touch command should ensure one way of success.
openssl genrsa -out DevDan.key 2048
Generating RSA private key, 2048 bit long modulus
......+++
.........+++
e is 65537 (0x10001)

touch $HOME/.rnd
openssl req -new -key DevDan.key \
-out DevDan.csr -subj "/CN=DevDan/O=development"

5. Using thew newly created request generate a self-signed certificate using the x509 protocol. Use the CA keys for the
Kubernetes cluster and set a 45 day expiration. You’ll need to use sudo to access to the inbound files.
sudo openssl x509 -req -in DevDan.csr \
-CA /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key \
-CAcreateserial \
-out DevDan.crt -days 45


Signature ok
subject=/CN=DevDan/O=development
Getting CA Private Key

6. Update the access config file to reference the new key and certificate. Normally we would move them to a safe directory
instead of a non-root user’s home.
kubectl config set-credentials DevDan \
--client-certificate=/home/student/DevDan.crt \
--client-key=/home/student/DevDan.key
User "DevDan" set.

7. View the update to your credentials file. Use diff to compare against the copy we made earlier.
diff cluster-api-config .kube/config
16a,19d15
> - name: DevDan
>
user:
>
as-user-extra: {}
>
client-certificate: /home/student/DevDan.crt
>
client-key: /home/student/DevDan.key

8. We will now create a context. For this we will need the name of the cluster, namespace and CN of the user we set or
saw in previous steps.
kubectl config set-context DevDan-context \
--cluster=kubernetes \
--namespace=development \
--user=DevDan
Context "DevDan-context" created.

9. Attempt to view the Pods inside the DevDan-context. Be aware you will get an error.
kubectl --context=DevDan-context get pods
Error from server (Forbidden): pods is forbidden: User "DevDan"
cannot list pods in the namespace "development"

10. Verify the context has been properly set.
kubectl config get-contexts
CURRENT
*

NAME
CLUSTER
AUTHINFO
NAMESPACE
DevDan-context
kubernetes
DevDan
development
kubernetes-admin@kubernetes kubernetes kubernetes-admin

11. Again check the recent changes to the cluster access config file.
diff cluster-api-config .kube/config


<output_omitted>

12. We will now create a YAML file to associate RBAC rights to a particular namespace and Role.
cp /home/student/LFS458/SOLUTIONS/s_15/role-dev.yaml .
vim role-dev.yaml

role-dev.yaml
2
4
6
8
10

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
namespace: development
name: developer
rules:
- apiGroups: ["", "extensions", "apps"]
resources: ["deployments", "replicasets", "pods"]
verbs: ["list", "get", "watch", "create", "update", "patch", "delete"]
# You can use ["*"] for all verbs

13. Create the object. Check white space and for typos if you encounter errors.
kubectl create -f role-dev.yaml
role.rbac.authorization.k8s.io/developer created

14. Now we create a RoleBinding to associate the Role we just created with a user. Create the object when the file has
been created.
cp /home/student/LFS458/SOLUTIONS/s_15/rolebind.yaml .
vim rolebind.yaml

rolebind.yaml
2
4
6
8
10
12

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
name: developer-role-binding
namespace: development
subjects:
- kind: User
name: DevDan
apiGroup: ""
roleRef:
kind: Role
name: developer
apiGroup: ""

kubectl create -f rolebind.yaml


rolebinding.rbac.authorization.k8s.io/developer-role-binding created

15. Test the context again. This time it should work. There are no Pods running so you should get a response of No
resources found.


kubectl --context=DevDan-context get pods

No resources found in development namespace.

16. Create a new pod, verify it exists, then delete it.
kubectl --context=DevDan-context \
create deployment nginx --image=nginx
deployment.apps/nginx created

kubectl --context=DevDan-context get pods
NAME
nginx-7c87f569d-7gb9k

READY
1/1

STATUS
Running

RESTARTS

AGE
5s

kubectl --context=DevDan-context delete \
deploy nginx
deployment.apps "nginx" deleted

17. We will now create a different context for production systems. The Role will only have the ability to view, but not create
or delete resources. Begin by copying and editing the Role and RoleBindings YAML files.
cp role-dev.yaml role-prod.yaml
vim role-prod.yaml

role-prod.yaml
2
4
6
8

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
namespace: production
#<<- This line
name: dev-prod
#<<- and this line
rules:
- apiGroups: ["", "extensions", "apps"]
resources: ["deployments", "replicasets", "pods"]
verbs: ["get", "list", "watch"] #<<- and this one

cp rolebind.yaml rolebindprod.yaml
vim rolebindprod.yaml


rolebindprod.yaml
2
4
6
8
10
12

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
name: production-role-binding #<-- Edit to production
namespace: production
#<-- Also here
subjects:
- kind: User
name: DevDan
apiGroup: ""
roleRef:
kind: Role
name: dev-prod
#<-- Also this
apiGroup: ""

18. Create both new objects.
kubectl create -f role-prod.yaml
role.rbac.authorization.k8s.io/dev-prod created

kubectl create -f rolebindprod.yaml
rolebinding.rbac.authorization.k8s.io/production-role-binding created

19. Create the new context for production use.
kubectl config set-context ProdDan-context \
--cluster=kubernetes \
--namespace=production \
--user=DevDan
Context "ProdDan-context" created.

20. Verify that user DevDan can view pods using the new context.
kubectl --context=ProdDan-context get pods
No resources found in production namespace.

21. Try to create a Pod in production. The developer should be Forbidden.
kubectl --context=ProdDan-context create \
deployment nginx --image=nginx
Error from server (Forbidden): deployments.apps is forbidden:
User "DevDan" cannot create deployments.apps in the
namespace "production"

22. View the details of a role.
kubectl -n production describe role dev-prod


Name:
dev-prod
Labels:
<none>
Annotations: kubectl.kubernetes.io/last-applied-configuration=
{"apiVersion":"rbac.authorization.k8s.io/v1","kind":"Role"
,"metadata":{"annotations":{},"name":"dev-prod","namespace":
"production"},"rules":[{"api...
PolicyRule:
Resources
Non-Resource URLs Resource Names Verbs
------------------------- -------------- ----deployments
[]
[]
[get list watch]
deployments.apps
[]
[]
[get list watch]
<output_omitted>

23. Experiment with other subcommands in both contexts. They should match those listed in the respective roles.
24. OPTIONAL CHALLENGE STEP: Become the DevDan user. Solve any missing configuration errors. Try to create
a deployment in the development and the production namespaces. Do the errors look the same? Configure as
necessary to only have two contexts available to DevDan.
kubectl config get-contexts
CURRENT
*

NAME
DevDan-context
ProdDan-context

CLUSTER
kubernetes
kubernetes

AUTHINFO
DevDan
DevDan

NAMESPACE
development
production


---

## Exercise 15.3: Admission Controllers

```bash
cd ~/lfs458/ch15-security/
```

The last stop before a request is sent to the API server is an admission control plug-in. They interact with features
such as setting parameters like a default storage class, checking resource quotas, or security settings. A newer feature
(v1.7.x) is dynamic controllers which allow new controllers to be ingested or configured at runtime.
1. View the current admission controller settings. Unlike earlier versions of Kubernetes the controllers are now compiled into the server, instead of being passed at run-time. Instead of a list of which controllers to use we can enable and
disable specific plugins.
sudo grep admission \
/etc/kubernetes/manifests/kube-apiserver.yaml
- --enable-admission-plugins=NodeRestriction


16.1

Overview . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 368

16.2

Stacked Database . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 369

16.3

External Database . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 370

16.4

Labs

. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 371


16.1


Overview

Cluster High Availability

• Uses load balancer
• Three nodes for database quorum
• Stacked control planes and etcd nodes
• External etcd nodes
• Use FQDN for SSL connections

A newer feature of kubeadm is the integrated ability to join multiple cp nodes with co-located etcd databases. This allows
for higher redundancy and fault tolerance. As long as the database services the cluster will continue to run and catch up with
kubelet information should the cp node go down and be brought back online.
Three instances are required for etcd to be able to determine quorum (50if the data is accurate or corrupt the database could
become unavailable. Once etcd is able to determine quorum it will elect a leader and return to functioning as it had before
failure.
One can either co-locate the database with control planes or use an external etcd database cluster. The kubeadm command
makes the co-located deployment easier to use.
To ensure that workers and other control plans continue to have access it is a good idea to use a load balancer. The default
configuration leverages SSL, so you may need to configure the load balancer as a TCP pass through unless you want the
extra work of certificate configuration. As the certificates will be decoded only for particular node names it is a good idea to
use a FQDN instead of an ip address, although there are many possible ways to handle access.


16.2. STACKED DATABASE

16.2

Stacked Database

Collocated Databases

• Default database location
• Simpler to deploy and manager
• Requires at least three nodes
• Instance failure would remove essential services and database

The easiest way to gain higher availability is to use the kubeadm command and join at least two more cp servers to the cluster.
The command is almost the same as a worker join except an additional --control-plane flag and a certificate-key. The
key will probably need to be generated unless the other cp nodes are added within two hours of the cluster initialization.
Should a node fail you would lose both a control plane and a database. As the database is the one object that cannot be
rebuilt this may not be an important issue.


16.3


External Database

Non-collocated Databases

• Configure HA etcd cluster
• Manually copy PKI certificates
• Configure first control plane
• Add more control planes
• Add worker nodes

Using an external cluster of etcd allows for less interruption should a node fail. Creating a cluster in this manner requires a lot
more equipment to properly spread out services and takes more work to configure.
The external etcd cluster needs to be configured first. The kubeadm command has options to configure this cluster, or other
options are available. Once the etcd cluster is running the certificates need to be manually copied to the intended first control
plane node.
The kubeadm-config.yaml file needs to be populated with the etcd set to external, endpoints, and the certificate locations.
Once the first control plane is fully initialized the redundant control planes need to be added one at a time, each fully initialized
before the next is added.


16.4

Labs


---


---

# Chapter 16: High Availability
**Working directory: `~/lfs458/ch16-ha/`**

## Exercise 16.1: High Availability Steps

```bash
cd ~/lfs458/ch16-ha/
```


Overview
In this lab we will add two more control planes to our cluster, change taints and deploy an application to a particular
node, and test that we can access it from outside the cluster. The nodes will handle various infrastructure services and
the etcd database and should be sized accordingly.
The steps are presented in two ways. First the general steps for those interested in more of a challenge. Following that
will be the detailed steps found in previous labs.
You will need three more nodes. One to act as a load balancer, the other two will act as cp nodes for quorum. Log into each
and use the ip command to fill in the table with the IP addresses of the primary interface of each node. If using GCE nodes it
would be ens4, yours may be different. You may need to install software such an editor on the nodes.
Proxy Node
Second Control Plane
Third Control Plane
As the prompts may look similar you may want to change the terminal color or other characteristics to make it easier to keep
them distinct. You can also change the prompt using something like: PS1=”ha-proxy$ ”, which may help to keep the terminals
distinct.
High level steps:
1. Deploy a load balancer configured to pass through traffic on your new proxy node. HAProxy is easy to deploy using
online documentation. Start with forwarding traffic of the cp alias to just the working cp.
2. Install the Kubernetes software on the second and third cp nodes.
3. Use kubeadm join on the second cp, adding it to the cluster as another control plane using the node name.
4. Join the third cp as another control plane to the cluster using the node name.
5. Update the proxy to use all three cps backend IPs.
6. Temporarily shut down the first cp and monitor traffic.


---

## Exercise 16.2: Detailed Steps

```bash
cd ~/lfs458/ch16-ha/
```

Deploy a Load Balancer
While there are many options, both software and hardware, we will be using an open source tool HAProxy to configure a load
balancer.
1. Deploy HAProxy. Log into the proxy node. Update the repos then install a the HAProxy software. Answer yes, should
you the installation ask if you will allow services to restart.
sudo apt-get update ; sudo apt-get install -y haproxy vim
<output_omitted>

2. Edit the configuration file and add sections for the front-end and back-end servers. We will comment out the second and
third cp node until we are sure the proxy is forwarding traffic to the known working cp.


sudo vim /etc/haproxy/haproxy.cfg
....
defaults
log global
#<-- Edit these three lines, starting around line 23
option tcplog
mode tcp
....
errorfile 503 /etc/haproxy/errors/503.http
errorfile 504 /etc/haproxy/errors/504.http
frontend proxynode
bind *:80
bind *:6443
stats uri /proxystats
default_backend k8sServers

#<-- Add the following lines to bottom of file

backend k8sServers
balance roundrobin
server cp 10.128.0.24:6443 check #<-- Edit these with your IP addresses, port, and hostname
#
server secondcp 10.128.0.30:6443 check #<-- Comment out until ready
#
server thirdcp 10.128.0.66:6443 check
#<-- Comment out until ready
listen stats
bind :9999
mode http
stats enable
stats hide-version
stats uri /stats

3. Restart the haproxy service and check the status. You should see the frontend and backend proxies report being
started.
sudo systemctl restart haproxy.service
sudo systemctl status haproxy.service
<output_omitted>
Aug 08 18:43:08 ha-proxy systemd[1]: Starting HAProxy Load Balancer...
Aug 08 18:43:08 ha-proxy systemd[1]: Started HAProxy Load Balancer.
Aug 08 18:43:08 ha-proxy haproxy-systemd-wrapper[13602]: haproxy-systemd-wrapper:
Aug 08 18:43:08 ha-proxy haproxy[13603]: Proxy proxynode started.
Aug 08 18:43:08 ha-proxy haproxy[13603]: Proxy proxynode started.
Aug 08 18:43:08 ha-proxy haproxy[13603]: Proxy k8sServers started.
Aug 08 18:43:08 ha-proxy haproxy[13603]: Proxy k8sServers started.

4. On the cp Edit the /etc/hosts file and comment out the old and add a new k8scp alias to the IP address of the proxy
server.
sudo vim /etc/hosts
10.128.0.64 k8scp
#10.128.0.24 k8scp
127.0.0.1 localhost
....

#<-- Add alias to proxy IP
#<-- Comment out the old alias, in case its needed

5. Use a local browser to navigate to the public IP of your proxy server. The http://34.69.XX.YY:9999/stats is an example
your IP address would be different. Leave the browser up and refresh as you run following steps. You can find your
public ip using curl. Your IP will be different than the one shown below.
ha-proxy$ curl ifconfig.io


34.69.73.159

Figure 16.1: Initial HAProxy Status

6. Check the node status from the cp node then check the proxy statistics. You should see the byte traffic counter increase.
kubectl get nodes
NAME
cp
worker

STATUS
Ready
Ready

ROLES
control-plane
<none>

AGE
2d6h
2d3h

VERSION
v1.34.1
v1.34.1

Install Software
We will add two more control planes with stacked etcd databases for cluster quorum. You may want to open up two more
PuTTY or SSH sessions and color code the terminals to keep track of the nodes.
Initialize the second cp before adding the third cp
1. Configure and install the kubernetes software on the second cp. Use the same steps as when we first set up the
cluster, earlier in the course. You may want to copy and paste from earlier commands in your history to make these
steps easier. All the steps up to but not including kubeadm init or kubeadm join A script k8sWorker.sh has been
included in the course tarball to make this process go faster, if you would like. View and edit the script to be the correct
version before running it.
2. Install the software on the third cp using the same commands.


Join Control Plane Nodes
1. Edit the /etc/hosts file ON ALL NODES to ensure the alias of k8scp is set on each node to the proxy IP address.
Your IP address may be different.
sudo vim /etc/hosts
10.128.0.64 k8scp
#10.128.0.24 k8scp
127.0.0.1 localhost
....

2. On the first cp create the tokens and hashes necessary to join the cluster. These commands may be in your history
and easier to copy and paste.
3. Create a new token.
sudo kubeadm token create
jasg79.fdh4p279l320cz1g

4. Create a new SSL hash.
openssl x509 -pubkey \
-in /etc/kubernetes/pki/ca.crt | openssl rsa \
-pubin -outform der 2>/dev/null | openssl dgst \
-sha256 -hex | sed 's/ˆ.* //'
f62bf97d4fba6876e4c3ff645df3fca969c06169dee3865aab9d0bca8ec9f8cd

5. Create a new cp certificate to join as a cp instead of as a worker.
sudo kubeadm init phase upload-certs --upload-certs
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
5610b6f73593049acddee6b59994360aa4441be0c0d9277c76705d129ba18d65

6. On the second cp use the previous output to build a kubeadm join command. Please be aware that multi-line copy
and paste from Windows and some MacOS has paste issues. If you get unexpected output copy one line at a time.
sudo kubeadm join k8scp:6443 \
--token jasg79.fdh4p279l320cz1g \
--discovery-token-ca-cert-hash sha256:f62bf97d4fba6876e4c3ff645df3fca969c06169dee3865aab9d0bca8ec9f8cd \
--control-plane --node-name=secondcp --certificate-key \
5610b6f73593049acddee6b59994360aa4441be0c0d9277c76705d129ba18d65
[preflight] Running pre-flight checks
[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended
driver \
is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
<output_omitted>

7. Return to the first cp node and check to see if the node has been added and is listed as a cp.
kubectl get nodes


NAME
cp
secondcp
worker

STATUS
Ready
Ready
Ready

ROLES
control-plane
control-plane
<none>

AGE
2d6h
10m
2d3h

VERSION
v1.34.1
v1.34.1
v1.34.1

8. Copy and paste the kubeadm join command to the third cp. Remember to change the node name to thirdcp Then
check that the third cp has been added.
kubectl get nodes
NAME
cp
secondcp
thirdcp
worker

STATUS
Ready
Ready
Ready
Ready

ROLES
control-plane
control-plane
control-plane
<none>

AGE
2d6h
13m
3m
2d3h

VERSION
v1.34.1
v1.34.1
v1.34.1
v1.34.1

9. Copy over the configuration file as suggested in the output at the end of the join command. Do this on both newly added
cp nodes.
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

10. On the Proxy node. Edit the proxy to include all three cp nodes then restart the proxy.
sudo vim /etc/haproxy/haproxy.cfg
....
backend k8sServers
balance roundrobin
server cp
10.128.0.24:6443 check
server secondcp 10.128.0.30:6443 check
server thirdcp
10.128.0.66:6443 check
....

#<-- Edit/Uncomment these lines
#<--

sudo systemctl restart haproxy.service

11. View the proxy statistics. When it refreshes you should see three new back-ends. As you check the status of the nodes
using kubectl get nodes you should see the byte count increase on each node indicating each is handling some of the
requests.


Figure 16.2: Multiple HAProxy Status

12. View the logs of the newest etcd pod. Leave it running, using the -f option in one terminal while running the following
commands in a different terminal. As you have copied over the cluster admin file you can run kubectl on any cp.
kubectl -n kube-system get pods |grep etcd
etcd-cp
etcd-secondcp
etcd-thirdcp

1/1
1/1
1/1

Running
Running
Running

0

2d12h
22m
18m

kubectl -n kube-system logs -f etcd-thirdcp
....
2025-10-09 01:58:03.768858 I | mvcc: store.index: compact 300473
2025-10-09 01:58:03.770773 I | mvcc: finished scheduled compaction at 300473 (took 1.286565ms)
2025-10-09 02:03:03.766253 I | mvcc: store.index: compact 301003
2025-10-09 02:03:03.767582 I | mvcc: finished scheduled compaction at 301003 (took 995.775µs)
2025-10-09 02:08:03.785807 I | mvcc: store.index: compact 301533
2025-10-09 02:08:03.787058 I | mvcc: finished scheduled compaction at 301533 (took 913.185µs)

13. Log into one of the etcd pods and check the cluster status, using the IP address of each server and port 2379. Your IP
addresses may be different. Exit back to the node when done.
kubectl -n kube-system exec -it etcd-cp -- /bin/sh

etcd pod
/ # ETCDCTL_API=3 etcdctl -w table \
--endpoints 10.128.0.66:2379,10.128.0.24:2379,10.128.0.30:2379 \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key \
endpoint status


+------------------+------------------+---------+---------+-----------+-----------+------------+
|
ENDPOINT
|
ID
| VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT
INDEX |
+------------------+------------------+---------+---------+-----------+-----------+------------+
| 10.128.0.66:2379 | 2331065cd4fb02ff | 3.5.7 |
24 MB |
true |
11 |
392573 |
| 10.128.0.24:2379 | d2620a7d27a9b449 | 3.5.7 |
24 MB |
false |
11 |
392573 |
| 10.128.0.30:2379 | ef44cc541c5f37c7 | 3.5.7 |
24 MB |
false |
11 |
392573 |
+------------------+------------------+---------+---------+-----------+-----------+------------+

Test Failover
Now that the cluster is running and has chosen a leader we will shut down containerd, which will stop all containers on that
node. This will emulate an entire node failure. We will then view the change in leadership and logs of the events.
1. Shut down the service on the node which shows IS LEADER set to true.
sudo systemctl stop containerd.service

If you chose cri-o as the container engine then the cri-o service and conmon processes are distinct. It may be easier to
reboot the node and refresh the HAProxy web page until it shows the node is down. It may take a while for the node to
finish the boot process. The second and third cp should work the entire time.
sudo reboot

2. You will probably note the logs command exited when the service shut down. Run the same command and, among
other output, you’ll find errors similar to the following. Note the messages about losing the leader and electing a new
one, with an eventual message that a peer has become inactive.
kubectl -n kube-system logs -f etcd-thirdcp
....
2025-10-09 02:11:39.569827 I | raft: 2331065cd4fb02ff [term: 9] received a MsgVote message with
higher \
term from ef44cc541c5f37c7 [term: 10]
2025-10-09 02:11:39.570130 I | raft: 2331065cd4fb02ff became follower at term 10
2025-10-09 02:11:39.570148 I | raft: 2331065cd4fb02ff [logterm: 9, index: 355240, vote: 0] cast
MsgVote \
for ef44cc541c5f37c7 [logterm: 9, index: 355240] at term 10
2025-10-09 02:11:39.570155 I | raft: raft.node: 2331065cd4fb02ff lost leader d2620a7d27a9b449 at
term 10
2025-10-09 02:11:39.572242 I | raft: raft.node: 2331065cd4fb02ff elected leader ef44cc541c5f37c7
at \
term 10
2025-10-09 02:11:39.682319 W | rafthttp: lost the TCP streaming connection with peer
d2620a7d27a9b449 \
(stream Message reader)
2025-10-09 02:11:39.682635 W | rafthttp: lost the TCP streaming connection with peer
d2620a7d27a9b449 \
(stream MsgApp v2 reader)
2025-10-09 02:11:39.706068 E | rafthttp: failed to dial d2620a7d27a9b449 on stream MsgApp v2 \
(peer d2620a7d27a9b449 failed to find local node 2331065cd4fb02ff)
2025-10-09 02:11:39.706328 I | rafthttp: peer d2620a7d27a9b449 became inactive (message send to
peer failed)
....


3. View the proxy statistics. The proxy should show the first cp as down, but the other cp nodes remain up.

Figure 16.3: HAProxy Down Status

4. View the status using etcdctl from within one of the running etcd pods. You should get an error for the endpoint you
shut down and a new leader of the cluster.
kubectl -n kube-system exec -it etcd-secondcp -- /bin/sh

etcd pod
/ # ETCDCTL_API=3 etcdctl -w table \
--endpoints 10.128.0.66:2379,10.128.0.24:2379,10.128.0.30:2379 \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key \
endpoint status
Failed to get the status of endpoint 10.128.0.66:2379 (context deadline exceeded)
+------------------+------------------+---------+---------+-----------+-----------+------------+
|
ENDPOINT
|
ID
| VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT
INDEX |
+------------------+------------------+---------+---------+-----------+-----------+------------+
| 10.128.0.24:2379 | d2620a7d27a9b449 | 3.5.7 |
24 MB |
true |
12 |
395729 |
| 10.128.0.30:2379 | ef44cc541c5f37c7 | 3.5.7 |
24 MB |
false |
12 |
395729 |
+------------------+------------------+---------+---------+-----------+-----------+------------+

5. Turn the containerd service back on. You should see the peer become active and establish a connection.
sudo systemctl start containerd.service
kubectl -n kube-system logs -f etcd-thirdcp
....
2025-10-09 02:45:11.337669 I | rafthttp: peer d2620a7d27a9b449 became active
2025-10-09 02:45:11.337710 I | rafthttp: established a TCP streaming connection with peer\
d2620a7d27a9b449 (stream MsgApp v2 reader)
....

6. View the etcd cluster status again. Experiment with how long it takes for the etcd cluster to notice failure and choose a
new leader with the time you have left.


17.1

Evaluation Survey . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 379


17.1


Evaluation Survey

Evaluation Survey
Thank you for taking this course brought to you by The Linux Foundation.
Your comments are important to us and we take them seriously, both to
measure how well we fulfilled your needs and to help us improve future
sessions.
• Please Evaluate your training using the link your
instructor will provide.
• Please be sure to check the spelling of your name
and use correct capitalization as this is how your
name will appear on the certificate.
• This information will be used to generate your Certificate of Completion, which will be sent to the Figure 17.1: Course Survey
email address you have supplied, so make sure it
is correct.


Appendices


Appendix A

Domain Review

A.1

CKA Exam . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 384

A.2

Exam Domain Review . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 385


A.1

APPENDIX A. DOMAIN REVIEW

CKA Exam

Domain Review

• Review the Candidate Handbook
• Find the proper version of the Curriculum Overview
• Write out each bullet point and steps
• Recognize good YAML examples in documentation
• Practice at speed

Navigate to the https://www.cncf.io/certification/cka/ page and scroll down to look at the Exam Resources section. Read
through each of the resources as you prepare for the CKA exam. Begin by looking through the Candidate Handbook. You
can either view it as several pages, or download the entire handbook as a pdf file. Read through each section. Take note of
the Resources allowed during exam section as this will list the URLs and browser you can use for the exam. Read the
rest of the document carefully.
Also navigate to the Exam Curriculum https://github.com/cncf/curriculum page. You should find PDF files for the CKA, CKAD,
and the CKS exam. There may be more than one PDF. Ensure the PDF you use matches the version of the exam you are
going to take. These update on a regular basis. It is your responsibility to revisit and ensure you are looking at the correct
version of the file.
Write out each of the knowledge, skills and abilities listed in the curriculum. Ensure you know the command line steps to
complete each of the items. Warning: The word understand means more than you have a general idea. It means you are
able to create, integrate, troubleshoot, and properly remove that object.
The exam now uses a secure browser, so you will not be able to use your own bookmarks. It is important you are familiar with
the allowed documentation, so that you can use known, working YAML with minimal searching. Test each for the version of
Kubernetes the exam is using. If the version changes you must re-check each YAML file to ensure it still works. It is better to
know it works than discover it during the exam and have to troubleshoot the issue.
Many people simply run out of time during the exam. Use a timer and practice the curriculum at the pace necessary to
complete each item during the exam, and have enough time to verify and troubleshoot your own work.


A.2. EXAM DOMAIN REVIEW

A.2

Exam Domain Review

Exercise A.1: Are you Ready?
1. If you have not searched for working and tested YAML examples, and can easily search for them again for all of the
subjects the domain review mentions for the exam, you may:
a. Run out of time while searching for good YAML examples.
b. Make more mistakes.
c. Use YAML that isn’t proper for the Kubernetes version of the exam and waste time trying to troubleshoot the issue.
d. All of the above.
2. Answer all that apply. In the context of the Curriculum Overview the term Understand when stating what a candidate
should be able to do means:
a. You only have a general idea what the object does.
b. You can create the object.
c. You can configure and integrate the object with other objects.
d. You can properly update and test the object.
e. You can troubleshoot the object.
3. Have you practiced creating, integrating, and troubleshooting all of the domain review items at speed?
a. No. I kind of did the exercises. So I’m good.
b. No. I did the labs twice over a two week period.
c. Yes. I know the exam is intense and practiced with a clock running to make sure I can get everything done and also
check my work.

Solution A.1
Are You Ready?
1. d.
2. b, c, d, e.
3. Hopefully c.

Exercise A.2: Preparing for the CKA Exam
Very Important
The source pages and content in this review could change at any time. IT IS YOUR RESPONSIBILITY TO CHECK
THE CURRENT INFORMATION.

Before Taking Exam
Use this exercise as a resource after you complete the course but before you take the exam. Review the resources, know what
good YAML looks like, and practice creating and working with objects at exam speed to assist with review and preparation.
1. Using a browser go to https://www.cncf.io/certification/cka/ and read through the program description.


APPENDIX A. DOMAIN REVIEW

2. In the Exam Resources section open the Curriculum Overview and Candidate-handbook in new tabs. Both of these
should be read and understood prior to sitting for the exam.
3. Navigate to the Curriculum Overview tab. You should see links for domain information for various versions of the exam.
Select the latest version, such as CKA Curriculum V1.25.pdf. The versions you see may be different. You should see
a new page showing a PDF.
4. Read through the document. Be aware that the term Understand, such as Understand Services, is more than just
knowing they exist. In this case expect it to also mean create, configure, update, and troubleshoot.
5. Using only the exam-allowed URLs and sub-domains search for YAML examples for each domain or skill item. Ensure it
works for the version of the exam you are taking, as the YAML may not have been re-tested after a new release. Become
familiar with out to find each good example again, so you can find the page again during the exam.
6. Using a timer see how long it takes you to create and verify the objects listed below. Write down the time. Try it again
and see how much faster you can complete and test each step.
”Practice until you get it right. Then practice until you can’t get it wrong” -Unknown

Domain Review Items
This list is copied from competency domains found on the PDF. Again, it remains your responsibility to check the web
page for any changes to this list.
• Cluster Architecture, Installation & Configuration
– Manage role based access control (RBAC)
– Use Kubeadm to install a basic cluster
– Manage a highly-available Kubernetes cluster
– Provision underlying infrastructure to deploy a Kubernetes cluster
– Preform a version upgrade on a Kubernetes cluster using Kubeadm
– Implement etcd backup and restore
• Workloads & Scheduling
– Understand deployments and how to preform rolling updates and rollbacks
– Use ConfigMaps and Secrets to configure applications
– Know how to scale applications
– Understand the primitives used to create robust, self-healing, application deployments
– Understand how resource limits can affect Pod scheduling
– Awareness of manifest management and common templating tools
• Services & Networking
– Understand host networking configuration on the cluster nodes
– Understand connectivity between Pods
– Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
– Know how to use Ingress controllers and Ingress resources
– Know how to configure and use CoreDNS
– Choose an appropriate container network interface plugin
• Storage
– Understand storage classes, persistent volumes


A.2. EXAM DOMAIN REVIEW
– Understand volume mode, access modes and reclaim policies for volumes
– Understand persistent volume claims primitive
– Know how to configure applications with persistent storage
• Troubleshooting
– Evaluate cluster and node logging
– Understand how to monitor applications
– Manage container stdout & stderr logs
– Troubleshoot application failure
– Troubleshoot cluster component failure
– Troubleshoot networking

Exercise A.3: Practicing Skills
This exercise is to help you practice your skills. It does not cover all the items listed in the domain review guide. You should
develop your own steps to build a full list of skill tests and steps.
Also note that all the detailed steps are not included. You should be able to complete these steps without being told what to
type.
In a work or exam environment you may not be told exactly what to do or how to do it. The following steps are meant to get
you used to thinking about solutions when the exact need isn’t clear.
1. Find and use the review1.yaml file included in the course tarball. Use the find output and copy the YAML file to your
home directory. Use kubectl create to create the object. Determine if the pod is running. Fix any errors you may
encounter. The use of kubectl describe may be helpful.
find ~ -name review1.yaml
cp <copy-paste-from-above>

.

kubectl create -f review1.yaml

2. After you get the pod running remove any pods or services you may have created as part of the review before moving
on to the next section. For example:
kubectl delete -f review1.yaml

3. Use the review2.yaml file to create a non-working deployment. Fix the deployment such that both containers are
running and in a READY state. The web server listens on port 80, and the proxy listens on port 8080.
4. View the default page of the web server. When successful verify the GET activity logs in the container log. The message
should look something like the following. Your time and IP may be different.
192.168.124.0 - - [3/Dec/2024:03:30:31 +0000] "GET / HTTP/1.1" 200 612 "-"
"curl/7.58.0" "-"

5. Find and use the review4.yaml file to create a pod, and verify it’s running
6. Edit the pod such that it only runs on your worker node using the nodeSelector label.
7. Determine the CPU and memory resource requirements of design-pod1.
8. Edit the pod resource requirements such that the CPU limit is exactly twice the amount requested by the container.
(Hint: subtract .22)
9. Increase the memory resource limit of the pod until the pod shows a Running status. This may require multiple edits
and attempts. Determine the minimum amount necessary for the Running status to persist at least a minute.


APPENDIX A. DOMAIN REVIEW

10. Use the review5.yaml file to create several pods with various labels.
11. Using only the –selector value tux to delete only those pods. This should be half of the pods. Hint, you will need to
view pod settings to determine the key value as well.
12. Create a new cronjob which runs busybox and the sleep 30 command. Have the cronjob run every three minutes.
View the job status to check your work. Change the settings so the pod runs 10 minutes from the current time, every
week. For example, if the current time was 2:14PM, I would configure the job to run at 2:24PM, every Monday.
13. Delete any objects created during this review. You may want to delete all but the cronjob if you’d like to see if it runs in
10 minutes. Then delete that object as well.
14. Create a new secret called specialofday using the key entree and the value meatloaf.
15. Create a new deployment called foodie running the nginx image.
16. Add the specialofday secret to pod mounted as a volume under the /food/ directory.
17. Execute a bash shell inside a foodie pod and verify the secret has been properly mounted.
18. Update the deployment to use the nginx:1.12.1-alpine image and verify the new image is in use.
19. Roll back the deployment and verify the typical, current stable version of nginx is in use again.
20. Create a new 200M NFS volume called reviewvol using the NFS server configured earlier in the lab.
21. Create a new PVC called reviewpvc which will uses the reviewvol volume.
22. Edit the deployment to use the PVC and mount the volume under /newvol
23. Execute a bash shell into the nginx container and verify the volume has been mounted.
24. Delete any resources created during this review.
25. Create a new deployment which uses the nginx image.
26. Create a new LoadBalancer service to expose the newly created deployment. Test that it works.
27. Create a new NetworkPolicy called netblock which blocks all traffic to pods in this deployment only. Test that all traffic
is blocked to deployment.
28. Create a pod running nginx and ensure traffic can reach that deployment.
29. Update the netblock policy to allow traffic to the pod on port 80 only. Test that you can now access the default nginx
web page.
30. Find and use the review6.yaml file to create a pod.
kubectl create -f review6.yaml

31. View the status of the pod.
32. Use the following commands to figure out why the pod has issues.
kubectl get pod securityreview
kubectl describe pod securityreview
kubectl logs securityreview

33. After finding the errors, log into the container and find the proper id of the nginx user.
34. Edit the pod such that the securityContext is in place and allows the web server to read the proper configuration files.
35. Create a new serviceAccount called securityaccount.
36. Create a ClusterRole named secrole which only allows create, delete, and list of pods in all apiGroups.


A.2. EXAM DOMAIN REVIEW

37. Bind the new clusterRole to the new serviceAccount.
38. Locate the token of the securityaccount. Create a file called /tmp/securitytoken. Put only the value of token: is
equal to, a long string that may start with eyJh and be several lines long. Careful that only that string exists in the file.
39. Remove any resources you have added during this review
40. Create a new pod called webone, running the nginx service. Expose port 80.
41. Create a new service named webone-svc. The service should be accessible from outside the cluster.
42. Update both the pod and the service with selectors so that traffic for to the service IP shows the web server content.
43. Change the type of the service such that it is only accessible from within the cluster. Test that exterior access no longer
works, but access from within the node works.
44. Deploy another pod, called webtwo, this time running the wlniao/website image. Create another service, called
webtwo-svc such that only requests from within the cluster work. Note the default page for each server is distinct.
45. Test DNS names and verify CoreDNS is properly functioning.
46. Install and configure an ingress controller such that requests for webone.com see the nginx default page, and requests
for webtwo.org see the wlniao/website default page. It does not matter which ingress controller you use.
47. Remove any resources created in this review.
48. Install a new cluster using an recent, previous version of Kubernetes. Backup etcd, then properly upgrade the entire
cluster.
49. Create a pod running busybox without the scheduler being consulted.
50. Continue to create objects, integrate them with other objects and troubleshoot until each domain item has been covered.


---
