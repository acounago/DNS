## Configura un contenedor con la imagen oficial de bind9 utilizando docker-compose.

Primero creamos la network:

    docker create network bind9_subnet

Para personalizar los parámetros:

        docker network create \
    --driver=bridge \
    --subnet=172.28.0.0/16 \
    --ip-range=172.28.5.0/24 \
    --gateway=172.28.5.254 \
Despues comfiguramos un docker-compose.yml con los siguientes parámetros:
    
    services:
        asir_bind9:
            container_name: asir_DNS1
            image: internetsystemsconsortium/bind9:9.16
            ports:
                - 53:53/tcp
                - 53:53/udp
            networks:
             bind9_subnet:
                 ipv4_address: 172.28.5.3
            volumes:
                 - ./conf:/etc/bind
                 - ./lib:/var/lib/bind
            networks:
                bind9_subnet: 
                external: true

    #Otra forma de hacerlo
    ##networks:
      ##network:
         ##ipam:
         ##config:
          ##subnet: 172.28.0.0/24
          ##gateway: 172.28.0.1

## Configuracion del DNS

Dentro del directorio "DNS" creamos un fichero llamado "named.conf" y encribimos dentro lo siguiente para apuntar a nuestros ficheros:

    include "/etc/bind/named.conf.options";

Después creamos los siguientes ficheros "named.conf.options", "named.conf.default-zones" y "named.conf.local" con las siguientes configuraciones:

"named.conf.options"

    options {
        directory "/var/cache/bind";
        forwarders { 
                8.8.8.8;
                1.1.1.1;
        };
        forward only;

        listen-on { any; };
        listen-on-v6 { any; };
        allow-recursion {
                none;
        };
        allow-query {
                any;
        };
};

named.conf.default-zones

    // prime the server with knowledge of the root servers
    zone "." {
	    type hint;
	    file "/usr/share/dns/root.hints";
    };

    // be authoritative for the localhost forward and reverse zones, and for
    // broadcast zones as per RFC 1912

    zone "localhost" {
	    type master;
	    file "/etc/bind/db.local";
    };

    zone "127.in-addr.arpa" {
	    type master;
	    file "/etc/bind/db.127";
    };

    zone "0.in-addr.arpa" {
	    type master;
	    file "/etc/bind/db.0";
    };

    zone "255.in-addr.arpa" {
	    type master;
	    file "/etc/bind/db.255";
    };

named.conf.local

    zone "asircastelao.int" {
	    type master;
	    file "/var/lib/bind/db.asircastelao.int";
	    allow-query {
		    any;
		};
	};

## Creamos tambien un fichero llamado "zonas" y dentro un fichero llamado "db.asircastelao.int" con la siguiente configuracion:

db.asircastelao.int

    $TTL 38400	; 10 hours 40 minutes
    @		IN SOA	ns.asircastelao.int. some.email.address. (
				10000002   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
    @		IN NS	ns.asircastelao.int.
    ns		IN A 	10.1.0.254
    asir_pDNS	IN A	172.5.5.12
    asir_pDNS2	IN A    172.5.5.13
    alias	IN TXT    mensaje
