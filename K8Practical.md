Kubernetes Practical
====================

This practical provides step-by-step guide to create a Web front-end in Kubernetes. It is designed to help a developer new to Kubernetes to create a work environment from scratch. A virtual machine running Ubuntu 18.04 LTS is assumed. The following exercises are built on top of each other:

#. `Reading 0: Adding Minikube to the new VMs`_
#. `Exercise 0.0: Starting Minikube`_
#. `Exercise 0.1: (Optional) Enabling GUI for VNC`_
#. `Exercise 1: Creating NGINX`_
#. `Exercise 2: Adding HTML to pods`_
#. `Exercise 3: Persistence and disaster recovery`_
#. `Exercise 4: Problem determination step-by-step`_
#. `Homework: Additional exercises after the workshop`_

This practical intends to use a simple example to demonstrate the features and characteristics of Kubernetes as an orchestrator for container-based workloads. You should be able to create your own work environment and to start implementing your own project after finishing this practical.

The code for the practical is under https://gitlab.ebi.ac.uk/TSI/tsi-ccdoc/tree/master/tsi-cc/ResOps/scripts. The deployment would look like the following once it is completed:

.. image:: /static/images/resops2019/K8SPractical.png

Reading 0: Adding Minikube to the new VMs
-----------------------------------------

This original exercise was designed to show user how to build a sandbox for an individual developer. This is automated to give users more time to focus on cloud-specific subjects. Read through this section so that you can build your own sandbox after the workshop.

Kubernetes has comprehensive documentation on if and how to use Minikube https://kubernetes.io/docs/setup/minikube/.

Access the VM assigned to you from your SSH client. The command is `ssh <user_id>@<IP_of_VM_assigned_to_you>`. Your user ID, password and IP address will be given to you in the workshop.

.. Access the VM assigned to you from your SSH client. The command is `ssh -i <full_path_to_private_key> ubuntu@<IP_of_VM_assigned_to_you>`, for example `ssh -i ~/.ssh/id_rsa ubuntu@193.62.55.21`. Your public key has been added to the assigned VM by one of us.

.. Access the virtual workstation via bastion server, for example `ssh -i ~/.ssh/id_rsa -o UserKnownHostsFile=/dev/null -o ProxyCommand="ssh -W %h:%p -i ~/.ssh/id_rsa ubuntu@193.62.55.21" ubuntu@10.0.0.5`

Install Docker as the runtime for Minikube::

  sudo apt-get update
  sudo apt install -y docker.io

Install kubectl as the client to access the Kubernetes cluster in Minikube::

  sudo snap install kubectl --classic

Download, install and configure Minikube::

  curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
  sudo cp minikube /usr/local/bin && rm minikube

Note that /home/ubuntu/.kube/config is owned by root. Run `chown` to avoid using `sudo`::

  sudo chown -R $USER $HOME/.kube $HOME/.minikube

If a sudor other than ubuntu is used, move the following two directories::

  sudo mv /home/ubuntu/.kube /home/ubuntu/.minikube $HOME

Exercise 0.0: Starting Minikube
-------------------------------

Start Minikube with Docker::

  minikube start --vm-driver='docker'

The following message should be displayed when Minikube is started successfully::

  ðŸ˜„  minikube v1.17.1 on Ubuntu 18.04 (amd64)
  âœ¨  Using the docker driver based on user configuration
  ðŸ‘  Starting control plane node minikube in cluster minikube
  ðŸšœ  Pulling base image ...
  ðŸ’¾  Downloading Kubernetes v1.20.2 preload ...
      > preloaded-images-k8s-v8-v1....: 491.22 MiB / 491.22 MiB  100.00% 18.07 Mi
  ðŸ”¥  Creating docker container (CPUs=2, Memory=2200MB) ...
  â—  This container is having trouble accessing https://k8s.gcr.io
  ðŸ’¡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
  ðŸ³  Preparing Kubernetes v1.20.2 on Docker 20.10.2 ...
      â–ª Generating certificates and keys ...
      â–ª Booting up control plane ...
      â–ª Configuring RBAC rules ...
  ðŸ”Ž  Verifying Kubernetes components...
  ðŸŒŸ  Enabled addons: storage-provisioner, default-storageclass
  ðŸ„  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

