newslynx-automation
===================

An Ansible Playbook + Vagrantfile for automating a `newslynx-core` and `newslynx-app` install.

## Getting Started

You can automate the NewsLynx Installation in one of two ways

1. Into a virtual machine on your local computer.
2. Onto an Amazon Web Services EC2 machine.

For both options you must first install [`vagrant`](https://www.vagrantup.com/) and [`ansible`](http://docs.ansible.com/). Each option has other dependencies that will be covered below.

These installers are fairly straightforward if you're on Mac OS X. If you're installing from a Linux command-line, such as if you're deploying from an EC2 machine, see our [Vagrant and Ansible Linux EC2](EC2-LINUX-INSTALL-GUIDE.md) install guide.

Next, you should rename [`config.yaml.sample`](config.yaml.sample) to `config.yaml` and fill it out according to your preferences.  For more information on how this file is structured, refer to the [configuration docs](http://newslynx.readthedocs.org/en/latest/config.html).

## Provisioning locally 

For a local setup, install [`virtualbox`](https://www.virtualbox.org/wiki/Downloads).

One you've done that, run the following command:

```shell
$ make vb_init
``` 

This will execute the ansible-playbook located in [`provisioning/main.yaml`](provisioning/main.yaml). This will take about 20-30 minutes to download all the dependencies and configure the machine.  Once it is finished you should be able to access the Newslynx API on your local machine on port `5001`:

```
curl http://localhost:5001/api/v1/me\?apikey=<your-apikey>
```

The app will be running at [http://localhost:3001](http://localhost:3001).  You'll be able to login with the `super_user_email` and `super_user_password` that you set in in `config.yaml`.

For more information on what to do next, please refer to our [getting started docs](http://newslynx.readthedocs.org/en/latest/getting-started.html).

## Provisioning on AWS 

To deploy an AWS instance, you first need to install the `vagrant` AWS plugin and dummy box:

```
vagrant plugin install vagrant-aws
vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box
```

Next, save [`secrets.yaml.sample`](secrets.yaml.sample) as `secrets.yaml` and insert your AWS credentials. Now, open up [`servers.yaml`](servers.yaml) and configure the options under `aws`. For more details on these options, refer to the [AWS Docs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html). 

The default ami, `ami-d05e75b8` is the basic Ubuntu 14.04 amd64 image for the us-east-1 region. You will have to change this if you are deploying to another region. To find out what you should change it to, visit the AWS "Launch an EC2 instance" page, scroll down to the Ubuntu 14.04 image and copy its AMI id. 

Once these are set, provision your EC2 box with the following command:

```
vagrant up --provider=aws
```

This will execute the ansible-playbook located in [`provisioning/main.yaml`](provisioning/main.yaml). This will take about 20-30 minutes to download all the dependencies and configure the machine.

Get the url of your EC2 machine and visit the app at `http://<ec2-url>.com:3000`.

**NOTE** If you get an error saying that the "Elastic IP could not be provisioned" it means that you have too many IP addresses already allocated to your AWS account. You can remove them by going to the "Elastic IP" panel in your EC2 dashboard and removing them.

## Operations

Once your machine is provisioned, in either setup, login with the following command:

```
vagrant ssh
```

You can now check the public IP of the box by running:

```
curl http://icanhazip.com
```

If you copy the output of this command and paste it into a browser, you should be directed to a login page for NewsLynx. 

You can also query the API on port `5000` of this same address:

```
curl http://<newslynx-location>:5000/api/v1/me 
```

## Debugging 

If something goes wrong with the deployment (which you should see in the logs), you can log into the VM using the same command:

```
vagrant ssh
```

All of the applications are installed as root, so you'll need to first:

```
sudo su
```

And then:

```
cd /opt/newslynx/
```

To check the logs of running processes, type:

```
tail -n 100 logs/app.log
```

If you should like to re-run the ansible playbook without fully destroying the Virtual Machine, run:

```
vagrant provision
```

## Problems?

If you have any problems with this process, please report an issue to our [opportunity tracker](https://github.com/newslynx/opportunities/issues).


## How does this work ?

`ansible` is a very fancy framework for running very ugly shell commands.  Each of the files located in [`provisioning/`](provisioning/), with the exclusion of `main.yaml`, represents an ansible "role" or a particular dependency that `newslynx` requires. These roles are fulfilled by running each of the commands listed in their respective files. Some "roles" - like [`newslynx-app`](provisioning/newslynx-app.yaml) and [`newslynx-core`](provisioning/newslynx-core.yaml) - rely on on other "roles".  These are "included" at the top of each respective file.  Most roles also require certain configuration variables which are set in [`provisioning/vars/`](provisioning/vars/).  Some of these roles also require their own configuration files.  To generate these, we create "templates" which we populate with the configuration variables.  These files are stored in [`provisioning/templates/`](provisioning/templates).

When you run `vagrant up`, the following steps are executed:

1. A virtual machine is provisioned. The specs of this machine are included in [`servers.yaml`](servers.yaml).
2. The ansible ["playbook"](provisioning/main.yaml), or list of all of newslynx's required roles, is executed on the Virtual Machine.
3. If all goes well, `newslynx-core` and `newslynx-app` will start up within the Virtual Machine on ports `5000`and `3000`, respectively. These will be forwarded to your local machine on ports `3001` and `5001`. On AWS, they will remain `5000` and `3000`.

## Colophon

- Ubuntu 14.0.4 (The operating system.)
- Python 2.7.6 (The langauge `newslynx-core` is written in.)
- Node 0.12 (The language `newslynx-app` is written in.)
- Postgres 9.3 (`newslynx-core`'s primary datastore.)
- Redis 2.8.4 (`newslynx-core`'s caching layer and task queue.)
- Supervisor (`newslynx-core`'s daemeon manager.)
- Nginx (The proxy server that sits in front of `newslynx-app` and the rest of the world.)
- Forever (Not currently used but is around for potential future convenience)

