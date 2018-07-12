# lightning-network

Con los canales de pago de Lightning, Bitcoin tiene la oportunidad de demostrar que puede escalar.

El objetivo del tutorial es ir de la nada a tener un nodo de Bitcoin y otro de LN en menos de 24horas. La mayor parte del tiempo se irá en la sincronización de la cadena de bloques de Bitcoin, por lo que es interesante comenzar el proceso antes de ir a la cama y terminarlo el día siguiente.

Vamos a instalar todo lo necesario en una instancia de Docker, por lo que empezaremos por instalar Docker.

Docker soporta sistemas de 64 bits con Linux kernel 3.10+. Comprobamos nuestro sistema con:
```
uname -m 
  x86_64
```
Para comprobar la versión del kernel:
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
Ahora estamos listos para proceder con la instalación del nodo Bitcoin.
Clonamos el repositorio:
```
https://github.com/otsarri/lightning-network.git
```
Construimos **(build)** la imagen de docker:
```
cd lightning-node
docker build . -t otsarri/bitcoind
```
Ejecutamos el nodo **bitcoind**:
```
mkdir -p /bitcoin/mainnet/bitcoind
docker run --name bitcoind_mainnet -d -v /bitcoin/mainnet/bitcoind:/data -p 8333:8333 -p 9735:9735 otsarri/bitcoind:latest
```
Ejecuta el comando **tail** para comprobar el progreso:
```
docker logs bitcoind_mainnet --tail "10"
```
Lo que estamos haciendo aquí es configurar un proceso de daemon **bitcoind (versión 0.16.1)** que se ejecuta en una red local privada de docker. Los datos de blockchain se guardan en /bitcoin/mainnet/bitcoind y exponemos los puertos necesarios para que los nodos bitcoin y lightning escuchen otros nodos (8333 y 9735, respectivamente). Así mismo ocultamos los puertos RPC para que no estén expuestos al público.
Una vez ejecutado el comando **(docker run)**, dependiendo de tu red la sincronización tardará varias horas y tomará alrededor de 180GB de espacio en el disco. Periodicamente puedes utilizar el comando **tail** para comprobar el progreso, una vez que llegue a la fecha de hoy la sincronización será completa.

A continuación vamos a configurar la herramienta de línea de comandos de bitcoin **(bitcoin-cli)**, para ello creamos el siguiente atajo:
```
touch /usr/local/bin/bitcoin-cli
```
En el que incluimos el siguiente comando:
```
docker run --rm --network container:bitcoind_mainnet -v /bitcoin/mainnet/bitcoind:/data otsarri/bitcoind:latest bitcoin-cli "$@"
```
Hacemos del archivo un ejecutable:
```
chmod +x /usr/local/bin/bitcoin-cli
```
Comprobamos la interfaz:
```
bitcoin-cli getnetworkinfo
{
  "version": 160000,
  "subversion": "/Satoshi:0.16.0/",
  "protocolversion": 70015,
  "localservices": "000000000000040c",
  "localrelay": true,
  "timeoffset": -9,
  "networkactive": true,
  "connections": 8,
  "networks": [
    {
      "name": "ipv4",
      "limited": false,
      "reachable": true,
      "proxy": "",
      "proxy_randomize_credentials": false
    },
    {
      "name": "ipv6",
      "limited": false,
      "reachable": true,
      "proxy": "",
      "proxy_randomize_credentials": false
    },
    {
      "name": "onion",
      "limited": true,
      "reachable": false,
      "proxy": "",
      "proxy_randomize_credentials": false
    }
  ],
  "relayfee": 0.00001000,
  "incrementalfee": 0.00001000,
  "localaddresses": [
  ],
  "warnings": ""
}

```
El script contiene un único comando que se ejecuta y luego se limpia así mismo. Hay que tener en cuenta que como hemos ocultado los puertos RPC, es necesario ejecutar la CLI en la misma red que los procesos de docker. Arriba se puede ver mi resultado al ejecutar **bitcoin-cli get networkinfo**
Una vez comprobado que tu nodo está sincronizado con la red (comprueba que el último block en **https://www.blockchain.com/explorer** coincide con el resultado del comando **bitcoin-cli getblockcount**) podemos iniciar el nodo de lightning. El proceso es similar al de **bitcoind** solo que más rapido.
```
root@docker-s-6vcpu-16gb-nyc3-01:~# docker run --rm --name lightning --network container:bitcoind_mainnet -v /scratch/bitcoin/mainnet/bitcoind:/root/.bitcoin -v /scratch/bitcoin/mainnet/clightning:/root/.lightning --entrypoint /usr/bin/lightningd cdecker/lightningd:latest --network=bitcoin --log-level=debug
```
Hay que tener en cuenta que estamos ejecutando el nodo de lightining en la misma interfaz de red de docker donde hemos ejecutado el nodo de Bitcoin, por lo que los clientes de RPC pueden chatear entre sí. Usaremos el mismo truco con la CLI de lightning:
```
touch /usr/local/bin/lightning-cli
```
Incluimos el siguiente comando:
```
#!/usr/bin/env bash
docker run --rm -v /bitcoin/mainnet/clightning:/root/.lightning --entrypoint /usr/bin/lightning-cli cdecker/lightningd:latest "$@"
```
Lo hacemos ejecutable:
```
chmod +x /usr/local/bin/lightning-cli
```
Y ejecutamos:
```
lightning-cli getinfo
{
  "id": "03cbabf3df186c29487fa8da334eef87b82e9744962f7602189d49982e26dc2647", 
  "port": 9735, 
  "address": [
  ], 
  "binding": [
    {
      "type": "ipv6", 
      "address": "::", 
      "port": 9735
    }
  ], 
  "version": "v0.5.2-2016-11-21-2796-g27a186b", 
  "blockheight": 531646, 
  "network": "bitcoin"
}
```
¡Enhorabuena! Tienes un nodo de lightning en marcha... solo queda conectarlo a otros nodos y abrir canales de pago. Esta ha sido la parte más difícil pero seguramente la más gratificante.