Enable addons such as metrics-server and ingress by running the following::

  minikube addons enable metrics-server
  minikube addons enable ingress

Verify the right addons are enabled::

  minikube addons list

  |-----------------------------|----------|--------------|
  |         ADDON NAME          | PROFILE  |    STATUS    |
  |-----------------------------|----------|--------------|
  | ambassador                  | minikube | disabled     |
  | csi-hostpath-driver         | minikube | disabled     |
  | dashboard                   | minikube | disabled     |
  | default-storageclass        | minikube | enabled âœ…   |
  | efk                         | minikube | disabled     |
  | freshpod                    | minikube | disabled     |
  | gcp-auth                    | minikube | disabled     |
  | gvisor                      | minikube | disabled     |
  | helm-tiller                 | minikube | disabled     |
  | ingress                     | minikube | enabled âœ…   |
  | ingress-dns                 | minikube | disabled     |
  | istio                       | minikube | disabled     |
  | istio-provisioner           | minikube | disabled     |
  | kubevirt                    | minikube | disabled     |
  | logviewer                   | minikube | disabled     |
  | metallb                     | minikube | disabled     |
  | metrics-server              | minikube | enabled âœ…   |
  | nvidia-driver-installer     | minikube | disabled     |
  | nvidia-gpu-device-plugin    | minikube | disabled     |
  | olm                         | minikube | disabled     |
  | pod-security-policy         | minikube | disabled     |
  | registry                    | minikube | disabled     |
  | registry-aliases            | minikube | disabled     |
  | registry-creds              | minikube | disabled     |
  | storage-provisioner         | minikube | enabled âœ…   |
  | storage-provisioner-gluster | minikube | disabled     |
  | volumesnapshots             | minikube | disabled     |
  |-----------------------------|----------|--------------|

Verify that Minicube is working and you can access it. You should see the following message::

  kubectl get node

  NAME       STATUS   ROLES                  AGE   VERSION
  minikube   Ready    control-plane,master   12m   v1.20.2

Now, you have a Kubernetes environment to develop and to test your workload. Never use it for production though.

Exercise 0.1: (Optional) Enabling GUI for VNC
---------------------------------------------

The xfce4 is a light weight GUI desktop. We use it together with VNC so that a web browser can be used on the VM. If this section is skipped, use `curl` for the rest of exercises.

Install xfce4 with the following command::

  sudo apt install -y xfce4 xfce4-goodies

Answer `Yes` to the question "Restart services during package upgrades without asking?"

Start VNC server with the preferred geometry and color depth, for example::

  vnc4server :1 -geometry 1600x900 -depth 24

Start another SSH session to enable SSH port forward with **your own** user ID and IP address, for example::

  ssh resops6@45.86.170.124 -C -L 5901:127.0.0.1:5901

Start your favourite VNC client, for example `Screen Sharing` on Mac OS. Connect to the VNC server as ``localhost:5901`` and with your logon password. Make sure to select "Use default configuration" when the desk initialises for the first time.

Exercise 1: Creating NGINX
--------------------------

Create a Kubernetes manifest file with the following command::

  nano ~/nginx.yml

You can copy and paste the manifest from https://gitlab.ebi.ac.uk/TSI/tsi-ccdoc/raw/master/tsi-cc/ResOps/scripts/minikube/nginx.yml to the editor. Review the file carefully to see what resources are to be created and how parts are connected with each other.

.. image:: /static/images/resops2019/nginx.yml.png

This manifest creates a StatefulSet with a corresponding service, where NGINX will be listening on port 80 for HTTP. There are two pods to be created in the set for load balancing, minimising downtime and to certain extend disaster recovering. The container NGINX is pulled from Docker Hub. It will be started to serve HTML pages from persistent volumes private to each pod.

Apply the manifest to create a service and statefulset::

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl apply -f nginx.yml
  service/nginx created
  statefulset.apps/web created

