## Wordpress in Kubernetes

1. Install Helm
2. Install NFS Server on Infrastructure Machine. RAID5 for Wordpress Files is mounted to `/mnt/bigvol`, RAID10 for MySQL to `/mnt/smallvol`
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
2. Deploy NFS PV provisioners:
```shell
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-raid5 nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -f helm-values/helm-nfs-raid5.values.yaml
helm install nfs-raid10 nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -f helm-values/helm-nfs-raid10.values.yaml
```
3. Put Base64-encoded passwords into Secrets `wordpress/wordpress-config.yaml`
4. Deploy Wordpress
```shell
kubectl apply -k wordpress/
```
5. Navidate to https://k8s-sth33.srequest.ru and finish the installation.
6. You may deploy one WP pod for setum and then scale to as many as you want - they have shared storage for uploads and themes!
