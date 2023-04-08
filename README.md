# cep-project
SimpliLearn PG - Kubernetes coures project (Kubernetes Dashboard)

## Cluster Admin Setup

#### mkdir cep-project folder 
`mkdir cep-project && cd cep-project`

#### copy the admin files into this folder
`ls ./`

#### apply the role yaml
`k apply -f role.yaml`

#### apply the roledinding file
`k apply -f rolebinding.yaml`

#### apply the service account yaml
`k apply -f sandry-sa.yml`

#### list the secrets in the cep-project1 namespace
`k get secrets -n cep-project1`

#### copy the sandry token secret resource name and get the token from it. Since the secret resource type in kubernetes encodes the data in base64, we have to decode it (the echo command is just for new line)
`k get secrets sandry-token-7bmsc -n cep-project1 -o jsonpath='{.data.token}' | base64 --decode && echo`

#### install kubernetes dashboard
`k apply -f  https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml`

#### confirm the pods are running
`k get pods -n kubernetes dashboard`

#### currently the service created for the pods are of type clusterIP, we need to change to nodeport
#### list the services in the kubernetes-dashboard namespace
`k get svc -n kubernetes-dashboard`

#### edit kubernetes-dashboard service and change type to nodeport
`k edit svc -n kubernetes-dashboard kubernetes-dashboard`

#### Confirm service type change and copy the port number
`k get svc -n kubernetes-dashboard`

On the VM monitor, open the browser and use this address (sandry can use this link to create her application)
`https://localhost:<nodeport>`. sandry can paste her token and because it is a self signed certificate, sandry can accept the risk and continue.

## Prepare NFS Storage On Node 3 (worker node 2)

#### install the nfs server on node 3
sudo apt update && sudo apt install nfs-kernel-server

#### create a directory to the shared on node 3
`sudo mkdir /mnt/db`
`sudo mkdir /mnt/wp`

#### update the permissions so other users can access the contents
`sudo chmod -R 1777 /mnt/db`
`sudo chmod -R 1777 /mnt/wp`

#### edit the nfs server file to share out the newly created directory and allow only ip range in the subnet to access
`sudo vi /etc/exports`
#### Paste the below lines
`/opt/db 172.16.0.0/16(rw,sync,no_root_squash,subtree_check)`

`/opt/wp 172.16.0.0/16(rw,sync,no_root_squash,subtree_check)`

#### Initialize the master export table with contents of /tec/exports
`sudo exportfs -ra`

#### create pv for the nfs storage so that the application can claim it.
`k apply -f pv-nfs-1.yml`

`k apply -f pv-nfs-2.yml`
