# denv (Docker Environment)

This is my development environment which build with docker. 
This build is target MacOs (cuz I'm using MacOS for now)
Some limitations:
- It use Docker Volume instead of mounting directory from host to docker. (due to mounting directory on MacOS has slow speed than Linux)
- Persistent data like source code, database, config will store on volume instead of your machine. It will difficult for you to backup or sharing them. 
- It use Visual Studio Code to edit direct persistent data on docker volume. 
- All docker images have already setup for my current job. (I'm Magento dev)

## Step by step guide
1. Clone this repository 
```
git clone https://github.com/ndquocphong/denv.git denv
```
2. Create your project docker-compose.yml file
```
cd denv
touch docker-compose.magento2.yaml 
```
4. Create a external volume which use to store persitent data
```
docker volume create denv-data
```
3. Add code-server-3.5.0 as first service into docker-compose.magento2.yaml.
```
version: "3"
services:
  code-server-3.5.0:
    image: phongndq/code-server:3.5.0
    ports:
      - 8080:8080
    volumes:
       - denv-data:/home/www-data
    user: "www-data:www-data"
volumes:
  denv-data:
    external: true
```
4. Create and start services from docker-compose 
```
docker-compose -f docker-compose.magento2.yaml up -d
```
- Navigate url http://127.0.0.1:8080 to access code-server-3.5.0 service. By default, it needs password to access Visual Studio Code screen. 
- Execute `bash` mode into `code-server-3.5.0` service by command `docker-compose -f docker-compose.magento2.yaml exec code-server-3.5.0 bash` 
- Into `bash mode`, execute command `cat /home/www-data/.config/code-server/config.yaml` to read config file. The content is below, copy `password` then use it to access Visual Studio Code screen
```
bind-addr: 127.0.0.1:8080
auth: password
password: 814e2c482f3a95fd7338835c
cert: false
```
- To change the password, use VSCode to modify `/home/www-data/.config/code-server/config.yaml`. Navigate url http://127.0.0.1:8080/?folder=/home/www-data , edit file `.config/code-server/config.yaml`, change password by your password
- Restart `code-server-3.5.0` service to see your changes applied. 
```
docker-compose -f docker-compose.magento2.yaml stop code-server-3.5.0
docker-compose -f docker-compose.magento2.yaml start code-server-3.5.0
```
- Try access http://127.0.0.1:8080/?folder=/home/www-data with your new password. 
5. Add php-apache-7.4 service. 
```
version: "3"
services:
  code-server-3.5.0:
    image: phongndq/code-server:3.5.0
    ports:
      - 8080:8080
    volumes:
       - denv-data:/home/www-data
    user: "www-data:www-data"
  php-7.4-apache:
    image: phongndq/php:7.4-apache
    ports:
      - 8074:80
      - 44374:443
      - "2274:22"
    volumes:
      - denv-data:/home/www-data
    command: bash -c "service ssh start && service apache2 reload & apache2-foreground"
volumes:
  denv-data:
    external: true
```
6. Create and start all services again. 
```
docker-compose -f docker-compose.magento2.yaml up -d
```
- Navigate http://127.0.0.1:8074/ to check `php-7.4-apache` service works as expected
- `php-7.4-apache` service was setup with virtual host config. There are default virtual host which point to /home/www-data directory in `php-7.4-apache` container. 
- Execute command `sudo vi /etc/hosts` to modify your `/etc/hosts`. Add host.local 
```

```
