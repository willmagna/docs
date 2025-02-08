# Caprover Frappe/ERPNext Deploy Instructions

## With Oficial Image

- Access [frappe/docker](https://github.com/frappe/frappe_docker/tree/main) and copy the `pwd.yml` file
- Paste it to a blank file
- Look for the mariadb service and modify the service to:

```bash
  db:
    image: mariadb:11.4.5
    deploy:
      restart_policy:
        condition: on-failure
    environment:
      MYSQL_ROOT_PASSWORD: $$MYSQL_ROOT_PASSWORD
      MARIADB_ROOT_PASSWORD: $$MYSQL_ROOT_PASSWORD
    ports:
      - "3306:3306"
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - frappe_network
```

For a better comprehension you can check the `one-click-app-reference.yaml` file

- After that, modify the file to ne use in One-Click App: [Caprover One-Click Apps](https://github.com/caprover/one-click-apps)
- Go to Caprover Interface, click on button **On-Click Apps/Databases** and type TEMPLATE on search bar
- Paste the modified file to the yaml code area in Caprover and click in **Next** button
- After finished entre into the caprover server by ssh and go to repository:

```bash
  cd var/lib/docker/volumes/captain--sites/\_data/
```

- Edit the `common_site_config.json` by executing the command

```bash
  nano common_site_config.json
```

- Paste this JSON:

```bash
{
 "db_host": "srv-captain--db",
 "db_port": "3306",
 "redis_cache": "redis://srv-captain--redis-cache:6379",
 "redis_queue": "redis://srv-captain--redis-queue:6379",
 "redis_socketio": "redis://srv-captain--redis-socketio:6379",
 "auto_update": false,
 "disable_website_cache": true,
 "domains": ["domain"],
 "socketio_port": "9000"
}
```

into the `common_site_config.json` and save it

- Return to caprover and correct all the env variables to the respective containers name use.

#### Now lets install the site

- Get inside the backend container by executing the command:

```bash
  docker exec -it <BACKEND_CONTAINER ID> /bin/bash
```

**There is 3 ways to install the site:**

- Oficial way

```bash
  bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext --set-default frontend;
```

- Second Way

```bench
bench new-site <nome do site> --mariadb-root-password=admin --admin-password=admin --install-app erpnext
```

- Third way

```bench
  bench new-site --admin-password=admin --mariadb-root-password=admin [frontend-caprover-container-name]
  bench --site [frontend-caprover-container-name] install-app erpnext;
```

- Done. For check if the site is indeed install you can execute the command:

```bash
 ~/frappe-bench$ ls ./sites/
```

If the return its like this:

```bash
  apps.json  apps.txt  assets  common_site_config.json [frontend-caprover-container-name]
```

your site is installed.

## With Custom Image

- Access your Caprover server
- Click on **One-Click Apps/Database** button
- Search for: **TEMPLATE**
- Click it
- Copy all the contents inside `one-click-app-willmagna-image.yaml` file
- Paste it on the CapRover **One Click Apps**
- Click on **Next**
- After finished the process in Caprover, **login into your server terminal via SSH**
- Go to the repository: `cd var/lib/docker/volumes/captain--sites/\_data`
- Execute the command:

```bash
 nano nano common_site_config.json
```

- Paste this JSON:

```bash
 {
  "db_host": "<CAPROVER_CONTAINER_NAME>-db",
  "db_port": "3306",
  "redis_cache": "redis://<CAPROVER_CONTAINER_NAME>-redis-cache:6379",
  "redis_queue": "redis://<CAPROVER_CONTAINER_NAME>-redis-queue:6379",
  "redis_socketio": "redis://<CAPROVER_CONTAINER_NAME>-redis-socketio:6379",
  "auto_update": false,
  "disable_website_cache": true,
  "domains": ["domain"],
  "socketio_port": "9000"
}
```

into the `common_site_config.json` and save it

- Go to Caprover's page
- Go to **websocket container**
- Go to deployment tab and check if the logs shows this message:

```bahs
 Realtime service listening on: 9000
```

It means that the JSON file worked has been imported correctly

- Now return to the **ssh terminal**
- List all the docker containers

```bash
  docker ps
```

- Look for the **backend container** CONTAINER_ID
- Get into the container by executing the followin command:

```bash
  docker exec -it <CONTAINER_ID> /bin/bash
```

- Let's install Frappe Site by executing the following command:

```bash
  bench new-site --admin-password=admin --mariadb-root-password=admin [frontend-caprover-container-name]
```

- Now install the ERPNext App inside the site:

```bash
  bench --site [frontend-caprover-container-name] install-app erpnext
```

#### If the One-Click App file (docker componse file) is set with a custom image, continue:

- Verify all the apps that are available to install inside the site by executing the following command:

```bash
  ls ./apps/
```

- Look: `erpnext` and `frappe` are already installed.
- If there is more apps then `erpnext` and `frappe` you can install by executing the following code:

```bash
  bench --site [frontend-caprover-container-name] install-app [app-name]
```

_Look you need to execute this command for each app, example:_

```bash
  bench --site [frontend-caprover-container-name] install-app crm;
  bench --site [frontend-caprover-container-name] install-app helpdesk;
  bench --site [frontend-caprover-container-name] install-app hrms;
  bench --site [frontend-caprover-container-name] install-app insights;
  bench --site [frontend-caprover-container-name] install-app lms;
  bench --site [frontend-caprover-container-name] install-app payments;
```
