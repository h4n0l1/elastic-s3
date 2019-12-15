# elastic-s3
#Requirement
1. Account at oracle.com
2. Vagrant (for Ubuntu 18.04 you can use version 2.2.6)
   - vagrant-env plugin, install it by executing command
     vagrant plugin install vagrant-env
3. VirtualBox version 5.2

# Setup Environtment
1. Clone repository from https://github.com/oracle/vagrant-boxes
   user@laptop:~$ git clone https://github.com/oracle/vagrant-boxes
2. Go to folder Kubernetes inside that repository
   user@laptop:~$ cd vagrant-boxes/Kubernetes
3. Copy .env file to .env.local
   user@laptop:~/vagrant-boxes/Kubernetes$ cp .env .env.local
4. Adjust some variables inside file .env.local as needed by uncommenting it, for example, in this exercise we will need only 1 worker and use 4GB memory for each virtualbox instances. Just made change to this lines:
   NB_WORKERS=1
   MEMORY=4096
5. Save it, then continue to install all virtualbox instances by executing this command
   user@laptop:~/vagrant-boxes/Kubernetes$ vagrant up
6. After finish, make sure all instances running by executing command: vagrant status
   Then you will have this output:
   Current machine states:

   master                    running (virtualbox)
   worker1                   running (virtualbox)
7. Now kubernetes already available in those instances, continue kubernetes cluster setup by executing script kubeadm-setup
   [for master node] 
   user@laptop:~/vagrant-boxes/Kubernetes$ vagrant ssh master
   [vagrant@master ~]$ sudo /vagrant/scripts/kubeadm-setup-master.sh

   [for worker node] 
   user@laptop:~/vagrant-boxes/Kubernetes$ vagrant ssh worker1
   [vagrant@worker1 ~]$ sudo /vagrant/scripts/kubeadm-setup-worker.sh

   You will be asked to put your account credential at oracle.com
8. Verify your kubernetes cluster by executing this command:
   user@laptop:~/vagrant-boxes/Kubernetes$ vagrant ssh master
   [vagrant@master ~]$ kubectl get nodes
   NAME                 STATUS   ROLES    AGE   VERSION
   master.vagrant.vm    Ready    master   39m   v1.12.7+1.2.3.el7
   worker1.vagrant.vm   Ready    <none>   37m   v1.12.7+1.2.3.el7

docker logout container-registry.oracle.com/kubernetes
docker login

