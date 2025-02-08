# How to Install Oficial and Custom Apps in Frappe

[Documentation](https://github.com/frappe/frappe_docker/wiki/Frequently-Asked-Questions#frequently-asked-questions)

How to install official or custom apps?
In case of production setup you need to build your custom image for installing apps. This repository only publishes frappe/erpnext image with no additional app. You **CANNOT** `bench get-app` **in running containers.**

To build image refer documentation to build [custom apps](https://github.com/frappe/frappe_docker/blob/main/docs/custom-apps.md). To automate it using CI refer [this post](https://discuss.frappe.io/t/container-builds/99916/2).

In case of development setup, you can do bench get-app as usual.

[Tutorial](https://github.com/frappe/frappe_docker/blob/main/docs/custom-apps.md)

## Execute this steps in Linux

**Executing this steps in Windows or Mac can cause erros, so:**

- Login into your server by ssh
- Clone Github project

```bash
 npm install my-project
 cd my-project
```

- Create a file names `apps.json`

```bash
 nano apps.json
```

- Inside the apps.json file, list all the apps that you want that you image will have, for example:

```bash
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/hrms",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/helpdesk",
    "branch": "main"
  },
  ...
]
```

Check the apps available list in [Frappe Github](https://github.com/frappe) or [Oficial site](https://frappe.io/products) and save the file.

- Generate base64 string from json file:

```bash
 export APPS_JSON_BASE64=$(base64 -w 0 apps.json)
```

- Check if the was successfully generated

```bash
 echo $APPS_JSON_BASE64
```

- Use the following command to decode and save the output into a JSON file named `apps-test-output.json`:

```bash
 echo -n ${APPS_JSON_BASE64} | base64 -d > apps-test-output.json
```

This command will generate an `apps-test-output.json` with the JSON inside just to make sure that the base64 string is right. Run `cat apps-test-output.json` to check the JSON app list.

- Build the image. In this example I will use the [Quick Build Image](https://github.com/frappe/frappe_docker/blob/main/docs/custom-apps.md#quick-build-image), but you can use [Custom build Image](https://github.com/frappe/frappe_docker/blob/main/docs/custom-apps.md#custom-build-image)

```bash
   docker build \
    --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
    --build-arg=FRAPPE_BRANCH=version-15 \
    --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
    --tag=ghcr.io/user/repo/custom:1.0.0 \
    --file=images/layered/Containerfile .
```

- Push image to Docker Hub

```bash
 docker login -u <USERNAME>
 docker push [tag-name]
```

Make sure the `--tag` is valid image name that will be pushed to registry.

# Deploy

#### First of all, make sure to undestand the "Docker Container" and "Caprover" deploy

### Docker

- Replace the **oficial image** to the `pushed-image-name` and add

```bash
 bench --site [site-name] install-app [app-name]
```

for each app that you want to install at `create-site` **service** inside `pwd.yml` file

For example:

```bash
 echo "sites/common_site_config.json found";
 bench new-site --mariadb-user-host-login-scope='%' --admin-password=admin --db-root-username=root --db-root-password=admin --install-app erpnext --set-default frontend;
 bench --site frontend install-app hrms;
 bench --site frontend install-app helpdesk;
 bench --site frontend install-app payments;
 bench --site frontend install-app insights;
 bench --site frontend install-app crm;
 bench --site frontend install-app lms;
```

- Run container

```bash
 docker compose -f pwd.yml up -d
```

- Done!

### Caprover

Replace the oficial image to the pushed-image-name inside `one-click-app-willmagna-image.yaml` file and check the Deploy Caprover repo
