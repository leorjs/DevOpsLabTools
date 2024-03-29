## Laboratorio  para poder probar las diferentes tecnologías que el DevOps y/o SRE nos lleva a conocer.

En este Lab que se instalará las siguientes tecnologías:

```` Software
➡️  Vagrant >> Es una herramienta para automatizar la creación y configuración de entornos de desarrollo virtualizados en distintos sistemas operativos.
➡️  Docker >> Es una plataforma de software que permite crear, ejecutar y distribuir aplicaciones en contenedores. Los contenedores permiten empaquetar una aplicación con todas sus dependencias y configuraciones, lo que facilita su implementación y gestión en diferentes entornos.
➡️  Ansible >> Es una herramienta de automatización de TI que permite configurar y administrar sistemas de manera eficiente y repetitiva. Utiliza un lenguaje de scripting simple para definir tareas y procedimientos, lo que facilita la creación de playbooks que automatizan la configuración y administración de infraestructuras.
➡️  Portainer >> Es una plataforma de administración de contenedores que proporciona una interfaz gráfica de usuario para administrar y visualizar los contenedores de Docker. Portainer permite gestionar múltiples hosts de Docker desde una única interfaz, lo que facilita la administración de aplicaciones y servicios en contenedores.
````

Como lo comente antes vamos a trabajar con la *herramienta de vagrant*, la cual nos permite ser flexible para agregar y/o eliminar nodos para nuestro distintos lab. (*Para este caso nuestro laboratorio base contendra 1 AdminServer y 3 nodos*)