This creates additional resources needed to for a cluster of NGINX servers(persistent volume claims, persistent volumes, pods, and services)::

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl get pvc
  NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  www-web-0   Bound    pvc-4cd5c31b-7718-11e9-8b0b-fa163ede6c1a   1Gi        RWO            standard       39s
  www-web-1   Bound    pvc-5ac9090b-7718-11e9-8b0b-fa163ede6c1a   1Gi        RWO            standard       16s

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl get pv
  NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
  pvc-4cd5c31b-7718-11e9-8b0b-fa163ede6c1a   1Gi        RWO            Delete           Bound    default/www-web-0   standard                41s
  pvc-5ac9090b-7718-11e9-8b0b-fa163ede6c1a   1Gi        RWO            Delete           Bound    default/www-web-1   standard                18s

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl get pod
  NAME    READY   STATUS    RESTARTS   AGE
  web-0   1/1     Running   0          52s
  web-1   1/1     Running   0          29s

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl get svc
  NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
  kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   61m
  nginx        ClusterIP   10.104.206.229   <none>        80/TCP    76s

You now have a cluster of two NGINX pods running in Minicube. They are listening on port 80 with one cluster IP. You have just built a web service infrastructure with redundancy managed by Kubernetes.

Exercise 2: Adding HTML to pods
-------------------------------

**Note**: if you completed `Exercise 0.1: (Optional) Enabling GUI for VNC`_, you can use `Firefox` on your desktop instead of `curl` for all the exercises.

Each of the two NGINX server has its own storage, running in its own pod but share the same cluster IP, for example `10.104.206.229`. The cluster IP is listed with `kubectl get svc` in the previous exercise. If you try to access the home page, you will get HTTP403::

  ubuntu@resops-1-k8s-node-nf-1:~$ curl http://10.104.206.229
  <html>
  <head><title>403 Forbidden</title></head>
  <body>
  <center><h1>403 Forbidden</h1></center>
  <hr><center>nginx/1.15.12</center>
  </body>
  </html>

Let's fix this on one of the two NGINX servers. Recall that the volume `www` is mounted on `/usr/share/nginx/html/`, which is the default document root for NGINX server. Connect to pod web-0 and you can see that there is no file to be served by NGINX. That's why the HTTP403 is sent back::

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl exec -it web-0 -- /bin/bash
  root@web-0:/# cd /usr/share/nginx/html/
  root@web-0:/usr/share/nginx/html# ls -la
  total 8
  drwxrwxrwx 2 root root 4096 May 15 13:49 .
  drwxr-xr-x 3 root root 4096 May  8 03:01 ..

Create a simple `/usr/share/nginx/html/index.html` with a title and heading "Hello World from web-0"::

  cat <<EOF > /usr/share/nginx/html/index.html
  <html>
  <head><title>Hello World from web-0</title></head>
  <body>
  <center><h1>Hello World from web-0</h1></center>
  <hr><center>nginx/1.15.12</center>
  </body>
  </html>
  EOF

Exit out of the pod web-0::

  root@web-0:/usr/share/nginx/html# exit
  exit

Repeat the process in pod web-1. It is important to use a title and heading "Hello World from web-1" so that the two pods have two different index.html files::

  kubectl exec -it web-1 -- /bin/bash

  cat <<EOF > /usr/share/nginx/html/index.html
  <html>
  <head><title>Hello World from web-1</title></head>
  <body>
  <center><h1>Hello World from web-1</h1></center>
  <hr><center>nginx/1.15.12</center>
  </body>
  </html>
  EOF

Exit out of the pod web-1. Send HTTP request to the cluster IP again. You will find that the two NGINX servers take turns to serve the home page. The HTTP 403 error is gone::

  ubuntu@resops-1-k8s-node-nf-1:~$ curl http://10.104.206.229
  <html>
  <head><title>Hello World from web-1</title></head>
  <body>
  <center><h1>Hello World from web-1</h1></center>
  <hr><center>nginx/1.15.12</center>
  </body>
  </html>

  ubuntu@resops-1-k8s-node-nf-1:~$ curl http://10.104.206.229
  <html>
  <head><title>Hello World from web-0</title></head>
  <body>
  <center><h1>Hello World from web-0</h1></center>
  <hr><center>nginx/1.15.12</center>
  </body>
  </html>

