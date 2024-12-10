# Tarefa-Apache-VirtualHost
### Martín Álvarez Comesaña
Obxetivo

O obxetivo desta práctica é integrar un servidor web (Apache) nun entorno Docker, onde tamén hai un servidor DNS funcional. O DNS será responsable de resolver dous dominios que apuntan ao servidor Apache. Usaremos un docker-compose.yml para configurar os contenedores e redes.
docker-compose.yml

Este arquivo de configuración define dous servizos (DNS e WEB) e unha rede (apache_red), asegurando que ambos os contenedores se comuniquen entre si.
```
version: '3'
    services:
    web:
        image: php:7.4-apache
        container_name: apache_server
        ports:
           - "80:80"
        volumes:
          - ./www:/var/www
          - ./confApache:/etc/apache2
        networks:
          apache_red:
            ipv4_address: 172.39.4.2

    dns:
        container_name: dns_server
        image: ubuntu/bind9
        ports:
          - "57:53"
        volumes:
          - ./confDNS/conf:/etc/bind
          - ./confDNS/zonas:/var/lib/bind
    networks:
      apache_red:
        ipv4_address: 172.39.4.3

    networks:
        apache_red:
        driver: bridge
        ipam:
          driver: default
          config:
            - subnet: 172.39.0.0/16
```
Explicación dos servizos:
DNS:
        Usamos a imaxe ubuntu/bind9 para o servidor DNS.
        Mapeará o porto 53 para que o servidor de nomes sexa accesible.
        A configuración do DNS e as zonas están almacenadas en volumes para persistir os cambios.

Web (Apache):
        Utilizamos a imaxe php:7.4-apache para configurar o servidor Apache con soporte para PHP.
        O contedor estará exposto no porto 80 para servir as páxinas web.
        A configuración de Apache e as páxinas web tamén se gardan en volumes.

Configuración de Apache

A configuración de Apache realizarase mediante Virtual Hosts para servir dúas páxinas web diferentes en función do dominio solicitado.
fabulasmaravillosas.conf

Este arquivo define o Virtual Host para fabulasmaravillosas.asircastelao.int.
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasmaravillosas.asircastelao.int
    ServerAlias www.fabulasmaravillosas.asircastelao.int
    DocumentRoot /var/www/fabulasmaravillosas
</VirtualHost>
```
fabulasoscuras.conf

Este arquivo define o Virtual Host para fabulasoscuras.asircastelao.int.
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName fabulasoscuras.asircastelao.int
    ServerAlias www.fabulasoscuras.asircastelao.int
    DocumentRoot /var/www/fabulasoscuras
</VirtualHost>
```
Configuración do DNS
Zonas

O arquivo de zonas define as IPs ás que deben resolverse os dominios. No noso caso, os dominios fabulasmaravillosas.asircastelao.int e fabulasoscuras.asircastelao.int apuntan á IP do servidor Apache.
```
$TTL    604800
@       IN      SOA     ns.asircastelao.int. admin.asircastelao.int. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      ns.asircastelao.int.
ns      IN      A       172.39.4.3
fabulasoscuras       IN      A       172.39.4.2
fabulasmaravillosas  IN      A       172.39.4.2
```
Configuración de BIND
```
En confDNS/conf, temos os seguintes arquivos de configuración de BIND.

    named.conf.local: Define as zonas e os arquivos asociados.

zone "asircastelao.int" {
    type master;
    file "/var/lib/bind/db.asircastelao.int";
    allow-query { any; };
};

    named.conf.options: Configuración adicional de BIND, como os forwarders e a resolución recursiva.

options {
    directory "/var/cache/bind";
    recursion yes;
    allow-query { any; };
    dnssec-validation no;
    forwarders {
        8.8.8.8;  # Google DNS
        1.1.1.1;  # Cloudflare DNS
    };
    listen-on { any; };
    listen-on-v6 { any; };
};
```
Configuración do Sistema

O script restore_systemd-resolved.sh serve para restablecer a configuración do DNS.

### Comprobación

Para comprobar que todo está funcionando correctamente, debemos buscar os dominios fabulasmaravillosas.asircastelao.int e fabulasoscuras.asircastelao.int nun navegador. Estes dominios deberían dirixirse ao servidor Apache e mostrar as páxinas correctas.
