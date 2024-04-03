

# KEYCLOAK PROJECT

- [KEYCLOAK PROJECT](#keycloak-project)
  - [ENVIRONMENT SETUP](#environment-setup)
    - [0. Prerequisite](#0-prerequisite)
    - [1. Install Vagrant](#1-install-vagrant)
    - [2. Provision VM using Vagrant with microk8s cluster](#2-provision-vm-using-vagrant-with-microk8s-cluster)
    - [3. Access VM](#3-access-vm)
  - [RUNNING APPLICATION](#running-application)
  - [ACCESSING APPLICATION](#accessing-application)
  - [TESTING APPLICATION (not wokring with remote infinispan)](#testing-application-not-wokring-with-remote-infinispan)
    - [1. Login into the app](#1-login-into-the-app)
    - [2. Create realm](#2-create-realm)
    - [3. Create user](#3-create-user)
    - [4. Create client](#4-create-client)
    - [5. Testing login](#5-testing-login)
  - [REVERTING CHANGES](#reverting-changes)
    - [Stopping application](#stopping-application)
    - [Exit VM terminal](#exit-vm-terminal)
    - [Stop VM](#stop-vm)
    - [Delete VM](#delete-vm)
  - [References:](#references)


## ENVIRONMENT SETUP

Microk8s cluster is used for deployment. 
Cluster is deployed using vagrant which will provision Virtual Machine with microk8s setup and required tools.
Additionally during microk8s setup, ingress and hostpath storage plugins will be enabled.
Ingress is used for accessing application, while hostpath storage for provisioning Persistent Volumes.

### 0. Prerequisite

Installed Virtual Box

### 1. Install Vagrant

Install Vagrant from link bellow

https://developer.hashicorp.com/vagrant/downloads

### 2. Provision VM using Vagrant with microk8s cluster

Run `vagrant up` command from the project root directory to provision VM with installed microk8s cluster. This process will take several minutes.
IP address of the machine can be hardcoded or dynamically set. Default value is hardcoded in the Vagrantfile `config.vm.network "private_network", ip: "192.168.56.7"` This value is used for accessing ingress domain, and changing it requires updating ingress URL as well.

```bash
# Confirm vagrant version
vagrant version

# Provision Virtual Machine
vagrant up
```

### 3. Access VM

Once the environment has been provisioned, access virtual machine terminal using `vagrant ssh` command.
Confirm that microk8s is running with `microk8s status` command. It should show running status, and enabled plugins. `hostpath-storage` and `ingress` plugins should be enabled. If they are not enabled run following commands:

```bash
microk8s enable hostpath-storage
microk8s enable ingress
```

During VM provisioning, `.bash_aliases` are also setup for commonly accessible kubernetes shortcuts.
```bash
alias kubectl='microk8s.kubectl'
alias k='microk8s.kubectl'
alias d="microk8s.kubectl get deploy"
alias p="microk8s.kubectl get pods"
alias s="microk8s.kubectl get svc"
alias a="microk8s.kubectl apply -f "
alias r="microk8s.kubectl delete -f "
```
By default, storage class will create persisten volumes under `/var/snap/microk8s/common/default-storage`. Default path can be changed by [creating new storage class](https://microk8s.io/docs/addon-hostpath-storage#customize-directory-used-for-persistentvolume-3).

## RUNNING APPLICATION

Navigate to kubernetes manifest directory and execute command bellow:

    kubectl apply -f 2_kc_db_infinispan_remote

or shorthand `a 2_kc_db_infinispan_remote` to use bash alias.

Application is deployed using PostgresDB, Keycloak and Infinispan pods.
Application startup process is currently slow, so it is expected to take several minutes until domain becomes available.

## ACCESSING APPLICATION

Application can be accessed using Ingress resouce.
If host ip in the Vagrant file was not changed (`config.vm.network "private_network", ip: "192.168.56.7"`) Keycloak should be accessible on `keycloak.192.168.56.7.nip.io` URL.
Open **keycloak.192.168.56.7.nip.io** URL and accept Unsecure domain (current implementation does not use certificate) for SSL.

## TESTING APPLICATION (not wokring with remote infinispan)

### 1. Login into the app
Click on `Administration Console` to access http://keycloak.192.168.56.7.nip.io/admin/ page and accept Security Warning.
Sign-in using provisioned credentials username: `admin`, password `admin`.

### 2. Create realm
Click on the `master` dropdown in the top left corner and click `Create Realm` button.
Populate `Realm Name` (e.g. *myrealm*) and save.

### 3. Create user
Navigate to `Users` and click `Create New User` button.
Fill in `Username` filed and click `Create`.
On `Credentials` tab click `Set Password`. Fill in password and uncheck `Set Temporary` and save.
Confirm to save password.

### 4. Create client
Navigate to `Clients` and click `Create Client` button.
Populate `ClientID` field (e.g *myclient*) and click Next.
For second step, click Next again.
On `Login Settings` page make these changes.
    Set `Valid redirect URIs` to https://www.keycloak.org/app/*
    Set `Web origins` to https://www.keycloak.org
Save the changes.

### 5. Testing login

Open next URL.

> **_NOTE:_**   If URL was changed in the Vagrantfile, update URL with new address


```bash
# Use this link as a template and update REALM_NAME and CLIENT_NAME
https://www.keycloak.org/app/#url=http://keycloak.192.168.56.7.nip.io/&realm=REALM_NAME&client=CLIENT_NAME

# e.g.
https://www.keycloak.org/app/#url=http://keycloak.192.168.56.7.nip.io/&realm=myrealm&client=myclient
```

Try to login using user from step 3.

## REVERTING CHANGES

### Stopping application

To destroy all Kubernetes pods run command bellow from manifests directory
`kubectl delete -f .`
or `r .` for shorthand command.

### Exit VM terminal

`exit`

### Stop VM

`vagrant halt`

### Delete VM

`vagrant destroy`

## References:

https://www.keycloak.org/high-availability/connect-keycloak-to-external-infinispan

https://www.keycloak.org/server/caching

https://www.keycloak.org/server/all-config

https://infinispan.org/docs/stable/titles/server/server.html#hot_rod

https://docs.jboss.org/infinispan/9.2/pdf/server_guide.pdf

https://infinispan.org/docs/stable/titles/configuring/configuring.html
