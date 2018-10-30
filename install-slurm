#!/bin/bash

export VER=17.11.7

# Create global user accounts on all of the nodes in the cluster. These must be created identically on all of the nodes and
# done prior to installing the RPMs
groupadd -K GID_MIN=500 -K GID_MAX=999 munge
useradd -K UID_MIN=500 -K UID_MAX=999 -m -c "MUNGE authentication service" -d /var/lib/munge -g munge  -s /sbin/nologin munge
MUNGEUSER=$(id -u munge)

groupadd -K GID_MIN=500 -K GID_MAX=999 slurm
useradd -K UID_MIN=500 -K UID_MAX=999 -m -c "Slurm workload manager" -d /var/lib/slurm  -g slurm  -s /bin/bash slurm
SLURMUSER=$(id -u slurm)

# In case Slurm already exists but with the wrong ids:
usermod -o -u $SLURMUSER slurm
groupmod -o -g $SLURMUSER slurm

# Install epel-release on all of the nodes
yum install -y epel-release

# Install MUNGE on all of the nodes 
yum install -y munge munge-libs munge-devel

# Set the permissions for MUNGE directories on all of the nodes
chown -R munge: /etc/munge/ /var/log/munge/
chmod 0700 /etc/munge/ /var/log/munge/

# Install the dependencies for Slurm
yum install -y rpm-build gcc openssl openssl-devel libssh2-devel pam-devel numactl numactl-devel 
yum install	-y hwloc hwloc-devel lua lua-devel readline-devel rrdtool-devel ncurses-devel gtk2-devel man2html 
yum install	-y libibmad libibumad perl-Switch perl-ExtUtils-MakeMaker
yum install -y mariadb-server mariadb-devel

# Download the Slurm 17.11.7 tarball
mv files/slurm-$VER.tar.bz2 /root
cd /root
#wget https://download.schedmd.com/slurm/slurm-17.11.7.tar.bz2

# Build the RPMs
rpmbuild -ta slurm-$VER.tar.bz2

# Install the RPMs on all of the nodes; RPMs on each node can vary by configuration, but
# this is my starting point
cd $HOME/rpmbuild/RPMS/x86_64
export VER=17.11.7-1

yum install -y slurm-$VER.el7.x86_64.rpm
yum install -y slurm-perlapi-$VER.el7.x86_64.rpm
yum install -y slurm-openlava-$VER.el7.x86_64.rpm
yum install -y slurm-contribs-$VER.el7.x86_64.rpm
yum install -y slurm-devel-$VER.el7.x86_64.rpm
yum install -y slurm-example-configs-$VER.el7.x86_64.rpm
yum install -y slurm-libpmi-$VER.el7.x86_64.rpm
yum install -y slurm-pam_slurm-$VER.el7.x86_64.rpm
yum install -y slurm-slurmctld-$VER.el7.x86_64.rpm
yum install -y slurm-slurmd-$VER.el7.x86_64.rpm
yum install -y slurm-torque-$VER.el7.x86_64.rpm 

# Create the files and directories for use by Slurm that are not created during install; set the 
# permissions for the files and directories
mkdir /var/spool/slurmctld /var/log/slurm
chown slurm: /var/spool/slurmctld /var/log/slurm
chmod 755 /var/spool/slurmctld /var/log/slurm

touch /var/log/slurm/slurmctld.log
chown slurm: /var/log/slurm/slurmctld.log

touch /var/log/slurm/slurm_jobacct.log /var/log/slurm/slurm_jobcomp.log
chown slurm: /var/log/slurm/slurm_jobacct.log /var/log/slurm/slurm_jobcomp.log

mkdir /var/spool/slurm
chown slurm: /var/spool/slurm

mkdir /var/spool/slurmd
chown slurm: /var/spool/slurmd  /var/log/slurm
chmod 755 /var/spool/slurmd  /var/log/slurm

touch /var/log/slurm/slurmd.log
chown slurm: /var/log/slurm/slurmd.log

chown slurm: /etc/slurm

touch /etc/slurm/cgroup.conf
