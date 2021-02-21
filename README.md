# my_ansible git
su fichero de configuración está en /etc/ansible/hosts


comandos
ansible IP -m ping (-m ejecuta un módulo)
ansible IP -a uptime (-a ejecuta un comanod)

para conectar a otro server generar un certificado
ssh-keygen
se introduce en el server a administrar
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.88.103

verificar
ansible 192.168.88.103 -m ping
en el caso de utilizar un usuario concreto añadir -u
ansible 192.168.88.103 -u user -m ping

existe la posibilidad de ejecutar los comanods sobre todos los equipos añadidos al fichero hotsts

ansible all -m ping

para usar diferentes usuarios en cadas server remoto
en el fichero hosts especificar el susuario
192.168.88.103 ansible_user=root

crear un usuario en un server remoto

ansible all -m user -a "name=test state=present"
[root@manager1 ~]# ansible all -m user -a "name=test state=present"
localhost | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1000,
    "home": "/home/test",
    "name": "test",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1000
}
192.168.88.103 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1000,
    "home": "/home/test",
    "name": "test",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1000
}

para utilizar un usuario que tenga permitido sudo añadir --become o sólo -b
ansible 192.168.88.103 -m user -a "name=test state=present" --become

INVENTARIO:

crear un grupo
[produccion]
192.168.88.103 ansible_user=root
[desarrollo]
192.168.88.104 ansible_user=root

y se puede crear un grupo con subgrupos
[todos:children]
desarrollo 
produccion

se puede definir variables:
ejemplo para añadir --become a las ejecuciones de comando
[todos:vars]
ansible_become=True

en los grupos se pueden utilizar patrones, para evitar especificar una lista de servidores servernode1, servernode2, etc

servernode[1:6].nosolovirtual.com

para usar alias en vez de IPs en el fichero hosts
centos1 ansible_host=192.168.88.103 ansible_user=root

[root@manager1 ~]# ansible centos1 -m ping
centos1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}


sobre grupos, los comandos por defecto se ejecutan en grupos de 5 servidores. 
especificar la opción -f para aplicar a un mayor número de servidores una ejecución simultánea

ansible -f 8 produccion -u test -a id

también se puede limitar un comando a ejecutar sobre servidores croncretos de un grupo añadiendo --limit
en el fichero hosts
[produccion]
centos1 ansible_host=192.168.88.103 ansible_user=root
centos2 ansible_host=192.168.88.104 ansible_user=root

ansible --limit centos1 produccion  -a id


Módulos

módulo setup
ansible  --limit centos1 produccion  -m setup

información de kernel, distro, memoria, etc. Ofrece una información muy detallada.

módulo copy, sirve para copiar ficheros al server remoto
ansible produccion -m copy -a "src=/root/ansible/COPYING dest=/tmp/COPYING"

múdlo de instalación yum/apt

 ansible produccion -m yum -a "name=vim state=present"

para instalar software, 
state = present - instala el software
state = absent - desinstala el software


muestra la lista de servers
 ansible produccion --list-host
  hosts (2):
    centos1
    centos2


comprobación si la tarea puede ser ejecutada, sin realizar cambios en el servidor remoto:
ejemplo:
[root@manager1 ansible]# ansible produccion -m yum -C -a "name=htop state=present"
centos2 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "msg": "No package matching 'htop' found available, installed or updated",
    "rc": 126,
    "results": [
        "No package matching 'htop' found available, installed or updated"
    ]
}
centos1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "installed": [
            "htop"
        ]
    },
    "results": []
}

el server centos2 no tiene el repo epel-release, por ello no localiza el paquete htop


configuración avanzada de ansible. está en el fichero ansible.cfg
a destacar
se puede carmabie el número de forks para que permita la ejecución de más de 5 tareas simultnáenas
 
 
PLAYBOOKS

para aplicar un playbook a varios grupos
hosts: produccion:&grupo2

se ejecutará en los servidores que pertenezcan sólo a ambos grupos, no a los 2 grupos

si un servidores está en 2 grupos, también se puede especificar que se ejecute en un grupo 
con una excepción
hosts: produccion:!grupo2
se ejecutará en el grupo de produccion con excepción de un server que pertenezca a grupo2

hosts: produccion:grupo2
se ejecutara en ambos grupos

en el caso de dudar sobre que servidores se va a aplicar el playbook se puede consultar con --list-hosts
ansible-playbook nombre.yml --list-hosts

también es posible limitar
ansible-playbook --limit 'grupo2' nombre.yml

chequear la sintaxis de un playbook
ansible-playbook --syntax-check nombre.yml

para listar las tareas de un playbook --list-tasks
para ejecutar y confirmar cada tarea paso a paso añadir --step
empezar por una tarea específica --start-at-task="Copiar"  -> se está especificando una tarea no un playbook en el caso que el fichero yml tenga varios
añadir -vvv para tener verbose, tiene 3 niveles añadiendo v

VARIABLES



Módulo de ficheros y OpenSSL
permite trabajar con ficheros , plantillas y directorios:
- blockinfile: Inserta/actualiza/elimna un bloque de texto en un fichero
- copy: copia fichero a remoto
- fetch: copia de servidor remoto a local
- file: establece atributos a ficheros
- lineinfile: asegura que una línea esté dentro de un fichero o reemplaza su contenido
- stat: obtiene información de ficheros o sistema de ficheros
- template: copia y procesa una plantilla a un nodo remoto
- unarchive: extrae ficheros ( después de copiarlo)

openssl_privatekey
openssl_publickey

módulo copy opciones:
backup yes/no
content = "contenido"
force = yes/no
owner, group, mode, src = /tmp/file

módulo stat para obtener información de un fichero

módulo fetch - obtener un fichero del servidor remoto, interesantre para backups

módulo APT
name =nombre:version
upgrade puede actualizar el sistema
upgrade no/yes/safe/full/dist
safe = sólo los paquetes que no sean de sistema
full = todos los paquetes
dist = de distribución


opciones command, con shell si permite utilizar caracteres especiales como && ||
	-chdir: cambia el directorio en el que está trabajando
	-execute: puedes especificar el interprete a usar /bin/bash, etc
 
script - transfiere un script y lo ejecuta
-name:
 script: /path/script.sh
 
raw - envía comandos sin filtrar por SSH
-name: 
 raw: yum install epel-release && yum install nginx
 
expect - ejecuta un comando permitiendo introducir datos interactivamente.

shell
-name: 
 shell: uptime >> file.log
 args:
   chdir: /tmp


wait_for: espera a que un puerto o fichero esté disponible, permite un delay antes de ejecutar una tarea.

Módulos notificaciones: destaco

pushbullet - notificaciones a dispositivos
mqt - para dispositivos IoT
slack - 
SNS - del servicio de AWs
telegram -
sendgrid - envío de mails usando la API
nexmo - envío de SMS
pushover - envío de notificaciones a móviles

Módulo mail:
subject= "mensaje"
host=servidor
port=puerto
secure=try,starttls,always,never
username=
password= 
to= email@nosolovirtual.com cuenta a la que se envía el mensaje
body= "servidor x"

-name: formateo HDD
 filesystem:
   dev: /ect/sdc1
   fstype: xfs
   
-name: deshabilitar firewalld
 firewalld:
   state: disabled

-name: nombre y timezone
 hosts: localhost
 task:
   - hostname: name=nombre_maquina
   - timezone: name="Europe/Madrid"

Conatiners
Necesita instalar docker-py

-pip: name=docker-py state=latest
-name: descarga imagen
 docker_image:
   name: centos
   state: present


