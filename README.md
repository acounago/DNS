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