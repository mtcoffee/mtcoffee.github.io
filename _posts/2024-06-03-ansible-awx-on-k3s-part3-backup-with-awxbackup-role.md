---
title: Ansible AWX on K3s Part3 - Backup with AWXBackup Role
---

# Summary
You don't technically need to use the AWXBackup role since simple postgress backup and copy of the secrety key is all that is needed, but this can be a handy way to automate backup and restore. In this example we will backup to a file mount to simulate backing up to an NFS store.

# Create the Backup 
First create a host path save to the backup to.  Once you have this, you can create a PVC that points to the path. Remember, this is a simulation and your best option is to create a PVC that points to a NFS mount or S3 mount.

```
sudo mkdir /mnt/data
sudo chmod 755 /mnt/data
sudo chown 26:26 /mnt/data #grant ownership to the postgres user that the pg container runs as

kubectl apply -f - <<EOF
#create pv
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: awx-pv-volume-local
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
#create pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: awx-backup-claim-mnt-data
  namespace: awx
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
EOF

#now create backup job
kubectl apply -f - <<EOF
---
apiVersion: awx.ansible.com/v1beta1
kind: AWXBackup
metadata:
     name: awxbackup-$(date +%Y-%m-%d-%H%M)
     namespace: awx
spec:
     deployment_name: ansible-awx
     clean_backup_on_delete: True
     backup_pvc: awx-backup-claim-mnt-data #pre-existing claim
EOF
```

After minute or so you should have backup in the path /mnt/data/tower-openshift-backup-{date}

If you run into trouble, use each of these commands

```
kubectl -n awx get awxbackup
kubectl -n awx logs -f deployments/awx-operator-controller-manager | grep awxbackup
kubectl describe awxbackup -n awx awxbkup 
```

# Restore the Backup
To restore the mount you can use the backup path as follows. On a full host restore you must create the PVC again first.

```
#delete the existing instance and postgres pvc (mandatory)
kubectl delete awx ansible-awx -n awx
kubectl delete pvc postgres-15-ansible-awx-postgres-15-0 -n awx

#run the restore
kubectl apply -f - <<EOF
---
apiVersion: awx.ansible.com/v1beta1
kind: AWXRestore
metadata:
  name: awxrestore-testing-folder
  namespace: awx
spec:
  deployment_name: ansible-awx
  backup_dir: /backups/tower-openshift-backup-2024-05-30-165303
  backup_pvc: awx-backup-claim
EOF
```

To verify the status of a restore, use each of these commands

```
 kubectl describe awxrestore awxrestore-testing-folder -n awx

 kubectl get awxrestore awxrestore-testing-folder -n awx -o yaml

 kubectl -n awx logs -f deployments/awx-operator-controller-manager | grep awxrestore
```