Note that Kubernetes tend to route the requests to the same pod for better performance. You may keep seeing your HTML page served from the same pod, for example web-0. If this is happening, rename the index.html in web-0. Then you will see the page gets served from the other pod web-1::

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl exec -it web-0 -- /bin/bash
  root@web-0:/# mv /usr/share/nginx/html/index.html /usr/share/nginx/html/index.html.bak
  root@web-0:/# exit
  exit

  ubuntu@resops-1-k8s-node-nf-1:~$ curl http://10.104.206.229
  <html>
  <head><title>Hello World from web-1</title></head>
  <body>
  <center><h1>Hello World from web-1</h1></center>
  <hr><center>nginx/1.15.12</center>
  </body>
  </html>

Change the index.html page in web-0 back::

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl exec -it web-0 -- /bin/bash
  root@web-0:/# mv /usr/share/nginx/html/index.html.bak /usr/share/nginx/html/index.html
  root@web-0:/# exit
  exit

From exercises 2 & 3, we understand that the NGINX serves independent copies from `/usr/share/nginx/html` persisted on two separate volumes via two pods web-0 and web-1. See output by `kubectl get` commands in exercise 2 and `curl http://10.104.206.229` command in exercise 3 again.

Exercise 3: Persistence and disaster recovery
---------------------------------------------

If a stateful set is removed, the persistent volumes are preserved by design. Run `kubectl delete -f nginx.yml` to simulate scheduled outage. The service and pods are deleted but persistent volumes are saved::

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl delete -f nginx.yml
  service "nginx" deleted
  statefulset.apps "web" deleted

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl get pvc
  NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  www-web-0   Bound    pvc-4cd5c31b-7718-11e9-8b0b-fa163ede6c1a   1Gi        RWO            standard       110m
  www-web-1   Bound    pvc-5ac9090b-7718-11e9-8b0b-fa163ede6c1a   1Gi        RWO            standard       110m

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl get pv
  NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
  pvc-4cd5c31b-7718-11e9-8b0b-fa163ede6c1a   1Gi        RWO            Delete           Bound    default/www-web-0   standard                110m
  pvc-5ac9090b-7718-11e9-8b0b-fa163ede6c1a   1Gi        RWO            Delete           Bound    default/www-web-1   standard                110m

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl get svc
  NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   173m

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl get pod
  No resources found.

Run `kubectl apply -f nginx.yml` to recreate the stateful set and service. The each new pod recreated will mount its original volume as if it was never deleted. The pod web-0 still has the index.html with the message of "Hello World from web-0"::

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl apply -f nginx.yml
  service/nginx created
  statefulset.apps/web created

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl get pod
  NAME    READY   STATUS    RESTARTS   AGE
  web-0   1/1     Running   0          14s
  web-1   1/1     Running   0          7s

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl exec -it web-0 -- /bin/bash
  root@web-0:/# cat /usr/share/nginx/html/index.html
  <html>
  <head><title>Hello World from web-0</title></head>
  <body>
  <center><h1>Hello World from web-0</h1></center>
  <hr><center>nginx/1.15.12</center>
  </body>
  </html>

  root@web-0:/# exit
  exit

Run `kubectl delete pod web-1` to simulate unscheduled outage. The recovery happens really fast. We need to chain two kubectl command to see what is happening::

  kubectl delete pod web-1 && kubectl get pod

Kubernete tries to restart pod web-1 immediately. After a little while web-1 will be running again as if noting happened. It will be mounted to its original volume. The index.html in used by pod web-1 still has the same message of "Hello World from web-1"::

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl delete pod web-1 && kubectl get pod
  pod "web-1" deleted
  NAME    READY   STATUS              RESTARTS   AGE
  web-0   1/1     Running             0          6m50s
  web-1   0/1     ContainerCreating   0          0s

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl get pod
  NAME    READY   STATUS    RESTARTS   AGE
  web-0   1/1     Running   0          11m
  web-1   1/1     Running   0          4m27s

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl exec -it web-1 -- /bin/bash
  root@web-1:/# cat /usr/share/nginx/html/index.html
  <html>
  <head><title>Hello World from web-1</title></head>
  <body>
  <center><h1>Hello World from web-1</h1></center>
  <hr><center>nginx/1.15.12</center>
  </body>
  </html>

