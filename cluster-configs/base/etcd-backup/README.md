# etcd backups
Etcd backups are taken daily at midnight at sent to an external Minio S3 bucket. They are stored for 5 days before the script removes them. These values are customizable.

The following component consists of:

* "Secret" (Manually handled for now, not in git)
* Config map
* Namespace
* Custom role and rolebinding
* Service account running the backup
* Cronjob which is scheduled for every day at midnight

## Environment Variables
The following variables are defined in the secret object.

- **AWS_CLI_OPTIONS**: Additional options to pass to the AWS CLI. Example: `--no-verify-ssl`.
- **AWS_BUCKET_ID**: The ID of the S3 bucket used for backups. Example: r6-mgmthub-etcd-backup
- **AWS_ACCESS_KEY_ID**: Value in Hashicorp Vault
- **AWS_SECRET_ACCESS_KEY**: Value in Hashicorp Vault
- **AWS_ENDPOINT_URL**: The S3 endpoint Example: https://s3-endpoint

The backup is being sent to a Minio Cluster at:
- https://s3-endpoint


## Command run by the script
```sh
mkdir -p /host/tmp/backup && touch /host/tmp/backup/init.txt && rm /host/tmp/backup/* && chroot /host /usr/local/bin/cluster-backup.sh /tmp/backup && aws ${AWS_CLI_OPTIONS} s3 cp --recursive /host/tmp/backup s3://${AWS_BUCKET_ID} && echo Purging old files... && ((aws ${AWS_CLI_OPTIONS} s3 ls ${AWS_BUCKET_ID}; date -d "5 days ago" +"%Y-%m-%d 00:00:00 DELETETOHERE")|sort -k1|sed -n '/DELETETOHERE/q;p'|awk '{print $4}'|xargs -I '{}' aws ${AWS_CLI_OPTIONS} s3 rm s3://${AWS_BUCKET_ID}/'{}')
```
> NOTE: We are adding a placeholder file in the /host/tmp/backup directory to ensure the script does not error out on new clusters

## Link of interest 
* [etcd backups](https://docs.openshift.com/container-platform/4.14/backup_and_restore/control_plane_backup_and_restore/backing-up-etcd.html)
* [etcd restores](https://docs.openshift.com/container-platform/4.14/backup_and_restore/control_plane_backup_and_restore/disaster_recovery/scenario-2-restoring-cluster-state.html)

## Useful commands
Verify backups are stored in the s3 bucket:
```bash
Insert command for checking this
```

Try running the script manually (This container image contains the AWS client):
```bash
oc debug node/NODENAME --image=quay.io/redhat-cop/tool-box:latest
```
Export the AWS variables and run the script manually.