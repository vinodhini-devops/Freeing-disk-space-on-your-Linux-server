# Freeing-disk-space-on-your-Linux-server
The following are quick commands to clear disk space on CentOS 6 or CentOS 7 servers.

&tldr;

curl -Ls http://bit.ly/clean-centos-disk-space | sudo bash

Scared to run that? Just run individual commands that the script runs.

Before anything, you have to install yum-utils package:

yum -y install yum-utils


1. Trim log files

find /var -name "*.log" \( \( -size +50M -mtime +7 \) -o -mtime +30 \) -exec truncate {} --size 0 \;
This will truncate any *.log files on the volume /var that are either older than 7 days and greater than 50M or older than 30 days.

2. Cleanup YUM cache
The simple command to cleanup yum caches:

yum clean all

Note that the above command will not remove everything related to yum. For instance, metadata for disabled repositories will not be affected.

You may want to free up space taken by orphaned data from disabled or removed repositories:

rm -rf /var/cache/yum

Also, when you accidentally run yum through a regular user (forgot sudo), yum will create user-cache. So let’s delete that too:

rm -rf /var/tmp/yum-*


3. Remove orphan packages
Check existing orphan packages

package-cleanup --quiet --leaves 

Confirm removing orphan packages
Now, if happy with suggestions given by the previous command, run:

package-cleanup --quiet --leaves | xargs yum remove -y

4. Remove WP CLI cached WordPress downloads
WP CLI saves WordPress archives every time you setup a new WordPress website. You can remove those caches by the following command:

rm -rf /root/.wp-cli/cache/*
rm -rf /home/*/.wp-cli/cache/*


5. Remove old kernels
Before removing old kernels, you might want to simply reboot first in order to boot up from the latest kernel.
That’s because you can’t remove an old kernel if you’re booted into it 🙂

The following commands will keep just 2 latest kernels installed:

(( $(rpm -E %{rhel}) >= 8 )) && dnf remove $(dnf repoquery --installonly --latest-limit=-2 -q)
(( $(rpm -E %{rhel}) <= 7 )) && package-cleanup --oldkernels --count=2

Note that with some VPS providers (Linode for example), servers use provider’s built kernels by default and not the ones on the server itself. So it makes little sense to keep more than 1 old kernel on the system. So:

(( $(rpm -E %{rhel}) >= 8 )) && dnf remove $(dnf repoquery --installonly --latest-limit=-1 -q)
(( $(rpm -E %{rhel}) <= 7 )) && package-cleanup --oldkernels --count=1


6. Remove Composer cache
rm -rf /root/.composer/cache
rm -rf /home/*/.composer/cache


7. Remove core dumps
If you had some severe failures with PHP which caused it to segfault and had core dumps enabled, chances are – you have quite a few of those.
They are not needed after you done debugging the problem. So:

find -regex ".*/core\.[0-9]+$" -delete


8. Remove error_log files (cPanel)
If you use the disgusting cPanel, you surely got dozens of error_log files scattered across your web directories. Much better if you can install the Citrus Stack. A temporary solution is to remove all those files:

find /home/*/public_html/ -name error_log -delete


9. Remove Node.js caches
rm -rf /root/.npm /home/*/.npm /root/.node-gyp /home/*/.node-gyp /tmp/npm-*


10. Remove Mock caches
Been building some RPM packages with mock? Those root caches can be quite large.
If you no longer intend to build RPM packages on a given machine:

rm -rf /var/cache/mock/* /var/lib/mock/*
P.S. the plan is to make this into an easily-installable app.
