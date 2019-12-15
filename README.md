
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
1. Apply label for each nodes and remove taint from `master` so it can be scheduled to receive deployment
   ```bash
   kubectl label nodes master.vagrant.vm serve=instance-a
   kubectl label nodes worker1.vagrant.vm serve=instance-b
   kubectl taint nodes --all node-role.kubernetes.io/master-
   ```
2. SSH to `master` node, then install `skaffold`, refer to this link for detail steps (https://skaffold.dev/docs/install/)

## Deployment
1. SSH to kubernetes master, and clone this repo to your favorite folder
   ```bash
   user@laptop:~/vagrant-boxes/Kubernetes$ vagrant ssh master
   [vagrant@master ~]$ git clone https://github.com/h4n0l1/elastic-s3.git target_folder
   ```
2. Made changes first to these files, by changing image name prefix to use your docker hub account instead of mine, from **dtriana** to **YOUR_DOCKER_HUB_ACCOUNT**
   ```html
   ./kubernetes/es-rc.yaml        
   ./kubernetes/nginx.yaml        
   ./skaffold.yaml  
   ```
2. Start to build images and deploy them in cluster with skaffold
   ```bash
   [vagrant@master ~]$ cd target_folder
   [vagrant@master ~/target_folder]$ skaffold run
   ```
3. Confirm deployed pods with `kubectl`
   ```bash
   [vagrant@master ~/target_folder]$ kubectl get  pods --output=wide
   NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE                 NOMINATED NODE
   es-k2dnz                 1/1     Running   0          32s   10.244.1.47   worker1.vagrant.vm   <none>
   nginx-6f48695fbd-2k2nw   1/1     Running   0          32s   10.244.0.12   master.vagrant.vm    <none>
   ```
4. Put this line to `/etc/hosts`
   ```bash
   127.0.0.1	instance-a search
   ```
5. Elasticsearch can be accessed from it pods at port 9200, since reverse-proxy already setup, so we can access it through `nginx` service through port 30881
   ```bash
   [vagrant@master ~/target_folder]$ curl http://search:30881
   {
      "name" : "SxPwUFF",
      "cluster_name" : "docker-cluster",
      "cluster_uuid" : "3VNMYpg2QDWImpF0Wg2jcw",
      "version" : {
        "number" : "6.8.5",
        "build_flavor" : "default",
        "build_type" : "docker",
        "build_hash" : "78990e9",
        "build_date" : "2019-11-13T20:04:24.100411Z",
        "build_snapshot" : false,
        "lucene_version" : "7.7.2",
        "minimum_wire_compatibility_version" : "5.6.0",
        "minimum_index_compatibility_version" : "5.0.0"
      },
      "tagline" : "You Know, for Search"
   }
   ```

## Automation Pipeline
1. Before we start, remove first current deployment with this command:
   ```bash
   [vagrant@master ~/target_folder]$ skaffold delete
   ```
2. Run the automation pipeline with this command:
   ```bash
   [vagrant@master ~/target_folder]$ skaffold dev --no-prune=true
   Listing files to watch...
   - dtriana/nginx
   - dtriana/es3
   Generating tags...
   .
   .
   .
   .
     - deployment.apps/nginx created
     - service/nginx created
   Watching for changes...
   ```
3. Now, every changes made in context folders (`./servers/elastic-s3` and/or `/servers/nginx`), will trigger build process, push new image(s) and deploy it.