Dejo el link oficial para la instalación de vagrant que dependera de tu sistema operativo. >> [Install vagrant](https://developer.hashicorp.com/vagrant/downloads)

Luego de seguir los pasos de la guia oficial, verifiquemos nuestro vagrant instalado. Agregamos el siguiente comando y nos arrojara la versión instalada:
`$ vagrant -v`

Para comenzar la modificación necesaria debemos iniciar vagrant para que se cree el archivo `Vagrantfile`

`$ vagrant init`

Nos genera un archivo por default llamado __*Vagranfile*__ la cual estaremos modificando con el archivo que he dejado en el repositorio


```Tips

Es posible que al momento de levantar nuestro vagrant nos pueda dar error para eso debemos instalar lo siguiente: 
$ vagrant plugin install vagrant-vbguest

-- Si estás en windows se debe ejecutar el powershell como `administrador`
-- Si estás en Linux se debe ejecutar con `sudo`
```

Para continuar debemos iniciar nuestro laboratorio con el siguiente comando:

$ vagrant up >> Con este comando iniciara tanto el AdminServer como los nodos

Ahora si quieres iniciar más adelante solamente un nodo o el AdminServer:
$ vagrant up AdminServer
$ vagrant up Nodo1

Si queremos bajar todo el Lab;
$ vagrant halt

De igual forma dejo un vagrant cheat-sheet que les podrá ayudar:
Ref: [Cheat-sheet](https://gist.github.com/wpscholar/a49594e2e2b918f4d0c4)

``` Tips

Para poder hacer ping desde el AdminServer al resto de los nodos debemos configurar el archivos `hosts` que se encuentra en: >> /etc/hosts (Host del AdminServer, en mi caso uso MobaXterm para entrar a los servidores vía ssh)

Agrego un ejemplo, la cual esta en los archivos de este repositorio

### hosts

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

### SSH-Keygen

🔎Primer paso: Vamos a crear nuestra key plublic/privada:

```` Command
$ ssh-keygen
The key fingerprint is: SHA256:K40ZGuut6/d13zasdasdfaVjWocQ23daf22dexW8zm2+xM vagrant@AdminServer
````

🔎Segundo Paso: Estaremos copiando nuestra ssh-key a todos los nodos para que tenga conectividad sin problema

``ssh-copy-id node1 && ssh-copy-id node2 && ssh-copy-id node3``

Con los pasos anteriores podemos  hacer ssh a los distintos nodos *sin password* entre los mismos nodos y Adminserver.

-----
## Creación de Ansible

Ahora vamos a instalar y configurar ansible.

🔎Primero vamos a crear nuestro inventario de hosts, en este ejemplo se llamara: `ansiblehosts`

````
[AdminServer]
AdminServer

[nodes]
node1
node2
node3
````

Nota: Para la extensión de Ansible en visual studio code que use fueron:

1. python
2. ansible-lint >> pip3 install ansible-lint

🔎Vamos a crear el playbook para verificar y ejecutar el docker en los nodos

````
---

- hosts: nodes
  become: true
  tasks:
  - name:  docker is installed
    apt:
      name: docker.io
      state: latest
      update_cache: yes

  - name:  docker-compose is installed
    apt:
      name: docker-compose
      state: latest
      update_cache: yes 

  - name: Added user to docker group
    user:
      name: vagrant
      groups: docker
````

🔎Pero primero desde el AdminServer vamos a instalar docker

1. $ sudo apt update
2. $ sudo apt-get install ansible -y   (Se instala desde el Adminserver)

🔎Vamos hacer una pequeña prueba de testeo para ver si Ansible llega a los nodos sin problema

$ ansible nodes -i ansiblehosts -m command -a hostname

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


🔎Ya que funciono vamos a instalar python en todos los nodos

$ ansible nodes -i hosts -m command -a 'sudo apt-get -y install python-simplejson'

🔎Ahora si vamos a correr nuestro playbook para la instalación de docker

$ ansible-playbook -i hosts -K playbook.yml


### Problemas al ejecutar el playbook

Se presento varios inconveniente al momento de ejecutar el playbook por primera vez, asi que les dejo como quedo la configuración del ansible.cfg (Se estará subiendo al repositorio directo)


⛩️ ### Importante ### ⛩️
Tambien debemos instalar lo siguiente:
*root@AdminServer:~# apt-get install sshpass*

Cambiar lo siguiente:
*cat /etc/ssh/ssh_config
StrictHostKeyChecking no*

Tips & Bonus: Para verificar que la conexión desde el adminserver a los nodos estan  al 100% correcto con el siguiente comando  lo verificamos:

$ ssh -v node1 (*Debe entrar directo al node, si pide clave debes volver al punto numero uno en la parte de generación de key*)


![ansible](./Images/ansibleok.png)


## Instalación de Portainer en docker

En esta guia de referencia solamente se extra la instalación del Portainer en el AdminServer
Ref: [Guia para la instalación de Portainer](https://www.ionos.es/digitalguide/servidores/configuracion/instalar-portainer-en-docker/)

Creamos el volumen
```volume
docker volume create portainer_data
```

```mixed
docker volume ls
```

```mixed
docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
--restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data \
cr.portainer.io/portainer/portainer-ce:2.9.3

```


Ya debería estar corriendo nuestro docker con Portainer

```
vagrant@AdminServer:~$ docker ps
CONTAINER ID   IMAGE                                          COMMAND        CREATED              STATUS              PORTS                                                                                            NAMES
773f15fef7a1   cr.portainer.io/portainer/portainer-ce:2.9.3   "/portainer"   About a minute ago   Up About a minute   0.0.0.0:8000->8000/tcp, :::8000->8000/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp, 9000/tcp   portainer
vagrant@AdminServer:
```


### Configurar el servidor Portainer en Docker

Si el servidor Portainer se ha instalado correctamente, **la interfaz web de Portainer es accesible en el host local** en el puerto especificado. En nuestro caso, es el puerto 9443. Ahora es el momento de configurar el servidor Portainer.

En nuestro caso tenemos una IP para el AdminServer:

Consola web: https://172.16.1.50:9443/ (Es importante agregarle el *https*)

Cuando entramos por primera vez nos pide crear un usuario de admin y la clave

![](./Images/Dashboardinicio.png)

Dentro del Dashboard de Portainer nos aparecerá lo siguiente la cual estaremos viendo solamente los contenedores locales donde se instalo el Portainer, en este caso *AdminServer*

![](./Images/home1.png)


## Instalación del Agent

Esta vez lo haremos de forma manual ya que solamente contamos con 3 nodos (en caso de tener muchos nodos estare dejando un template de un playbook que pueden usar y modificar)


1. En la parte izquierda nos vamos para *environments* > *add environments* > *Edge Agent* > Name: *node1* 
![](./Images/dashboard3.png)
![](./Images/dashboard4.png)
2. Al ejecutarlo nos dará varias opciones con algunos comando que debemos ejecutar en el node que queremos instalar el agent.
![](./Images/dashboard5.png)
3. En nuestro caso le damos la opción de Docker Standalone dejare un ejemplo:

````
docker run -d \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /var/lib/docker/volumes:/var/lib/docker/volumes \
    -v /:/host \
    -v portainer_agent_data:/data \
    --restart always \
    -e EDGE=1 \
    -e EDGE_ID=fe3e547e-9cfe-408c-88a0-07b3eecdab67 \
    -e EDGE_KEY=aHR0cHM6Ly8xNzIuMTYuMS41MDo5NDQzfDE3Mi4xNi4xLjUwOjgwMDB8YzQ6NzE6N2Y6YzI6YjY6MDk6NzI6NmU6MGU6Y2E6NTI6MDI6NTM6NTQ6NTg6N2V8OA \
    -e CAP_HOST_MANAGEMENT=1 \
    -e EDGE_INSECURE_POLL=1 \
    --name portainer_edge_agent \
    portainer/agent:2.9.3
````

4. Ahora entraremos a nuestros node4 y ejecutaremos ese bloque de comando, una vez finalizado vamos al Home de nuestro Portainer
![](./Images/node3.png)
5. Y allí ya observaremos que tendremos nuestro node conectado a nuestro Portainer
![](./Images/homenodes.png)

