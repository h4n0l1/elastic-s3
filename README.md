
# elastic-s3
## Requirement
1. Account at oracle.com, docker.com
2. Vagrant (for Ubuntu 18.04 you can use version 2.2.6)
   - vagrant-env plugin, install it by executing command
    ```bash
     vagrant plugin install vagrant-env
     ```
3. VirtualBox version 5.2

## Setup Environtment
1. Clone repository from https://github.com/oracle/vagrant-boxes
    ```bash 
     user@laptop:~$ git clone https://github.com/oracle/vagrant-boxes
     ```
2. Go to folder Kubernetes inside that repository
    ```bash
    user@laptop:~$ cd vagrant-boxes/Kubernetes
    ```
4. Copy .env file to .env.local
   ```bash
   user@laptop:~/vagrant-boxes/Kubernetes$ cp .env .env.local
   ```
5. Adjust some variables inside file .env.local as needed by uncommenting it, for example, in this exercise we will need only **1 worker** and use **4GB** memory for each virtualbox instances. Just made change to this lines:
   ```bash
   NB_WORKERS=1
   MEMORY=4096
   ```
6. Save it, then continue to install all virtualbox instances by executing this command
    ```bash
    user@laptop:~/vagrant-boxes/Kubernetes$ vagrant up
    ```
7. After finish, make sure all instances running by executing command: vagrant status
   Then you will have this output:
   ```bash
   Current machine states:
   master                    running (virtualbox)
   worker1                   running (virtualbox)
   ```
8. Now kubernetes already available in those instances, continue kubernetes cluster setup by executing script kubeadm-setup
   > **[for master node]** 
   ```bash
   user@laptop:~/vagrant-boxes/Kubernetes$ vagrant ssh master
   [vagrant@master ~]$ sudo /vagrant/scripts/kubeadm-setup-master.sh  ##you'll be asked for your credential at oracle.com
   [vagrant@master ~]$ sudo docker logout
   [vagrant@master ~]$ docker login   ##you'll be asked for your credential at docker.com
   ```

   > **[for worker node]** 
   ```bash
   user@laptop:~/vagrant-boxes/Kubernetes$ vagrant ssh worker1
   [vagrant@worker1 ~]$ sudo /vagrant/scripts/kubeadm-setup-worker.sh   ##you'll be asked for your credential at oracle.com
   [vagrant@worker1 ~]$ sudo docker logout
   [vagrant@worker1 ~]$ docker login   ##you'll be asked for your credential at docker.com
   ```

9. Verify your kubernetes cluster by executing this command:
   ```bash
   user@laptop:~/vagrant-boxes/Kubernetes$ vagrant ssh master
   [vagrant@master ~]$ kubectl get nodes
   NAME                 STATUS   ROLES    AGE   VERSION
   master.vagrant.vm    Ready    master   39m   v1.12.7+1.2.3.el7
   worker1.vagrant.vm   Ready    <none>   37m   v1.12.7+1.2.3.el7
   ```

## Pre Deployment
1. ```bash
   kubectl label nodes master.vagrant.vm serve=instance-a
   kubectl label nodes worker1.vagrant.vm serve=instance-b
   kubectl taint nodes --all node-role.kubernetes.io/master-
   ```
2. SSH to master node, then install skaffold, refer to this link for detail steps (https://skaffold.dev/docs/install/)

