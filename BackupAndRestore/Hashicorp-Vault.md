- [Introduction](#introduction)
- [Backup Hashicorp Vault](#backup-hashicorp-vault)
- [Restore Hashicorp Vault](#restore-hashicorp-vault)

## Introduction
This procedure was prepared for a Hashicorp Vault deployment on Openshift 4.x platform.

Most of the solution online relies on RAFT mechanism. But this one uses one single hashicorp vault instance running on Openshift and also Openshift Container Storage. Since OCP and OCS are already clustered we did not see any reason to deploy raft in our environment.

Here there are the instructions how to export data from Hashicorp Vault and restore back using vault migrate option to a folder.

<br>


## Backup Hashicorp Vault

1. Change your project to Hashicorp Vault.

    ```
    $ oc project hashicorp
    ```

2. Connect to your vault container.
    ```
    % oc get pods
    NAME                                    READY   STATUS    RESTARTS   AGE
    vault-0                                 1/1     Running   0          44h
    vault-agent-injector-xxx-yyy   1/1     Running   0          44h

    % oc rsh vault-0
    / $

    ```

3. Create migrate configuration.

    ```
    % oc rsh vault-0
    / $ cat << 'EOF' > /tmp/backup.hcl
    storage_source "file" {
    path = "/vault/data"
    }

    storage_destination "file" {
    path = "/tmp/backup"
    }
    EOF
    ```
   *** Important Note ***
   Please be sure you are using default path for Vault data. Default: /vault/data

4. Run Migrate inside the container and compress backup folder 
    ```
    $ vault operator migrate -config /tmp/backup.hcl
    $  tar czvf /tmp/backup.tar.gz /tmp/backup
    ```

5. Copy backup from container.
    ```
    oc rsync vault-0:/tmp/backup.tar.gz .
    ```
## Restore Hashicorp Vault

1. Change your project to Hashicorp Vault.

    ```
    $ oc project hashicorp
    ```

2. Copy backup file to container and extract.
    ```
    % oc get pods
    NAME                                    READY   STATUS    RESTARTS   AGE
    vault-0                                 1/1     Running   0          44h
    vault-agent-injector-xxx-yyy   1/1     Running   0          44h

    % oc cp backup.tar.gz vault-0:/tmp/
    % oc rsh vault-0
     $ tar xvfz /tmp/backup.tar.gz 

    ```

3. Create restore configuration.

    ```
% oc rsh vault-0
$ cat << 'EOF' > /tmp/restore.hcl
storage_source "file" {
  path = "/tmp/backup"
}

storage_destination "file" {
  path = "/vault/data"
}
EOF
    ```
   *** Important Note ***
   Please be sure you are using default path for Vault data. Default: /vault/data
   Please be sure you are using right path for Vault backup you extracted above. 

4. Run Migrate inside the container.
    ```
    $ vault operator migrate -config /tmp/restore.hcl
    ```

5. Restart vault pod
    ```
    oc delete pod vault-int-0
    ```
6. Same unseal key(s) and tokens can be used.