As you can see, Kubernetes restarts web-1 immediately. The newly started pod still mount to the same persistent volume as if the pod was never killed.

Exercise 4: Problem determination step-by-step
----------------------------------------------

Frequently, a manifest file that we have created does not behave as we expected or contains bugs. Here is a micky-mouse example how we can investigate what is going on.

First, repeat the first step in the previous exercise to remove the deployment::

  ubuntu@resops-1-k8s-node-nf-1:~$ kubectl delete -f nginx.yml
  service "nginx" deleted
  statefulset.apps "web" deleted

Run `kubernetes get` commands to confirm if the resources are deleted. Check the instructions in the previous exercise if you do not remember the details.

Second, inspect https://gitlab.ebi.ac.uk/TSI/tsi-ccdoc/blob/master/tsi-cc/ResOps/scripts/minikube/mickymouse.yml to see if you can spot any problems in the manifest.

Third, apply mickymouse.yml::

  resops49@resops-k8s-node-17:~$ kubectl apply -f https://gitlab.ebi.ac.uk/TSI/tsi-ccdoc/raw/master/tsi-cc/ResOps/scripts/minikube/mickymouse.yml
  service/nginx created
  statefulset.apps/web created

Cool! Or, is it? Let's repeat the `kubernetes get` commands to see if there is anything unusual. Here is what you would likely see::

  resops49@resops-k8s-node-17:~$ kubectl get pvc
  NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  www-web-0   Bound    pvc-4290e18c-d2b1-4714-b960-2f892121fd5b   1Gi        RWO            standard       6m
  www-web-1   Bound    pvc-dcbd9627-9b02-4d5a-8be1-c40045c0670c   1Gi        RWO            standard       5m48s

  resops49@resops-k8s-node-17:~$ kubectl get pv
  NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
  pvc-4290e18c-d2b1-4714-b960-2f892121fd5b   1Gi        RWO            Delete           Bound    default/www-web-0   standard                6m6s
  pvc-dcbd9627-9b02-4d5a-8be1-c40045c0670c   1Gi        RWO            Delete           Bound    default/www-web-1   standard                5m54s

  resops49@resops-k8s-node-17:~$ kubectl get pod
  No resources found.

  resops49@resops-k8s-node-17:~$ kubectl get svc
  NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
  kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        6d22h
  nginx        NodePort    10.107.235.150   <none>        80:31541/TCP   3m54s

We are expecting two pods running but nothing is created. PVs and PVCs are bounded. However, you would notice the mismatch of the timestamp between Service and PersistentVolumes. This is because, again, StatefulSet always leaves PersistentVolumes untouched by design even if the reclaim policy is `delete`. We need to delete these volumes explicitly to fully understand how naughty mickymouse.yml is::

  resops49@resops-k8s-node-17:~$ kubectl delete -f https://gitlab.ebi.ac.uk/TSI/tsi-ccdoc/raw/master/tsi-cc/ResOps/scripts/minikube/mickymouse.yml
  service "nginx" deleted
  statefulset.apps "web" deleted

  resops49@resops-k8s-node-17:~$ kubectl delete pvc www-web-0
  persistentvolumeclaim "www-web-0" deleted

  resops49@resops-k8s-node-17:~$ kubectl delete pvc www-web-1
  persistentvolumeclaim "www-web-1" deleted

  resops49@resops-k8s-node-17:~$ kubectl get pv
  No resources found.

  resops49@resops-k8s-node-17:~$ kubectl get svc
  NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6d23h

