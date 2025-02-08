# Deploy Docker Container

[Oficial Documentation](https://github.com/frappe/frappe_docker)

- Clone the repo

```bash
 git clone https://github.com/frappe/frappe_docker
 cd frappe_docker
```

- Then run

```bash
 docker compose -f pwd.yml up -d
```

See the site been create in create-site container

```base
 docker logs <CONTAINER_ID> -f
```

- Done!
