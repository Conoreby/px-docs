---
layout: page
title: "Create Portworx-based Volume Stacks with Docker Compose"
sidebar: home_sidebar
---
Docker Compose provides a simple powerful way of quickly deploying application stacks. Version 2 of Compose provides the ability to use PX volume drivers in one of the following ways:

* Create new volumes on the fly
* Reuse existing volumes

Here's a sample docker-compose.yml file that brings up a wordpress and mysql stack together:

```yaml
version: '2'
services:

  wordpress:
    image: wordpress
    container_name: wordpress
    working_dir: /var/www/html
    volumes:
      - wpvol:/var/www/html
    links:
      - db:mysql
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_PASSWORD: password
      WORDPRESS_DB_HOST: <IPADDR>:3306

  db:
    image: mysql
    container_name: mysql
    network_mode: "host"
    volumes:
      - sqlvol:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: password

volumes:
  wpvol:
    driver: pxd
    external: false
    driver_opts:
       size: 7
  sqlvol:
    driver: pxd
    external: false
    driver_opts:
       size: 6
```

After `docker-compose up -d`, the following volumes are automatically created through docker-compose, through the existence of `external: false`. You can also create volumes out of band with `pxctl`, and reference them with `external: true`.

```
[root@PX-SM3 ~]# pxctl v l
ID          NAME        SIZE    HA       SHARED STATUS
274113421587995748  wp_wpvol    7.0 GiB 1   no  up - attached on 93a68f30-edcf-4ef4-9122-1b4e0be6ce8b
517652068653682856  wp_sqlvol   6.0 GiB 1   no  up - attached on 93a68f30-edcf-4ef4-9122-1b4e0be6ce8b
```

To bring down the stack, use `docker-compose down --volumes` to remove any volumes that were automatically created.
