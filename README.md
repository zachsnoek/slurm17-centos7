# slurm-centos7
Installation for the Slurm job scheduler on a CentOS 7 cluster.</br>

## Prerequisites
* [btools](https://github.com/zachsnoek/btools)

## Conventions
* The head node is identified as `headnode`
* The compute nodes are identified as `node01`, `node02`, and `node03`</br>

## Installation
0. On each node, confirm the Slurm and Munge UIDs and GIDs set in `install-slurm`. They must be the same on all of the nodes; modify the values if they are not available.</br></br>

1. On each node, run the installation script (this takes 20-30 minutes to complete)</br></br>
`$ ./install-slurm`</br></br>    

2. On the head node, create a pseudorandom secret key for MUNGE to use on all of the compute nodes</br></br>
`$ ./create-munge-key`</br></br>

3. Copy the MUNGE secret key to all of the compute nodes</br></br>
`$ bpush /etc/munge/munge.key /etc/munge/munge.key`</br></br>

4. Change the owner of `/etc/munge/munge.key` to the munge user on all of the nodes</br></br>
`$ chown munge: /etc/munge/munge.key`</br>
`$ bexec chown munge: /etc/munge/munge.key`</br></br>

5. Enable and start the MUNGE service on all of the nodes</br></br>
`$ systemctl enable munge`</br>
`$ systemctl start munge`</br>
`$ bexec systemctl enable munge`</br>
`$ bexec systemctl start munge`</br></br>

6. From any computer, complete the Slurm configuration file [generator](https://slurm.schedmd.com/configurator.easy.html); edit the fields according to the values below (fields not addressed below should be left as their default value or empty if there is no default value)</br>
     - ControlMachine: `headnode`
     - NodeNames: `node[01-03]`
     - CPUs, Sockets, CoresPerSocket, and ThreadsPerCore: Values can be found by listing the CPU information on your machine with the `lscpu` command</br>
     - StateSaveLocation: `/var/spool/slurm`
     - SlurmctldLogFile: `/var/log/slurm/slurmctld.log`
     - SlurmdLogFile: `/var/log/slurm/slurmd.log`</br></br>

7. Click `submit` at the bottom of the page to generate the configuration file</br></br>

8. Copy the configuration file to the head node and save the file to `/etc/slurm/slurm.conf`</br></br>

9. Copy the configuration file to all of the compute nodes</br></br>
`$ bpush /etc/slurm/slurm.conf /etc/slurm/slurm.conf`</br></br>

10. Move the cgroup configuration file to `/etc/slurm/cgroup.conf` (overwrite the existing file created with the install script)</br></br>
`$ mv files/cgroup.conf /etc/slurm/cgroup.conf`</br></br>

11. Copy the cgroup configuration file to all of the compute nodes</br></br>
`$ bpush /etc/slurm/cgroup.conf /etc/slurm/cgroup.conf`</br></br>

12. Disable and stop the firewalld service on all of the compute nodes</br></br>
`$ bexec systemctl disable firewalld`</br>
`$ bexec systemctl stop firewalld`</br></br>

13. On the head node, open port 6817 for Slurm</br></br>
`$ firewall-cmd --permanent --zone=public --add-port=6817/tcp`</br>
`$ firewall-cmd --reload`</br></br>

14. On the head node, enable and start the slurmctld service</br></br>
`$ systemctl enable slurmctld`</br>
`$ systemctl start slurmctld`</br></br>

15. On all of the compute nodes, enable and start the slurmd service</br></br>
`$ systemctl enable slurmd`</br>
`$ systemctl start slurmd`</br></br>

## Testing
Run the tests in `slurm-centos7/tests`</br></br>
`$ sbatch <file>`</br></br>

Check the Slurm configuration on all of the compute nodes</br></br>
`$ slurmd -C`</br></br>

Confirm that all of the compute nodes are reporting to the head node</br></br>
`$ scontrol show nodes`</br></br>

Run an interactive job</br></br>
`$ srun --pty bash`</br>
