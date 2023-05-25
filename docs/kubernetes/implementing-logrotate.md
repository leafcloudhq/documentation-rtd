# Implementing logrotate

This tutorial will walk you through the steps to implement logrotate on your Magnum Kubernetes cluster. OpenStack Magnum uses Podman to provision core containers for a Kubernetes cluster (for example: API, schedule, etcd, Kublet, and manager) and runs all of those containers as system services. Kubernetes is reliable because it provides repetitive components and ensures the application containers can be scheduled across multiple nodes, monitoring them and maintaining the cluster. All of this creates API calls, which are stored in the log file.

Unfortunately, the logs aren't deleted by default. So, after running a Magnum Kubernetes cluster for a while, the logs will pile up and eventually lead to full disks, lots of evicted pods, performance issues, or even your application not being available anymore.

To prevent this, we will need to implement a ‘logrotate.’ This log rotation will remove the old logs after a set amount of time, thus avoiding full disks and performance issues. Soon we will implement this function into the existing Kubernetes template, but for now, you can follow these steps to add log rotation to your Magnum Kubernetes cluster:

## Create a logrotated file called podman

    $ vi /etc/logrotate.d/podman
    /sysroot/ostree/deploy/fedora-coreos/var/lib/containers/storage/overlay-containers/*/userdata/ctr.log {
      rotate 5
      daily
      size 1M
      copytruncate
      prerotate
        if [ $(mount | grep '/sysroot' | awk '{print $NF}' | grep 'ro,' &> /dev/null; echo $?) -eq 0 ]; then \
          /usr/bin/mount -o remount,rw /sysroot; \
        fi; \
      endscript
      maxage 15
      compress
    }

## Test the lograte
Next, test what this logrotate will do in debug mode, without affecting the current

    $ logrotate -d /etc/logrotate.d/podman

## Disable SELinux

    $ setenforce 0

You can now run this log rotate manually out of scheduled dates, but make sure that you don't have a big log files, and if you do please truncate them.

    $ find /sysroot/ostree/deploy/ -name ctr.log* | xargs ls -lh | sort -k4 -hr
    $ truncate -s 50M ${file_path}     # if u have big file and need to truncate before run logrotate manually
    $ logrotate -dv /etc/logrotate.d/podman

## Starting podman containers
If you have any failed podman containers, start them with systemctl

    $ podman ps        # use name for failed service (ex: kube-apiserver)
    $ systemctl start kube-apiserve

## Permenet disable SELinux
If we reboot node selinux will be enabled again, we need to disable it

    $ sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

*Expert Note:* The logrotate service is triggered daily by logrotate.timer, so there’s no need to restart it.

Hopefully this fix will help you deal with rogue pods and log pile-ups.