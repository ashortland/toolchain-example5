Toolchain example #5
====================

This example shows how to automate application build and deployment using a loosely-coupled toolchain

Application
-----------
 
DTO Lab's fork of the [Seam Examples](https://github.com/dtolabs/seam-examples) Booking application running on JBoss with MySQL.

Infrastructure
--------------

A single pre-provisioned Red Hat Linux (tested on "CentOS release 6.2") instance for build, repository, deployment, application and database services.

Toolchain
---------

* Souce code management: GitHub/Git
* Build tool: Maven and Rerun
* Build console: Jenkins
* Package format: RPM
* Package repository: Jenkins
* Deployment console: Rundeck
* Modular automation: Rerun

Requirements
------------

* System requirements:
    * 2GB RAM, 8 GB disk
    * 64-bit CentOS Linux 6.2 or later
    * Internet access (to GitHub and standard Yum repositories)
    * Disable firewall configuration:
<pre>
[root@centos63-toolchain-example5 ~]# chkconfig iptables off
[root@centos63-toolchain-example5 ~]# chkconfig ip6tables off
[root@centos63-toolchain-example5 ~]# service iptables stop
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Setting chains to policy ACCEPT: filter          [  OK  ]
iptables: Unloading modules:                               [  OK  ]
^[[root@centos63-toolchain-example5 ~]# service ip6tables stop
ip6tables: Flushing firewall rules:                        [  OK  ]
ip6tables: Setting chains to policy ACCEPT: filter         [  OK  ]
ip6tables: Unloading modules:                              [  OK  ]
</pre>
    * Ensure the system's node name is a resovable hostname. e.g.:
<pre>
[anthony@centos63-toolchain-example5 ~]$ uname -n 
centos63-toolchain-example5
[anthony@centos63-toolchain-example5 ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 centos63-toolchain-example5
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
</pre>

    * Disable SELinux and reboot:
<pre>
[chuck@localhost ~]$ sudo vi /etc/sysconfig/selinux 
[chuck@localhost ~]$ grep ^SELINUX= /etc/sysconfig/selinux 
SELINUX=disabled
[chuck@localhost ~]$ sudo reboot
</pre>

* User requirements:
    * Non-root user account ...
    * ... with sudo access to run any command as root without a password (e.g. wheel group membership)
    * ... sudo requiretty disabled to run any command without an interactive shell session (e.g. "Defaults requiretty" commented out)

* Repositories:
   * Configure the [EPEL repository](http://dl.fedoraproject.org/pub/epel/6/x86_64/repoview/epel-release.html). e.g:
<pre>
[anthony@centos63-toolchain-example5 ~]$ sudo rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-7.noarch.rpm
Retrieving http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-7.noarch.rpm
warning: /var/tmp/rpm-tmp.4fSCLZ: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Preparing...                ########################################### [100%]
   1:epel-release           ########################################### [100%]
</pre>

* Git setup:
    * Install git:
<pre>
[anthony@centos63-toolchain-example5 ~]$ sudo yum -y install git 
.
.
.
Complete!
[anthony@centos63-toolchain-example5 ~]$ git --version
git version 1.7.1
</pre>

* Obtain latest version of the rerun-modules repo definition from [Rerun Modules Downloads](https://github.com/rerun-modules/rerun-modules/downloads), obtain the link and execute the following replacing example url shown inline:
<pre>
[chuck@sdp-centos-63-64-1 toolchain-example5]$  sudo rpm -Uvh https://github.com/downloads/rerun-modules/rerun-modules/rerun-modules-repo-1.0-21.noarch.rpm
Retrieving https://github.com/downloads/rerun-modules/rerun-modules/rerun-modules-repo-1.0-21.noarch.rpm
Preparing...                ########################################### [100%]
   1:rerun-modules-repo     ########################################### [100%]
</pre>

* Obtain latest version of Rerun from [Rerun Downloads](https://github.com/rerun/rerun/downloads), obtain the link and execute the following replacing example url shown inline:
<pre>
[chuck@mvn-sdp-0 development]$ sudo rpm -Uvh https://github.com/downloads/rerun/rerun/rerun-1.0-109.noarch.rpm
Retrieving https://github.com/downloads/rerun/rerun/rerun-1.0-109.noarch.rpm
Preparing...                ########################################### [100%]
   1:rerun                  ########################################### [100%]
</pre>

* Install rpm-build, mysql, and dependent Rerun modules
<pre>
[chuck@sdp-centos-63-64-1 toolchain-example5]$ sudo yum -y install rpm-build mysql  rerun-mysql rerun-jenkins rerun-rundeck rerun-jboss-as
.
.
.
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
Updating   : mysql-libs-5.1.66-1.el6_3.x86_64                                                                                                                                                 1/3 
Installing : mysql-5.1.66-1.el6_3.x86_64                                                                                                                                                      2/3 
Cleanup    : mysql-libs-5.1.61-4.el6.x86_64                                                                                                                                                   3/3 
Verifying  : mysql-5.1.66-1.el6_3.x86_64                                                                                                                                                      1/3 
Verifying  : mysql-libs-5.1.66-1.el6_3.x86_64                                                                                                                                                 2/3 
Verifying  : mysql-libs-5.1.61-4.el6.x86_64                                                                                                                                                   3/3 
Installed:
 mysql.x86_64 0:5.1.66-1.el6_3                                                                                                                                                                     
Dependency Updated:
 mysql-libs.x86_64 0:5.1.66-1.el6_3                                                                                                                                                                
Complete!
</pre>

* Clone the [Toolchain Example #5](https://github.com/dtolabs/toolchain-example5) repository:
<pre>
[anthony@centos63-toolchain-example5 ~]$ mkdir src
[anthony@centos63-toolchain-example5 ~]$ cd src
[anthony@centos63-toolchain-example5 src]$ git clone git@github.com:dtolabs/toolchain-example5.git
Initialized empty Git repository in /home/anthony/src/toolchain-example5/.git/
remote: Counting objects: 90, done.
remote: Compressing objects: 100% (38/38), done.
remote: Total 90 (delta 22), reused 87 (delta 19)
Receiving objects: 100% (90/90), 4.04 MiB | 1.48 MiB/s, done.
Resolving deltas: 100% (22/22), done.
</pre>

* Deploy the toolchain build console:
<pre>
[chuck@mvn-sdp-0 toolchain-example5]$  rerun -M . toolchain-build-console: deploy
Shutting down Jenkins                                      [  OK  ]
Failed to set locale, defaulting to C
Failed to set locale, defaulting to C
Package rerun-rpm-1.0.0-25.noarch already installed and latest version
Failed to set locale, defaulting to C
Package rerun-apache-maven-1.0-8.noarch already installed and latest version
Failed to set locale, defaulting to C
Failed to set locale, defaulting to C
Package rpm-build-4.8.0-27.el6.x86_64 already installed and latest version
Failed to set locale, defaulting to C
Package 1:java-1.6.0-openjdk-devel-1.6.0.0-1.50.1.11.5.el6_3.x86_64 already installed and latest version
Failed to set locale, defaulting to C
Package matching xmlstarlet-1.3.1-1.el6.x86_64 already installed. Checking for update.
Starting Jenkins                                           [  OK  ]
reloading http://localhost:8080
Shutting down Jenkins                                      [  OK  ]
Starting Jenkins                                           [  OK  ]
</pre>

![building-console-jobs](https://github.com/dtolabs/toolchain-example5/raw/master/doc/build-console-jobs.jpg)
![building-console-jobs](https://github.com/dtolabs/toolchain-example5/raw/master/doc/build-console-jobs-built.jpg)

* Deploy the toolchain deploy console:
<pre>
[chuck@mvn-sdp-0 toolchain-example5]$ rerun -M . toolchain-deploy-console: deploy
Stopping rundeckd:                                         [  OK  ]
Failed to set locale, defaulting to C
Failed to set locale, defaulting to C
Package rerun-jboss-as-1.0-17.noarch already installed and latest version
Failed to set locale, defaulting to C
Package 1:java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.x86_64 already installed and latest version
Failed to set locale, defaulting to C
Package matching xmlstarlet-1.3.1-1.el6.x86_64 already installed. Checking for update.
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror, refresh-packagekit, security
Setting up Local Package Process
latest.rpm                                                                                                                                                                   | 2.1 kB     00:00     
Examining /var/tmp/yum-root-RQyb9A/latest.rpm: rundeck-repo-2-0.noarch
/var/tmp/yum-root-RQyb9A/latest.rpm: does not update installed package.
Nothing to do
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
 base: mirror.sanctuaryhost.com
 epel: linux.mirrors.es.net
 extras: holmes.umflint.edu
 updates: centos.mirror.sea.rackd.net
Setting up Install Process
Package rundeck-1.4.4-1.3.noarch already installed and latest version
Nothing to do
Starting rundeckd:                                         [  OK  ]
nohup: redirecting stderr to stdout
</pre>

* TODO: display deploy console here

* Append Rundeck Public SSH Key to your own authorized_keys file
<pre>
[chuck@sdp-centos-63-64-1 ]$ sudo cat /var/lib/rundeck/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
[chuck@mvn-sdp-0 toolchain-example5]$ sudo su - rundeck
[rundeck@mvn-sdp-0 ~]$ set -o vi
[rundeck@mvn-sdp-0 ~]$ ssh chuck@localhost id
uid=500(chuck) gid=500(chuck) groups=500(chuck),10(wheel),503(jboss-as)
[rundeck@mvn-sdp-0 ~]$ exit
[chuck@mvn-sdp-0 toolchain-example5]$ 
</pre>

* Run the deploy-seam-booking deploy job... TODO: display dtolabs-toolchain-example5-deploy-seam-booking here


