# DNSmasq: DHCP + DNS
Este caso de configuración de servidores DNS no es recursivo, sino **fordward**. Suele usarse en escenarios pequeños. 

El servidor conoce todos los nombres que tiene en **/etc/hosts**, de tal forma que todos los nombres que haya en este fichero del servidor también serán resultos por los clientes (que en realidad no lo resuelve los clientes, sino el servidor). 

Este sistema, además de fordward, es **cache**. Las resoluciones de los nombres guardados en /etc/hosts serñan resultas muy rapidamente, pero no solo si está gurdado en /etc/hosts, sino que se guarda también nombres que estén fuera. Por lo que podemos considerar este sistema como un hosts centralizado. 


## DNSmasq como DNS cache/forward en una red localsshse
El paquete dnsmasq permite poner en marcha un servidor DNS de una forma muy sencilla. Simplemente instalando y arrancando el servicio dnsmasq, sin realizar ningún tipo de configuración adicional, nuestro PC se convertirá en un servidor caché DNS y además, resolverá los nombres que tengamos configurados en el archivo /etc/hosts de nuestro servidor. La resolución funcionará tanto en sentido directo como en sentido inverso.

Queremos instalar un servidor DNS local en nuestra intranet que nos permita gestionar los nombres de las máquinas y recursos de nuestra red. Las características del servidor DNS que queremos instalar son las siguientes:

- Vamos a suponer que tenemos un servidor web que se llame www.iesgn.com y que está en 192.168.1.200 (esto es ficticio).
- Vamos a suponer que tenemos un servidor ftp que se llame ftp.iesgn.com y que está en 192.168.1.201 (esto es ficticio).
- Además queremos nombrar a otras máquinas.

~~~
vagrant@servidor:~$ sudo apt install dnsmasq
~~~

~~~
  GNU nano 3.2                    /etc/hosts

127.0.0.1       localhost
127.0.1.1       servidor        servidor

192.168.1.200 www.iesgn.com
192.168.1.201 ftp.iesgn.com
~~~

Una vez instalado, el paquete, editamos el fichero /etc/dnsmasq.conf y modificamos las siguientes líneas:
- Descomentamos strict-order para que se realicen las peticiones DNS a los servidores que aparecen en el fichero /etc/resolv.conf en el orden en el aparecen.
- Incluimos las interfaces de red que deben aceptar peticiones DNS, descomentando la línea interface por ejemplo: interface=eth0

~~~
strict-order
interface=eth2
~~~


Finalmente reiniciamos el servicio.

1. Configura los clientes para que utilicen el servidor DNS que has instalado. 

Se configura el fichero /etc/resolv.conf del cliente:
~~~
domain gonzalonazareno.org
search gonzalonazareno.org
nameserver 192.168.100.1
~~~


2. Realiza las consultas dig/nslookup desde los clientes preguntando por los siguientes:
- Dirección de www.iesgn.org, ftp.iesgn.org
~~~
vagrant@nodolan1:~$ sudo dig www.iesgn.com

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.iesgn.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52010
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.iesgn.com.			IN	A

;; ANSWER SECTION:
www.iesgn.com.		0	IN	A	192.168.1.200

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Mon Nov 04 08:15:25 GMT 2019
;; MSG SIZE  rcvd: 58
~~~

- La dirección IP de www.josedomingo.org
~~~
vagrant@nodolan1:~$ dig www.josedomingo.org

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.josedomingo.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47381
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 5, ADDITIONAL: 6

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 9e5c220b9422956269102d305dbfde80d90a4d5f394a582f (good)
;; QUESTION SECTION:
;www.josedomingo.org.		IN	A

;; ANSWER SECTION:
www.josedomingo.org.	206	IN	CNAME	playerone.josedomingo.org.
playerone.josedomingo.org. 785	IN	A	137.74.161.90

;; AUTHORITY SECTION:
josedomingo.org.	86285	IN	NS	ns1.cdmon.net.
josedomingo.org.	86285	IN	NS	ns3.cdmon.net.
josedomingo.org.	86285	IN	NS	ns2.cdmon.net.
josedomingo.org.	86285	IN	NS	ns5.cdmondns-01.com.
josedomingo.org.	86285	IN	NS	ns4.cdmondns-01.org.

;; ADDITIONAL SECTION:
ns1.cdmon.net.		29172	IN	A	35.189.106.232
ns2.cdmon.net.		29172	IN	A	35.195.57.29
ns3.cdmon.net.		29172	IN	A	35.157.47.125
ns4.cdmondns-01.org.	45974	IN	A	52.58.66.183
ns5.cdmondns-01.com.	29172	IN	A	52.59.146.62

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Mon Nov 04 08:17:04 GMT 2019
;; MSG SIZE  rcvd: 322
~~~

- El nombre de la ip 192.168.1.200
~~~
vagrant@nodolan1:~$ dig -x 192.168.1.200

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> -x 192.168.1.200
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37916
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;200.1.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
200.1.168.192.in-addr.arpa. 0	IN	PTR	www.iesgn.com.

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Mon Nov 04 08:23:00 GMT 2019
;; MSG SIZE  rcvd: 82
~~~

- ¿Te devuelve el nombre del servidor DNS con autoridad para la zona iesgn.com?
~~~
vagrant@nodolan1:~$ dig ns iesgn.com

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> ns iesgn.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63559
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 350e407524e083d7422e06a75dbfe01178321a6ed6d9c72c (good)
;; QUESTION SECTION:
;iesgn.com.			IN	NS

;; Query time: 1 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Mon Nov 04 08:23:45 GMT 2019
;; MSG SIZE  rcvd: 66
~~~


3. Demuestra que dnsmasq en un servidor web cache
~~~
vagrant@nodolan1:~$ dig www.marca.com

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.marca.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38425
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 4, ADDITIONAL: 5

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: cc669d30626f307f6bbfce1f5dbfe09d9dd3e3b8e9bb86fd (good)
;; QUESTION SECTION:
;www.marca.com.			IN	A

;; ANSWER SECTION:
www.marca.com.		300	IN	CNAME	unidadeditorial.map.fastly.net.
unidadeditorial.map.fastly.net.	30 IN	A	151.101.133.50

;; AUTHORITY SECTION:
fastly.net.		106197	IN	NS	ns2.fastly.net.
fastly.net.		106197	IN	NS	ns1.fastly.net.
fastly.net.		106197	IN	NS	ns3.fastly.net.
fastly.net.		106197	IN	NS	ns4.fastly.net.

;; ADDITIONAL SECTION:
ns1.fastly.net.		106197	IN	A	23.235.32.32
ns2.fastly.net.		106197	IN	A	104.156.80.32
ns3.fastly.net.		106197	IN	A	23.235.36.32
ns4.fastly.net.		106197	IN	A	104.156.84.32

;; Query time: 306 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Mon Nov 04 08:26:05 GMT 2019
;; MSG SIZE  rcvd: 266

vagrant@nodolan1:~$ dig www.marca.com

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> www.marca.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2029
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.marca.com.			IN	A

;; ANSWER SECTION:
www.marca.com.		296	IN	CNAME	unidadeditorial.map.fastly.net.
unidadeditorial.map.fastly.net.	26 IN	A	151.101.133.50

;; Query time: 5 msec
;; SERVER: 192.168.100.1#53(192.168.100.1)
;; WHEN: Mon Nov 04 08:26:10 GMT 2019
;; MSG SIZE  rcvd: 102
~~~


