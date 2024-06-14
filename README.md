# Vaultwarden K8S Manifests

This is a simple [vaultwarden](https://github.com/dani-garcia/vaultwarden/wiki) setup for kubernetes using plain k8s manifest.
This implementation is made for myself and for my use-case, because of that NFS is used for storage.

In `manifests/vaultwarden` the manifests for vaultwarden itself can be found. 
Next to that in `manifests/vaultwarden-backup` you can find the resources which automate backups by utilizing CronJobs.

In case you want to use this, please note that you will have to replace all occurrences of values wrapped in <> characters. 