Because of the reclaim policy, deleting the PVCs cleans out PVs as well. Now the environment is clean. Let's apply mickymouse.yml, again::

  resops49@resops-k8s-node-17:~$ kubectl apply -f https://gitlab.ebi.ac.uk/TSI/tsi-ccdoc/raw/master/tsi-cc/ResOps/scripts/minikube/mickymouse.yml
  service/nginx created
  statefulset.apps/web created

  resops49@resops-k8s-node-17:~$ kubectl get pvc
  NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  www-web-0   Bound    pvc-9d3be2c4-3605-4f86-95d2-0e1f7328ded4   1Gi        RWO            standard       45s

  resops49@resops-k8s-node-17:~$ kubectl get pv
  NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
  pvc-9d3be2c4-3605-4f86-95d2-0e1f7328ded4   1Gi        RWO            Delete           Bound    default/www-web-0   standard                54s

  resops49@resops-k8s-node-17:~$ kubectl get pod
  No resources found.

  resops49@resops-k8s-node-17:~$ kubectl get svc
  NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
  kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        6d23h
  nginx        NodePort    10.105.177.74   <none>        80:30482/TCP   74s

The Service and StatefulSet seem created successfully again. The timestamps of Service, PV and PVC are close enough now. We still do not have any pods. In addition, we only see one pair of PV and PVC instead of two pairs. Things are going from bad to worse.

Fourth, check the events recorded by Kubernetes to see if there have been any failures. The event log can be long and messy. Always remember to sort the events by last seen or to apply various filters. See `kubectl get --help` to see how to apply filters::

  resops49@resops-k8s-node-17:~$ kubectl get event --sort-by='{.metadata.creationTimestamp}'
  LAST SEEN   TYPE      REASON                  OBJECT                            MESSAGE
  60m         Normal    Starting                node/minikube                     Starting kube-proxy.
  60m         Normal    RegisteredNode          node/minikube                     Node minikube event: Registered Node minikube in Controller
  46m         Warning   FailedScheduling        pod/web-0                         pod has unbound immediate PersistentVolumeClaims
  46m         Normal    ProvisioningSucceeded   persistentvolumeclaim/www-web-0   Successfully provisioned volume pvc-4290e18c-d2b1-4714-b960-2f892121fd5b
  46m         Normal    Provisioning            persistentvolumeclaim/www-web-0   External provisioner is provisioning volume for claim "default/www-web-0"
  46m         Normal    ExternalProvisioning    persistentvolumeclaim/www-web-0   waiting for a volume to be created, either by external provisioner "k8s.io/minikube-hostpath" or manually created by system administrator
  46m         Normal    SuccessfulCreate        statefulset/web                   create Pod web-0 in StatefulSet web successful
  46m         Normal    SuccessfulCreate        statefulset/web                   create Claim www-web-0 Pod web-0 in StatefulSet web success
  46m         Normal    Scheduled               pod/web-0                         Successfully assigned default/web-0 to minikube
  46m         Normal    Pulling                 pod/web-0                         Pulling image "nginx"
  46m         Normal    Pulled                  pod/web-0                         Successfully pulled image "nginx"
  46m         Normal    SuccessfulCreate        statefulset/web                   create Claim www-web-1 Pod web-1 in StatefulSet web success
  46m         Normal    ExternalProvisioning    persistentvolumeclaim/www-web-1   waiting for a volume to be created, either by external provisioner "k8s.io/minikube-hostpath" or manually created by system administrator
  46m         Normal    Provisioning            persistentvolumeclaim/www-web-1   External provisioner is provisioning volume for claim "default/www-web-1"
  46m         Normal    ProvisioningSucceeded   persistentvolumeclaim/www-web-1   Successfully provisioned volume pvc-dcbd9627-9b02-4d5a-8be1-c40045c0670c
  46m         Normal    SuccessfulCreate        statefulset/web                   create Pod web-1 in StatefulSet web successful
  46m         Warning   FailedScheduling        pod/web-1                         pod has unbound immediate PersistentVolumeClaims
  46m         Normal    Created                 pod/web-0                         Created container nginx
  46m         Normal    Started                 pod/web-0                         Started container nginx
  46m         Normal    Scheduled               pod/web-1                         Successfully assigned default/web-1 to minikube
  46m         Normal    Pulling                 pod/web-1                         Pulling image "nginx"
  46m         Normal    Pulled                  pod/web-1                         Successfully pulled image "nginx"
  46m         Normal    Started                 pod/web-1                         Started container nginx
  46m         Normal    Created                 pod/web-1                         Created container nginx
  45m         Normal    Killing                 pod/web-1                         Stopping container nginx
  45m         Normal    Killing                 pod/web-0                         Stopping container nginx
  32m         Warning   FailedCreate            statefulset/web                   create Pod web-0 in StatefulSet web failed error: Pod "web-0" is invalid: spec.containers[0].volumeMounts[0].name: Not found: "html"
  18m         Normal    ExternalProvisioning    persistentvolumeclaim/www-web-0   waiting for a volume to be created, either by external provisioner "k8s.io/minikube-hostpath" or manually created by system administrator
  18m         Normal    Provisioning            persistentvolumeclaim/www-web-0   External provisioner is provisioning volume for claim "default/www-web-0"
  18m         Normal    ProvisioningSucceeded   persistentvolumeclaim/www-web-0   Successfully provisioned volume pvc-9d3be2c4-3605-4f86-95d2-0e1f7328ded4
  18m         Normal    SuccessfulCreate        statefulset/web                   create Claim www-web-0 Pod web-0 in StatefulSet web success
  7m5s        Warning   FailedCreate            statefulset/web                   create Pod web-0 in StatefulSet web failed error: Pod "web-0" is invalid: spec.containers[0].volumeMounts[0].name: Not found: "html"

