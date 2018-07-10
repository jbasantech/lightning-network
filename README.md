# lightning-network

Con los canales de pago de Lightning, Bitcoin tiene la oportunidad de demostrar que puede escalar.

El objetivo del tutorial es ir de la nada a tener un nodo de Bitcoin y otro de LN en menos de 24horas. La mayor parte del tiempo se irá en la sincronización de la cadena de bloques de Bitcoin, por lo que es interesante comezar el proceso antes de ir a la cama y terminarlo el día siguiente.

Vamos a instalar todo lo necesario en una instancia de Docker, por lo que empezaremos por instalar Docker.

Docker soporta sistemas de 64 bits con Linux kernel 3.10+. Comprobamos nuestro sistema con:
```
uname -m 
  x86_64
```
Versión del kernel:
```
uname -r
  Linux jb-system 4.4.0-124-generic #148-Ubuntu SMP Wed May 2 13:00:18 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
Podemos utilizar el siguiente comando para actualizar nuestro kernel a la última versión:
```
sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade
```
Mi sistema es Linux Mint 18.3 y utilizo este script **(docker-instalación.sh)** para la instalación de Docker-ce:
```
#!/usr/bin/env bash

#https://docs.docker.com/install/linux/docker-ce/ubuntu/
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"
sudo apt-get update
sudo apt-get install docker-ce

# https://docs.docker.com/compose/install/
sudo curl -L https://github.com/docker/compose/releases/download/1.20.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# https://docs.docker.com/install/linux/linux-postinstall/
sudo groupadd docker
sudo usermod -aG docker $USER
```
Si todo ha ido bien puedes comprobar la versión de Docker:
```
docker version
```
Más información con:
```
docker info
```
Para comprobar si la instalación funciona Docker provee de una imagen muy simple llamada hello-world:
```
docker run hello-world
```
Ahora estamos listos para proceder con la instalación del nodo Bitcoin:
