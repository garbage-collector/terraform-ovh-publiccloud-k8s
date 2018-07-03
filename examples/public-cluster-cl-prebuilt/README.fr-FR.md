# Cluster Kubernetes depuis une Image Glance préconstruite

Cet exemple montre comment utiliser le module terraform-ovh-publiccloud-k8s pour démarrer un cluster Kubernetes sur le Public Cloud OVH, en se basant sur une image CoreOS Stable avec kubernetes pré-installé.

## Pré-requis

- Une installation de Terraform

  Vous pouvez retrouver toutes les informations pour installer Terraform [ici](https://www.terraform.io/intro/getting-started/install.html).

- Un project Public Cloud OVH

  Créer un projet Public Cloud en suivant la [documentation officielle](https://docs.ovh.com/fr/public-cloud/debuter-avec-public-cloud-premiere-connexion/).

  Créer un utilisateur Openstack ([documentation officielle](https://docs.ovh.com/fr/public-cloud/creer-un-acces-a-horizon/#acces-a-horizon)).

  Télécharger ensuite le fichier de configuration OpenStack. Vous pouvez le retrouver sur votre [Manager OVH](https://www.ovh.com/manager/cloud/), ou sur [l'interface Horizon](https://horizon.cloud.ovh.net/project/api_access/openrc/).

  Prenez le fichier de configuration comme source de votre environnement:

  ```bash
  $ source openrc.sh
  Please enter your OpenStack Password:
  ```
  
- (Optionnel) Installer le CLI OpenStack

  ```bash
  $ sudo pip install python-openstackclient==3.15.0
  ```

- Une clé SSH ajoutée à votre Projet Public Cloud.


  Exemple: 

   ```bash
   # Generate a new keypair without passphrase
   $ ssh-keygen -f terraform_ssh_key -q
   # Add it to the ssh-agent 
   $ eval $(ssh-agent)
   $ ssh-add terraform_ssh_key
   ```
   
   Ou:
   
   ```bash
   $ openstack keypair create -f value k8s > ssh_key
   $ openstack keypair show --public-key -f value k8s > ssh_key.pub
   $ chmod 0600 ./ssh_key
   # Add it to the ssh-agent
   $ eval $(ssh-agent)
   $ ssh-add ./ssh_key
   ```

## Quickstart

### Initialisation

```bash
$ terraform init
Initializing modules...
- module.network
- module.kube
[...]
Terraform has been successfully initialized!
```

### Lancer le Cluster

Vous devez choisir une région OpenStack pour lancer le Cluster, ainsi qu'un nom de clé SSH. Vous pouvez, au choix, placer ces variables dans le fichier de customisation `.tfvars` ou les passer en ligne de commande.

Afin de lister les régions disponibles pour démarrer un cluster, vous pouvez saisir cette commande:

```bash
$ openstack catalog show nova
```

N'oubliez pas de changer votre région si nécessaire:

```bash
$ export OS_REGION_NAME=GRA3
```

Si vous souhaitez lister les clés SSH ajoutées dans cette région:

```bash
$ openstack keypair list
```

Puis, lancez cette commande afin de démarrer votre Cluster Kubernetes. N'oubliez pas de modifier le nom de la paire de clé SSH (key_pair) par le nom de votre clé ajoutée au préalable.

```bash
$ terraform apply -var region=$OS_REGION_NAME -var key_pair=k8s
```

A la fin de cette commande, vous devriez disposer d'une infrastructure avec 3 masters Kubernetes dans un réseau public, avec Canal (Flannel + Calico) CNI, des noeuds Untainted (les pods peuvent démarrer sur les masters) et kube-proxy pour toute la partie services. 

## Démarrer avec Kubernetes

Tapez cette commande pour plus d'information sur l'utilisation de votre Cluster Kubernetes:

```bash
$ terraform output helper
Your kubernetes cluster is up.

Retrieve k8s configuration locally:

    $ mkdir -p ~/.kube/myk8s
    $ ssh core@A.B.C.D sudo cat /etc/kubernetes/admin.conf > ~/.kube/myk8s/config
    $ kubectl --kubeconfig ~/.kube/myk8s/config get nodes

You can also ssh into one of your instances:

    $ ssh core@A.B.C.D
    $ ssh core@A.B.D.E
    $ ssh core@A.B.C.F

Enjoy!
```