From the event log, we can see an error message `create Pod web-0 in StatefulSet web failed error: Pod "web-0" is invalid: spec.containers[0].volumeMounts[0].name: Not found: "html"`. Checking the `mickymouse.yml` by following the path `spec.containers[0].volumeMounts[0].name`, we see a mismatch of names between `volumeMounts.name=html` and `volumeClaimTemplates.metadata.name=www`. This needs to be fixed.

Delete everything created by mickymouse.yml. Remember to delete the PVC as well. By now, you should be able to find relevant commands in this practical to complete the task.

Fifth, clone the repository and start modifying mickymouse.yml locally::

  resops49@resops-k8s-node-17:~$ git clone https://gitlab.ebi.ac.uk/TSI/tsi-ccdoc.git
  Cloning into 'tsi-ccdoc'...
  remote: Enumerating objects: 388, done.
  remote: Counting objects: 100% (388/388), done.
  remote: Compressing objects: 100% (196/196), done.
  remote: Total 4488 (delta 200), reused 302 (delta 145)
  Receiving objects: 100% (4488/4488), 102.69 MiB | 30.49 MiB/s, done.
  Resolving deltas: 100% (1904/1904), done.

  resops49@resops-k8s-node-17:~$ cd tsi-ccdoc/tsi-cc/ResOps/scripts/minikube/

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ nano ./mickymouse.yml

Change name from `html` to `www`. Apply mickymouse.yml to see if the problem is fixed. Things certainly look much better. Everything seems created successfully. Is everything working? If not, why? How are you going to find out? How are you going to fix the problems? ::

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ kubectl apply -f ./mickymouse.yml
  service/nginx created
  statefulset.apps/web created

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ kubectl get pv
  NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
  pvc-8b5f0d99-f4dd-461d-8f46-2514d23e784c   1Gi        RWO            Delete           Bound    default/www-web-0   standard                8s
  pvc-ad55fa2d-d194-4b58-8b6c-815da976d038   1Gi        RWO            Delete           Bound    default/www-web-1   standard                1s

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ kubectl get pvc
  NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  www-web-0   Bound    pvc-8b5f0d99-f4dd-461d-8f46-2514d23e784c   1Gi        RWO            standard       18s
  www-web-1   Bound    pvc-ad55fa2d-d194-4b58-8b6c-815da976d038   1Gi        RWO            standard       11s

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ kubectl get pod
  NAME    READY   STATUS    RESTARTS   AGE
  web-0   1/1     Running   0          27s
  web-1   1/1     Running   0          20s

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ kubectl get svc
  NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
  kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        7d
  nginx        NodePort    10.106.111.118   <none>        80:31542/TCP   55s

Homework: Additional exercises after the workshop
-------------------------------------------------

There are two more problems and one enhancement for your exercise after the workshop. If you get stuck, compare `mickymouse.yml` against `nginx.yml`. It might be easier if you know what you are looking for.

Here are some general guidance:

