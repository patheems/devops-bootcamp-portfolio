# Private Docker repository on Nexus

For this part, I reused the Nexus instance from module 6.

## Create Docker repository on Nexus and push to it

### Create the Docker hosted repository
- Created a `docker-hosted` repository in Nexus

![added-docker-repository.png](img/added-docker-repository.png)

- Configured an HTTP connector on port 8083 for the repository. The Docker client talks to this port directly, not to the Nexus UI on 8081

![added-docker-repository-settings.png](img/added-docker-repository-settings.png)

![configured-docker-hosted-port.png](img/configured-docker-hosted-port.png)

- Created an `nx-docker` role with access to the Docker repository and assigned it to my Nexus user

![add-nexus-role-and-assigned-to-user.png](img/add-nexus-role-and-assigned-to-user.png)

- Activated the **Docker Bearer Token Realm**, which `docker login` needs to get a token from Nexus

![added-docker-bearer-to-realms.png](img/added-docker-bearer-to-realms.png)

- Opened port 8083 in the DigitalOcean firewall

![added-port-to-firewall.png](img/added-port-to-firewall.png)

Nexus now listens on both 8081 and 8083:

![docker-registry-port-running.png](img/docker-registry-port-running.png)

### Log in and push
The registry runs on plain HTTP, so I added it to `insecure-registries` in the Docker Desktop engine settings. Otherwise, Docker refuses to connect to it.

![added-insecure-registry-settings.png](img/added-insecure-registry-settings.png)

![succesful-login-docker-registry.png](img/succesful-login-docker-registry.png)

To push the image to Nexus, it needs a tag that includes the registry address:

```bash
docker tag my-app:1.0 <nexus-host>:8083/my-app:1.0
docker push <nexus-host>:8083/my-app:1.0
```

![tagged-my-app.png](img/tagged-my-app.png)

![push-myapp-to-nexus-docker-repository.png](img/push-myapp-to-nexus-docker-repository.png)

![pushed-myapp-to-nexus-docker-repository.png](img/pushed-myapp-to-nexus-docker-repository.png)

I verified the upload with the Nexus REST API:

![access-docker-repository-via-curl.png](img/access-docker-repository-via-curl.png)

### Pull the image back from Nexus

I changed the `my-app` service in the [compose file](../local-dev-docker/course-code/mongo/docker-compose.yaml) to use the image from Nexus. The registry address comes from a `DOCKER_REGISTRY` variable in a local `.env` file, which is not committed:

```yaml
my-app:
  image: ${DOCKER_REGISTRY}/my-app:1.0
```

Running `docker compose up` on my Mac pulled the app image from the private Nexus registry and started it together with MongoDB and Mongo Express. The deployment to the Droplet is described in [local-dev-docker](../local-dev-docker/README.md#deploy-docker-application-on-a-server-with-docker-compose).

![deployed-docker-application-on-server.png](img/deployed-docker-application-on-server.png)
