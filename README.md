# Cop  K8S Ops Datio

Comunidad de conocimiento y práctica donde desplegaremos un cluster de K8s con Kubespray en Openstack

![](https://img.shields.io/github/last-commit/felixrod78/cop_k8s) ![](https://img.shields.io/github/issues-pr/felixrod78/cop_k8s?style=plastic) ![](https://img.shields.io/github/languages/code-size/felixrod78/cop_k8s) ![](https://img.shields.io/badge/CoP-K8s-brightgreen)


### Objetivos

- Creación infraestructura automatizada
- Despliegue básico cluster de K8S con Kubespray

##  Paso a paso

### Creación infra en Openstack

Descarga archivo credenciales desde Horizon
 ```bash
   source  openstack.sh
```

Generamos dos roles en ansible:
- Crear instancia
- Configurar instancia

En dichos roles, configuraremos las instancias,repos,claves de acceso.

```bash
ansible-playbook main.yml
```
### Despliegue cluster K8S con Kubespray

Pasos a seguir desde tu maquina controller (local)

```bash
 apt -y install python3-pip
 git clone https://github.com/kubernetes-sigs/kubespray.git
 cd kubespray
 pip3 install -r requirements.txt
```
```bash
cp -rfp inventory/sample inventory/mycluster
declare -a IPS=(IPs instancias creadas)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```
Importante tener en cuenta lo siguiente al hacer el deploy en un entorno que no este en tu misma red local:
```bash
all:
  hosts:
    node1:
      ansible_host: IP Publica de acceso
      ansible_user: cloud-user 
      ip: IP privada
      access_ip: : IP Publica de acceso
 ```

Configurar tus variables 

```bash
cat inventory/mycluster/group_vars/all/all.yml
cat inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
```

Despliegue del cluster:

```bash
ansible-playbook -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml
```
Para poder gestionar el cluster desde tu maquina:

Server Master remoto
```bash
sudo cp /root/.kube/config /home/cloud-user/
sudo chown cloud-user /home/cloud-user/config 
```
Maquina Local
```bash
scp cloud-user@172.16.51.179:/home/cloud-user/config conections/
```
En el archivo kubeconfig, poner IP publica de acceso en el caso que sea necesario

Para testear el status del cluster
```bash
kubectl --kubeconfig /home/frodriguez/Desktop/FORMACION/cop_k8s/conections/config get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   23h   v1.17.5
node1    Ready    <none>   23h   v1.17.5
node2    Ready    <none>   23h   v1.17.5
```


# cd ~

cp -rfp inventory/sample inventory/mycluster
declare -a IPS=(172.16.49.51 172.16.49.66  172.16.49.93)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
Review and change the host configuration file - inventory/mycluster/hosts.yaml.

Example:

inventory/mycluster/hosts.yaml
all:
  hosts:
    node1:
      ansible_host: 192.168.222.111
      ip: 192.168.222.111
      access_ip: 192.168.222.111
    node2:
      ansible_host: 192.168.222.101
      ip: 192.168.222.101
      access_ip: 192.168.222.101
    node3:
      ansible_host: 192.168.222.102
      ip: 192.168.222.102
      access_ip: 192.168.222.102
    node4:
      ansible_host: 192.168.222.103
      ip: 192.168.222.103
      access_ip: 192.168.222.103
    node5:
      ansible_host: 192.168.222.104
      ip: 192.168.222.104
      access_ip: 192.168.222.104
  children:
    kube-master:
      hosts:
        node1:
    kube-node:
      hosts:
        node2:
        node3:
        node4:
        node5:
    etcd:
      hosts:
        node1:
    k8s-cluster:
      children:
        kube-master:
        kube-node:
    calico-rr:
      hosts:
Review and change cluster installation parameters under inventory/mycluster/group_vars:

# cat inventory/mycluster/group_vars/all/all.yml
# cat inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
The default Kubernetes CNI can be changed by setting the desired kube_network_plugin value parameter in inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml.


### configure repo

^C
[root@node1 cloud-user]# cat /etc/yum.repos.d/default.repo 
[rhel-7-server-rpms]
baseurl = http://172.16.49.46/repos/rhel-7-server-rpms/
enabled = 1
gpgcheck = 0
name = rhel-7-server-rpms
proxy=_none_

[rhel-7-server-extras-rpms]
baseurl = http://172.16.49.46/repos/rhel-7-server-extras-rpms/
enabled = 1
gpgcheck = 0
name = rhel-7-server-extras-rpms
proxy=_none_

[rhel-7-server-optional-rpms]
baseurl = http://172.16.49.46/repos/rhel-7-server-optional-rpms/
enabled = 1
gpgcheck = 0
name = rhel-7-server-optional-rpms
proxy=_none_

[rhel-7-server-rh-common-rpms]
baseurl = http://172.16.49.46/repos/rhel-7-server-rh-common-rpms/
enabled = 1
gpgcheck = 0
name = rhel-7-server-rh-common-rpms
proxy=_none_

[rhel-7-server-supplementary-rpms]
baseurl = http://172.16.49.46/repos/rhel-7-server-supplementary-rpms/
enabled = 1
gpgcheck = 0
name = rhel-7-server-supplementary-rpms
proxy=_none_




wget http://mirror.centos.org/centos/7/extras/x86_64/Packages/slirp4netns-0.4.3-4.el7_8.x86_64.rpm
yum install wget
yum localinstall slirp4netns-0.4.3-4.el7_8.x86_64.rpm

Install K8s Cluster Using Ansible Playbook
Deploy K8s cluster with Kubespray Ansible Playbook:

# ansible-playbook -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml
Server remoto
 sudo cp /root/.kube/config /home/cloud-user/
sudo chown cloud-user /home/cloud-user/config 

Local
scp cloud-user@172.16.51.179:/home/cloud-user/config conections/
cambiar ip a publica 

 kubectl --kubeconfig /home/frodriguez/Desktop/FORMACION/cop_k8s/conections/config get namespaces
