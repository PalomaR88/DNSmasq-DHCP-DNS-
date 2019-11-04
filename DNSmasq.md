# DNSmasq: DHCP + DNS
Este caso de configuración de servidores DNS no es recursivo, sino **fordward**. Suele usarse en escenarios pequeños. 

El servidor conoce todos los nombres que tiene en **/etc/hosts**, de tal forma que todos los nombres que haya en este fichero del servidor también serán resultos por los clientes (que en realidad no lo resuelve los clientes, sino el servidor). 

Este sistema, además de fordward, es **cache**. Las resoluciones de los nombres guardados en /etc/hosts serñan resultas muy rapidamente, pero no solo si está gurdado en /etc/hosts, sino que se guarda también nombres que estén fuera. Por lo que podemos considerar este sistema como un hosts centralizado. 


