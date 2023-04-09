# cep-project 1
SimpliLearn PG - Kubernetes coures project (Kubernetes Dashboard)

## Cluster Admin Setup

1. Clone this repo in your $HOME directory and go to the cep-project/admin/access directory - `git clone https://github.com/klemzi/cep-project.git && cd cep-project/admin/access`

2. List ythe files this folder - `ls ./`

3. Apply the role yaml - `k apply -f role.yaml`

4. Apply the roledinding file -`k apply -f rolebinding.yaml`

5. Apply the service account yaml - `k apply -f sandry-sa.yml`

6. List the secrets in the cep-project1 namespace - `k get secrets -n cep-project1`

7. Copy the sandry token secret resource name and get the token from it. Since the secret resource type in kubernetes encodes the data in base64, we have to decode it (the echo command is just for new line) - `k get secrets sandry-token-7bmsc -n cep-project1 -o jsonpath='{.data.token}' | base64 --decode && echo`

8. Install kubernetes dashboard - `k apply -f  https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml`

9. Confirm the pods are running - `k get pods -n kubernetes dashboard`

10. Currently, the service created for the pods are of type clusterIP, we need to change to nodeport. List the services in the kubernetes-dashboard namespace - `k get svc -n kubernetes-dashboard`

11. edit kubernetes-dashboard service and change type to nodeport - `k edit svc -n kubernetes-dashboard kubernetes-dashboard`

12. Confirm service type change and copy the port number - `k get svc -n kubernetes-dashboard`

13. On the VM monitor, open the browser and use this address (sandry can use this link to create her application)
`https://localhost:<nodeport>`. sandry can paste her token and because it is a self signed certificate, sandry can accept the risk and continue.

## Prepare NFS Storage On Node 3 (worker node 2)

1. install the nfs server on node 3 - `sudo apt update && sudo apt install nfs-kernel-server`

2. Create a directory to the shared on node 3.

`sudo mkdir /mnt/db`

`sudo mkdir /mnt/wp`

3. Update the permissions so other users can access the contents.

`sudo chmod -R 1777 /mnt/db`

`sudo chmod -R 1777 /mnt/wp`

4. Edit the nfs server file to share out the newly created directory and allow only ip range in the subnet to access - `sudo vi /etc/exports`

5. Paste the below lines

`/opt/db 172.16.0.0/16(rw,sync,no_root_squash,subtree_check)`

`/opt/wp 172.16.0.0/16(rw,sync,no_root_squash,subtree_check)`

6. Initialize the master export table with contents of /etc/exports - `sudo exportfs -ra`

7. Create pv for the nfs storage so that the application can claim it

`k apply -f pv-nfs-1.yml`

`k apply -f pv-nfs-2.yml`
