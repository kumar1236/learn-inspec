# Try InSpec

InSpec is an open-source testing framework by Chef that enables you to specify compliance, security, and other policy requirements.

Built on the Ruby programming language, InSpec tests are meant to be human-readable.


### Steps to use Inspec, in this module you'll scan Docker containers running Linux to get a sense of how InSpec works. The concepts apply equally to Windows.

### 1. Install Docker and Docker Compose

Refer:
https://docs.docker.com/compose/install/

### 2. Set up your environment

The setup includes two systems – one that acts as your workstation and a second that acts as the target, or the system you want to monitor.

Both Docker images are based on Ubuntu 16.04. The workstation image comes with inspec, the InSpec command-line interface (CLI), and several utilities pre-installed. In practice, you would write InSpec tests from the workstation you use daily. The workstation image provides a learning environment for you to see quickly how InSpec works.

Start by creating a working directory. We recommend ~/learn-inspec.

```
mkdir ~/learn-inspec;cd ~/learn-inspec
```

Next, get the Docker Compose file. Run the command that matches your system to download a file named docker-compose.yml.

```
wget https://raw.githubusercontent.com/learn-chef/inspec/master/docker-compose.yml
```

Next, run the following docker-compose command to retrieve the latest workstation images.
```
docker-compose pull
```

Next, run the following docker-compose command to start the containers. The -d argument starts the containers in the background.
```
docker-compose up -d
```


Now that your containers are running in the background, run this command to start an interactive Bash session on the workstation container.
```
docker exec -it workstation bash
```

### 3. Detect using InSpec

From your workstation container, start by running inspec help to see what commands are available.
```
inspec help
```

run inspec detect to list information about the operating system.

```
inspec detect

== Platform Details

Name:      ubuntu
Families:  debian, linux, unix
Release:   16.04
Arch:      x86_64
```

### 4. Download the auditd profile

Start by moving to the /root directory on your container.
```
cd /root
```
Next, run this git clone command to download a profile named auditd.
```
git clone https://github.com/learn-chef/auditd.git
Cloning into 'auditd'...
```
Run tree to see what's in the auditd profile.
```
tree auditd
auditd
├── README.md
├── controls
│   └── example.rb
├── inspec.lock
└── inspec.yml
 
1 directory, 4 files
```
These files are collectively called a profile.

The file /root/auditd/controls/example.rb contains your InSpec test. Print it to the console.
```
cat auditd/controls/example.rb
describe package('auditd') do
  it { should be_installed }
end 
```
Run the auditd profile
```
inspec exec /root/auditd
 
Profile: InSpec Profile (auditd)
Version: 0.1.0
Target:  local://
 
  System Package auditd
     ✔  should be installed
 
Test Summary: 1 successful, 0 failures, 0 skipped
```
If you had not installed the auditd package, the test would fail.

### 5.  Run the auditd profile Remotly

To run a profile remotely, you run inspec exec much like you just did. However, you also specify the -t argument to specify the URI of your target system. Here, you'll use the ssh:// scheme to target your Docker container over SSH.

```
inspec exec auditd -t ssh://root:password@target
```
Format output
```
inspec exec auditd -t ssh://root:password@target --reporter=json | jq .
```

Package your profile

run the following inspec check command to verify your profile is free of errors.
```
inspec check auditd
```
Now run the following inspec archive command to archive your profile.
```
inspec archive auditd
ls | grep auditd
```
You can run InSpec directly from an archive. To see this, run inspec exec like this.
```
inspec exec auditd-0.1.0.tar.gz -t ssh://root:password@target
```
You can publish your archive to a location where your systems can access it. You can also run the archived profile from that archive location. We've provided a copy of auditd-0.1.0.tar.gz on GitHub so you can see this in action. Run it like this.
```
inspec exec https://github.com/learn-chef/auditd/releases/download/v0.1.0/auditd-0.1.0.tar.gz -t ssh://root:password@target
```

### 6. Use a community profile

You can browse InSpec profiles on Chef Supermarket, supermarket.chef.io/tools. You can also see what's available from the InSpec CLI.

Run inspec supermarket profiles to see what's available.
```
inspec supermarket profiles
```
You can run inspec supermarket info to get more info about a profile. As you explore the available profiles, you might discover the dev-sec/linux-baseline profile. Run this command to learn more about it.
```
inspec supermarket info dev-sec/linux-baseline
```
Run the linux-baseline profile
run the following inspec supermarket exec command.
```
inspec supermarket exec dev-sec/linux-baseline -t ssh://root:password@target
```
Run the linux-baseline profile
Let's try this profile on your container. To do so, run the following inspec supermarket exec command.
```
inspec supermarket exec dev-sec/linux-baseline -t ssh://root:password@target
```
Run only certain controls
The previous command produced lots of output. What if you wanted to focus only to the tests contained in the "package-08" control, which test the auditd package?

One way might be to run the profile and search for the control's name in the output. Try it like this.
```
inspec exec https://github.com/dev-sec/linux-baseline -t ssh://root:password@target | grep -A 7 package-08
```
A better way might be to specify the --controls argument to run only certain controls; in this case, the "package-08" control.

Run the inspec exec command again, this time specifying the --controls package-08 argument.
```
inspec exec https://github.com/dev-sec/linux-baseline -t ssh://root:password@target --controls package-08
```





















 









