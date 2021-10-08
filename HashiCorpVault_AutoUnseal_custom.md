1. Create a secret and store your master key and root token there:

```
    % oc get secret vault-master-key
    NAME               TYPE     DATA   AGE
    vault-master-key   Opaque   2      12d

    content:
    apiVersion: v1
    data:
      master-key: *****
      root_token: *****
    kind: Secret
```

2. Edit statefulset and add following:

```
lifecycle:
  postStart:
    exec:
      command:
        - /bin/sh
        - '-c'
        - sleep 30 && cat /etc/masterKey/master-key | xargs vault operator unseal
.....

volumeMounts:
.....
    - name: masterkey
      mountPath: /etc/masterKey
......
volumes:
- name: masterkey
  secret:
    secretName: vault-master-key
    defaultMode: 420
```

3. Restart pod
