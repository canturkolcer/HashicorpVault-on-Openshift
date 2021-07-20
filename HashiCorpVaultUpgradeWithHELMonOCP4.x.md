# HashiCorp Vault Upgrade With HELM on OCP4.x

- [HashiCorp Vault Upgrade With HELM on OCP4.x](#hashicorp-vault-upgrade-with-helm-on-ocp4x)
  - [Introduction](#introduction)
  - [Upgrade HashiCorp Vault on Openshift 4.x](#upgrade-hashicorp-vault-on-openshift-4x)
  - [Rollback HashiCorp Vault on Openshift 4.x](#rollback-hashicorp-vault-on-openshift-4x)

## Introduction
Upgrade Hashicorp Vault on Openshift 4.x using HELM

In this document, there are steps applied to upgrade HashiCorp Vault on Openshift 4.x and rollback if needed.

Topology:

* Openshift version 4.6
* HELM 3
* Hashicorp Vault 1.5.2
  
<br>

## Upgrade HashiCorp Vault on Openshift 4.x

1. Check if you have Hashicorp Repository is added or not. If not add it.

  ```
  % helm repo list
  Error: no repositories to show

  % helm repo add hashicorp https://helm.releases.hashicorp.com
  "hashicorp" has been added to your repositories

  % helm repo list
  NAME             	URL
  hashicorp        	https://helm.releases.hashicorp.com

  % helm repo list
  NAME             	URL
  hashicorp        	https://helm.releases.hashicorp.com

  ```

2. Login to your cluster and change your project to Hashicorp Vault namespace.

  ```
  % oc login 10.72.90.153:6443
  Authentication required for https://10.72.90.153:6443 (openshift)
  Username: kubeadmin
  Password:
  Login successful.

  You have access to 80 projects, the list has been suppressed. You can list all projects with ' projects'

  Using project "default".
  
  % oc project hashicorp
  Already on project "hashicorp" on server "https://10.72.90.153:6443".

  ```  

3. Check your Hashicorp Vault deployment and version
   
   ```
    % helm list
    NAME     	NAMESPACE    	REVISION	UPDATED                              	STATUS  	CHART      	APP VERSION
    vault-int	hashicorp-int	1       	2021-07-20 08:34:35.359349 +0200 CEST	deployed	vault-0.7.0	1.5.2

    % helm history vault-int
    REVISION	UPDATED                 	STATUS    	CHART       	APP VERSION	DESCRIPTION
    1       	Mon Jul 19 16:03:54 2021	superseded	vault-0.7.0 	1.5.2      	Install complete
   ```

4. Upgrade Hashicorp Vault. 

  ```
  % helm  upgrade vault-int hashicorp/vault
  Release "vault-int" has been upgraded. Happy Helming!
  NAME: vault-int
  LAST DEPLOYED: Tue Jul 20 09:10:06 2021
  NAMESPACE: hashicorp-int
  STATUS: deployed
  REVISION: 4
  NOTES:
  Thank you for installing HashiCorp Vault!

  Now that you have deployed Vault, you should look over the docs on using
  Vault with Kubernetes available here:

  https://www.vaultproject.io/docs/


  Your release is named vault-int. To learn more about the release, try:

    $ helm status vault-int
    $ helm get manifest vault-int

  ```

5. Delete Hashicorp Vault pod to start it with upgraded version

  ```
  % oc get pods
  NAME                                        READY   STATUS    RESTARTS   AGE
  vault-int-0                                 1/1     Running   0          9m53s
  vault-int-agent-injector-7c6fb49488-msv85   1/1     Running   0          14s
  
  % oc delete pod vault-int-0
  pod "vault-int-0" deleted
  
  % oc get pods
  NAME                                        READY   STATUS    RESTARTS   AGE
  vault-int-0                                 1/1     Running   2          73s
  vault-int-agent-injector-7c6fb49488-msv85   1/1     Running   0          13m

  ```

6. Confirm new version deployed or not.

   ```
   % oc exec vault-int-0 vault status | grep Version
    Version         1.7.3
   ```  


7. Unseal Vault and keep using.

## Rollback HashiCorp Vault on Openshift 4.x

1. Check if you have Hashicorp Repository is added or not. If not add it.

  ```
  % helm repo list
  Error: no repositories to show

  % helm repo add hashicorp https://helm.releases.hashicorp.com
  "hashicorp" has been added to your repositories

  % helm repo list
  NAME             	URL
  hashicorp        	https://helm.releases.hashicorp.com

  % helm repo list
  NAME             	URL
  hashicorp        	https://helm.releases.hashicorp.com

  ```

2. Login to your cluster and change your project to Hashicorp Vault namespace.

  ```
  % oc login 10.72.90.153:6443
  Authentication required for https://10.72.90.153:6443 (openshift)
  Username: kubeadmin
  Password:
  Login successful.

  You have access to 80 projects, the list has been suppressed. You can list all projects with ' projects'

  Using project "default".
  
  % oc project hashicorp
  Already on project "hashicorp" on server "https://10.72.90.153:6443".

  ```  

3. Check your Hashicorp Vault deployment and version
   
   ```
    % helm list
    NAME     	NAMESPACE    	REVISION	UPDATED                              	STATUS  	CHART       	APP VERSION
    vault-int	hashicorp-int	2       	2021-07-20 09:23:16.805406 +0200 CEST	deployed	vault-0.13.0	1.7.3

    % helm history vault-int
    REVISION	UPDATED                 	STATUS    	CHART       	APP VERSION	DESCRIPTION
    1       	Mon Jul 19 16:03:54 2021	superseded	vault-0.7.0 	1.5.2      	Install complete
    2       	Tue Jul 20 08:29:07 2021	superseded	vault-0.13.0	1.7.3      	Upgrade complete

   ```

4. Rollback Hashicorp Vault. 

  ```
  % helm rollback vault-int 1
  Rollback was a success! Happy Helming!

  ```

5. Delete Hashicorp Vault pod to start it with previous version version

  ```
  % oc get pods
  NAME                                        READY   STATUS    RESTARTS   AGE
  vault-int-0                                 1/1     Running   0          9m53s
  vault-int-agent-injector-7c6fb49488-msv85   1/1     Running   0          14s
  
  % oc delete pod vault-int-0
  pod "vault-int-0" deleted
  
  % oc get pods
  NAME                                        READY   STATUS    RESTARTS   AGE
  vault-int-0                                 1/1     Running   2          73s
  vault-int-agent-injector-7c6fb49488-msv85   1/1     Running   0          13m

  ```

6. Confirm new version deployed or not.

   ```
   % oc exec vault-int-0 vault status | grep Version
    Version         1.5.2
   ```  


7. Unseal Vault and keep using.