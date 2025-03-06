# Nextcloud Installation on Synology NAS

This repository contains a step-by-step guide for installing Nextcloud on a Synology NAS using Docker and Portainer. The setup includes configuring Nextcloud with a MariaDB database in Docker containers.

## Prerequisites
- A Synology NAS with Docker and Portainer installed
- Basic knowledge of Docker and SSH access to the NAS
- Docker containers for Nextcloud and MariaDB
- (Optional) Nginx Proxy Manager for SSL and domain management

## Step-by-Step Installation

### 1. Prepare the Synology NAS
Ensure Docker and Portainer are installed and running on your NAS. Access Portainer via its web interface to manage containers.

### 2. Create a MariaDB Container
First, create a MariaDB container that will be used to store the Nextcloud database.

- Pull the official MariaDB image:
  ```bash
  docker pull mariadb

Create a new MariaDB container:

    docker run -d --name mariadb-for-nextcloud \
      -e MYSQL_ROOT_PASSWORD=my-secret-pw \
      -e MYSQL_DATABASE=nextcloud \
      -e MYSQL_USER=nextclouduser \
      -e MYSQL_PASSWORD=nextcloudpass \
      -p 3307:3306 \
      --network bridge \
      mariadb

### 3. Create a Nextcloud Container
Now create the Nextcloud container and link it to the MariaDB database.

Pull the official Nextcloud image:

    docker pull nextcloud

Create a new Nextcloud container:

    docker run -d --name nextcloud \
      -p 8080:80 \
      --network bridge \
      nextcloud

### 4. Connect Nextcloud to MariaDB
In the Nextcloud web interface, configure the database connection:

Database User: nextclouduser
Database Password: nextcloudpass
Database Name: nextcloud
Database Host: Use the IP address of the MariaDB container. For example, 172.18.0.4:3306.
If you don't know the IP address of the MariaDB container, you can find it using:

    docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mariadb-for-nextcloud

### 5. Access Nextcloud
After installation, you can access Nextcloud through your NAS's IP address and the specified port, for example:

    http://your-nas-ip:8080

### 6. (Optional) Secure Nextcloud with SSL using Nginx Proxy Manager
You can add SSL support and domain management by configuring Nginx Proxy Manager in Docker.

## Troubleshooting
If you encounter database connection issues, ensure that the Nextcloud and MariaDB containers are in the same network and that the correct IP address is being used in the Nextcloud configuration.

Use the following commands to verify:

    docker inspect [container-name] | grep -A 10 "Networks"

License
This project is licensed under the MIT License.
