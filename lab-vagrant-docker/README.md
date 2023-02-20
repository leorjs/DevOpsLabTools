## Se crea un laboratorio con un AdminServer y 3 nodos para poder probar las diferentes tecnologia que el DevOps y/o SRE nos lleva a conocer.

Se instala vagrant > [Installl vagrant](https://developer.hashicorp.com/vagrant/downloads)

Se crea los archivos para inciar vagrant:
hosts > Se encuentra los 4 host virtuales que correra vagrant, con sus nombres e IP correspondiente


Luego de darle vagrant init dentro de la carpeta del proyecto, se creara un archivo por default `Vagrantfile` la cual estaremos modificando para que puedan levantar el AdminServer y los 3 nodo que usaremos para los futuros laboratorios

tips: Para que no de error al correr el vagranfile se debe instalar: 
vagrant plugin install vagrant-vbguest
Se debe ejecutar el powershell como administrador

tips:2 >> Para poder hacer ping desde el AdminServer al resto de los nodos debemos configurar el archivos `hosts` que se encuentra en: `/etc/hosts` 

```
127.0.0.1       localhost

127.0.1.1       vagrant.vm      vagrant

The following lines are desirable for IPv6 capable hosts

::1     localhost ip6-localhost ip6-loopback

ff02::1 ip6-allnodes

ff02::2 ip6-allrouters

  

172.16.1.50 AdminServer

172.16.1.51 node1

172.16.1.52 node2

172.16.1.53 node3
```

Vamos a crear el ssh-keygen la cual arroja lo siguiente:
The key fingerprint is:
SHA256:K40ZGuut6/d13zt9V4zLBVjWocQVYgk7UexW8zm2+xM vagrant@AdminServer

Y estaremos copiando nuestra ssh-key a todos los nodos para que tenga conectividad sin problema

ssh-copy-id node1 && ssh-copy-id node2 && ssh-copy-id node3

Luego con esto podemos hacer ssh sin password

-----
## Creación de Ansible

Ahora vamos a instalar y configurar ansible

Primero vamos a crear nuestro inventario de hosts, en este ejemplo se llama: `ansiblehosts`

````
[AdminServer]
AdminServer

[nodes]
node1
node2
node3
````

Nota: Para la extensión de Ansible en visual studio code, se debe instalar:

1. python
2. ansible-lint >> pip3 install ansible-lint

Vamos a crear el playbook para verificar y ejecutar el docker en los nodos

````
---

- hosts: nodes

  become: true

  tasks:

  - name: ensure docker is installed

    apt:

      name: docker.io

      state: latest

  

  - name: ensure docker-compose is installed

    apt:

      name: docker-compose

      state: latest

  

  - name: Added user to docker group

    user:

      name: vagrant

      groups: docker
````

Pero primero desde el AdminServer vamos a instalar docker
1. $ sudo apt update
2. $ sudo apt-get install ansible -y   (Se instala desde el Adminserver)

Vamos hacer una pequeña prueba de testeo para ver si Ansible llega a los nodos sin problema

ansible nodes -i ansiblehosts -m command -a hostname

````
vagrant@AdminServer:/vagrant$ ansible nodes -i hosts -m command -a hostname
 [WARNING]: Found both group and host with same name: AdminServer

node1 | SUCCESS | rc=0 >>
node1

node2 | SUCCESS | rc=0 >>
node2

node3 | SUCCESS | rc=0 >>
node3

````

Ya que funciono vamos a instalar python en todos los nodos
$ ansible nodes -i hosts -m command -a 'sudo apt-get -y install python-simplejson'

Ahora si vamos a correr nuestro playbook para la instalación de docker

$ ansible-playbook -i hosts -K playbook.yml


### Problemas al ejecutar el playbook

Se presento varios inconveniente al momento de ejecutar el playbook por primera vez, asi que les dejo como quedo la configuración del ansible.cfg

Tambien debemos instalar lo siguiente:
root@AdminServer:~# apt-get install sshpass

Cambiar lo siguiente:
cat /etc/ssh/ssh_config
StrictHostKeyChecking no

Tips & Bonus: Para verificar que la conexión desde el adminserver a los nodes están correcta;

$ ssh -v node1 (Debe entrar directo al node, si pide clave debes volver al punto numero uno en la parte de generación de key)
