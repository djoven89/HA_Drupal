# HA sobre Drupal

Antes de comenzar a ejecutar constainers, habrá que tener claras las direcciones IP para cada container:

1. *Database ->* 172.20.0.18
2. *Load balancer ->* 172.20.0.19
3. *Drupal ->* 172.20.0.20, 172.20.0.21, 172.20.0.22

También habrá que crear los siguientes volúmenes para que el almacenamiento sea persistente:

- ** Database -> ** db_data **->** /var/lib/mysql
- ** Drupal -> ** drupal_data **->** /var/www/html

**El primer paso será crear una red (no es necesario) , pero así se tiene seguridad en que las direcciones IP no están en uso:**

    docker network create --subnet=172.20.0.0/24 stack_drupal

** Después, se crearán los volúmenes: **

    docker volume create db_data
    docker volume create drupal_data

** MYSQL será el prmero container que se iniciará: **

    docker run --name db -d --restart=always --hostname database -v db_data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=docker -e MYSQL_DATABASE=drupal \
    -e MYSQL_USER=drupal -e MYSQL_PASSWORD=drupal --net stack_drupal \
    --ip 172.20.0.18 mio/mysql


** El siguiente será el balanceador de carga: **

    docker run -d --name loadbalancer -p 9090:80 \
    --network=stack_drupal --ip=172.20.0.19 mis_imagenes/loadbalancer

** Y por último, los Drupales: **

    docker run -d --restart=always --name web_1 -h web1 --link db:database \
    -p 9091:80 --net=stack_drupal --ip=172.20.0.20 -v drupal_data:/var/www/html library/drupal:8

    docker run -d --restart=always --name web_2 -h web2 --link db:database \
    -p 9092:80 --net=stack_drupal --ip=172.20.0.21 -v drupal_data:/var/www/html library/drupal:8

    docker run -d --restart=always --name web_3 -h web3 --link db:database \
    -p 9093:80 --net=stack_drupal --ip=172.20.0.22 -v drupal_data:/var/www/html library/drupal:8

Ya sòlo quedaría acceder desde el navegador web a la dirección IP: **'172.20.0.19'** .
