# Cop  K8S Ops Datio

Comunidad de conocimiento y práctica donde desplegaremos un cluster de K8s con Kubespray en Openstack

![](https://img.shields.io/github/last-commit/felixrod78/cop_k8s) ![](https://img.shields.io/github/issues-pr/felixrod78/cop_k8s?style=plastic) ![](https://img.shields.io/github/languages/code-size/felixrod78/cop_k8s) ![](https://img.shields.io/badge/CoP-K8s-brightgreen)


### Objetivos

- Creación infraestructura automatizada
- Despliegue básico cluster de K8S con Kubespray

### Requerimientos

> ansible 2.9

>python3 

>pip:
  -kubernetes
  -openshift
  -google.auth


# Creación infra en Openstack

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
# Despliegue cluster K8S con Kubespray

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
# Empezamos a trabajar en el cluster
## Creación namespace  
Creamos un role "deployments" 
Ejecutamos exclusivamente el tag específico de creación de namespace
Usaremos el main.yml para gobernar los despliegues de incrementos del cluster

```bash
ansible-playbook main.yml --tags "namespace" -vvv
```
```bash
kubectl get namespaces
NAME              STATUS   AGE
default           Active   40h
development       Active   11m
kube-node-lease   Active   40h
kube-public       Active   40h
kube-system       Active   40h  
```