#. Check the event log again to confirm that deployment was successful. Use `kubectl get event --help` to learn how to use it.
#. Check the runtime logs to see if there are errors. Use `kubectl logs --help` to learn how to use it.
#. Explore on `Kubernetes Dashboard` to see if you can spot anything unusual. Note that Kubernetes Dashboard would not work in the sandbox as the VM does not have GUI enabled.
#. Connect to pods and containers in the pods to see if everything is configured properly.
#. Test runtime behaviour to see if the system acts as designed. In this case, run `curl http://` to see what happens.

Here are some steps to look for issues. You may wan to continue working on this after the workshop::

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ kubectl get event --sort-by='{.metadata.creationTimestamp}'
  LAST SEEN   TYPE      REASON                  OBJECT                            MESSAGE
  34m         Warning   FailedCreate            statefulset/web                   create Pod web-0 in StatefulSet web failed error: Pod "web-0" is invalid: spec.containers[0].volumeMounts[0].name: Not found: "html"
  10m         Normal    SuccessfulCreate        statefulset/web                   create Claim www-web-0 Pod web-0 in StatefulSet web success
  10m         Normal    ExternalProvisioning    persistentvolumeclaim/www-web-0   waiting for a volume to be created, either by external provisioner "k8s.io/minikube-hostpath" or manually created by system administrator
  10m         Normal    ProvisioningSucceeded   persistentvolumeclaim/www-web-0   Successfully provisioned volume pvc-8b5f0d99-f4dd-461d-8f46-2514d23e784c
  10m         Normal    Provisioning            persistentvolumeclaim/www-web-0   External provisioner is provisioning volume for claim "default/www-web-0"
  10m         Warning   FailedScheduling        pod/web-0                         pod has unbound immediate PersistentVolumeClaims
  10m         Normal    SuccessfulCreate        statefulset/web                   create Pod web-0 in StatefulSet web successful
  10m         Normal    Scheduled               pod/web-0                         Successfully assigned default/web-0 to minikube
  10m         Normal    Pulling                 pod/web-0                         Pulling image "nginx"
  10m         Normal    Pulled                  pod/web-0                         Successfully pulled image "nginx"
  10m         Normal    Created                 pod/web-0                         Created container nginx
  10m         Normal    Started                 pod/web-0                         Started container nginx
  10m         Normal    ExternalProvisioning    persistentvolumeclaim/www-web-1   waiting for a volume to be created, either by external provisioner "k8s.io/minikube-hostpath" or manually created by system administrator
  10m         Normal    Provisioning            persistentvolumeclaim/www-web-1   External provisioner is provisioning volume for claim "default/www-web-1"
  10m         Warning   FailedScheduling        pod/web-1                         pod has unbound immediate PersistentVolumeClaims
  10m         Normal    SuccessfulCreate        statefulset/web                   create Claim www-web-1 Pod web-1 in StatefulSet web success
  10m         Normal    SuccessfulCreate        statefulset/web                   create Pod web-1 in StatefulSet web successful
  10m         Normal    ProvisioningSucceeded   persistentvolumeclaim/www-web-1   Successfully provisioned volume pvc-ad55fa2d-d194-4b58-8b6c-815da976d038
  10m         Normal    Scheduled               pod/web-1                         Successfully assigned default/web-1 to minikube
  10m         Normal    Pulling                 pod/web-1                         Pulling image "nginx"
  10m         Normal    Created                 pod/web-1                         Created container nginx
  10m         Normal    Pulled                  pod/web-1                         Successfully pulled image "nginx"
  10m         Normal    Started                 pod/web-1                         Started container nginx

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ kubectl logs web-0 --all-containers=true

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ kubectl exec -it web-0 -- bash
  root@web-0:/# ls -l /usr/share/nginx/html
  total 0

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ kubectl get svc
  NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
  kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        7d
  nginx        NodePort    10.106.111.118   <none>        80:31542/TCP   15m

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ curl http://10.106.111.118
  curl: (7) Failed to connect to 10.106.111.118 port 80: Connection refused

  resops49@resops-k8s-node-17:~/tsi-ccdoc/tsi-cc/ResOps/scripts/minikube$ curl http://10.106.111.118:31542
  curl: (7) Failed to connect to 10.106.111.118 port 31542: Connection timed out
