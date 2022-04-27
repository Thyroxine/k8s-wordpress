## Wordpress in Kubernetes

1. Install [Helm](https://helm.sh/docs/intro/install/)
2. Install **NFS Server** on Infrastructure Machine (AlmaLinux). RAID5 for Wordpress Files is mounted to `/mnt/bigvol`, RAID10 for MySQL to `/mnt/smallvol`
```shell
sudo dnf -y install nfs-utils
sudo cp ./nfs-config/exports /etc/exports
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --reload
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
sudo chmod 777 /mnt/smallvol # Needed for provisioner to work
sudo chmod 777 /mnt/bigvol
sudo exportfs -arv # Check that export works correctly
```
3. Install the NFS client package to K8s hosts:
```shell
sudo dnf install nfs-utils nfs4-acl-tools         [On CentOS/RHEL]
sudo apt install nfs-common nfs4-acl-tools   [On Debian/Ubuntu]
```
4. Deploy NFS PV provisioners:
```shell
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-raid5 nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -f helm-values/helm-nfs-raid5.values.yaml
helm install nfs-raid10 nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -f helm-values/helm-nfs-raid10.values.yaml
```
5. Put Base64-encoded passwords into Secrets section of file `wordpress/wordpress-config.yaml`
6. Deploy Wordpress
```shell
kubectl apply -k wordpress/
```
7. Navidate to https://k8s-sth33.srequest.ru and finish the installation.
8. You may deploy one WP pod for setup and then scale to as many as you want - they have shared storage for uploads and themes!
