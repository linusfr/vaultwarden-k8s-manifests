# how to manage the state of this thing

## backup

- [official guide](https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault)

All the data lies in the `/data` directory.  
The data is divided into a sqllite3 database which can be found under `/data/db.sqlite3` and the remaining data.  
The remaining data is stored on disk and entails the following data:

- sends
- rsa files
- attachments
- config.json

Sadly the [vaultwarden-server image](https://hub.docker.com/r/vaultwarden/server) does not include the sqlite3 binary.  
Since we need to access the data on the same data whilst avoiding ReadWriteMany we must utilize a sidecar here.  
So firstly you must extend the statefulset with another container.  
Since containers sit in the same pvc sharing the same volumes we can access the database from the second container.

```yaml
spec:
  template:
    spec:
      containers:
        - name: sqlite3
          image: keinos/sqlite3
          args: [/bin/sh, -c, "tail -f /dev/null"]
          volumeMounts:
            - name: vaultwarden-data
              mountPath: /data
        - name: vaultwarden
          image: vaultwarden/server:latest
          volumeMounts:
            - name: vaultwarden-data
              mountPath: /data
          ...
          volumeMounts:
            - name: vaultwarden-data
              mountPath: /data
```

After applying the patched yaml the statefulset will rotate creating a new pods with two containers.  
You can now use the second container to create the database dump.

```
kubectl exec -it --container sqlite3 -n vaultwarden vaultwarden-0 -- sqlite3 /data/db.sqlite3 ".backup /tmp/backup.sq3"
```

The backup can now be found under `/tmp/backup.sq3`.  
In the next step you need to transfer this backupfile to your new instance.
Because the build in copy function from kubectl does not provide pod-to-pod data transfer we need to pull the data to our local system first.

```
kubectl cp --container sqlite3 vaultwarden/vaultwarden-0:/tmp/backup.sq3 /tmp/backup.sq3
```

Now we can push the data onto the restore target.  
On the target container we also need a sidecar, because the sqlite3 binary is of course also missing here.

```
kubectl cp --container sqlite3 /tmp/backup.sq3 vaultwarden/vault-0:/tmp/backup.sq3
```

## restore

[RESTORE](https://www.ibiblio.org/elemental/howto/sqlite-backup.html)

```
sqlite3 /data/db.sqlite3 ".restore backup.sq3"
```

TODO: data migration -> copy /data directory for sends, attachments and rsa files

Must be root for permissions!