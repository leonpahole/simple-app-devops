# Deployment of simple microservice app using Docker swarm, Ansible and nginx-proxy on a single server with automated SSL certificates.

This project deploys and updates an application with a Node.js backend, Postgres database and Angular frontend. It also takes care of all SSL certificates and communication between services. Additionally, Docker secrets are supported for complete security of the application.

Frontend: https://github.com/leonpahole/simple-frontend-angular
Backend: https://github.com/leonpahole/simple-backend-nodejs

# Setting up files

Copy `ansible/production.example` to `ansible/production` and fill out the IP address of the server.
Copy `ansible/group_vars/all.example.yml` to `ansible/group_vars/all.yml` and fill out missing information. You can also use Ansible vault to encrypt this file and push it to version control instead of keeping it only as an example.

Install role dependencies: `ansible-galaxy install -r requirements.yml` (in `ansible` directory).

# Workflow

1. Push properly tagged images of backend or frontend to Docker registry (like Docker Hub) - see bin/release.sh script in above repositories. You can build these images manually or using CI/CD.

2. If you want, test new system locally (see below).

3. Bump version(s) of images in `ansible/files/docker-stack.yml` and commit the change to keep a log.

4. Update the system with Ansible.

5. If you need, run migrations and seeds

# Testing system locally

Use command `docker-compose -f docker-stack.dev.yml up -d`. You will also need to update your `/etc/hosts` with local domains, like so:

```
127.0.0.1       simple.leonpahole.com.local
127.0.0.1       api.simple.leonpahole.com.local
127.0.0.1       db.simple.leonpahole.com.local
```

Keep in mind that this is more or less only for checking if configuration is stable. Frontend won't be able to make requests to your local API as the production url will be different that local one. You could solve this by adding a flag in angular to switch the url to local url (like ?local=yes).

# First time configuration

Before deploying, don't forget to properly change configurations and versions in `docker-stack.yml`. Also don't forget to add DNS records for your domain.

Run `ansible-playbook -i production deploy.yml` when deploying for the first time to install al required packages, initialize swarm, add secrets and deploy the app.

When just updating the app, use `ansible-playbook -i production deploy.yml --tags "update"`.
When you need to update secrets, use `ansible-playbook -i production deploy.yml --tags "secrets"`.

To remove application stack, use `ansible-playbook -i production deploy.yml --tags "remove"`.

# Migrations and seeds

So far the best option to run migrations is to manually log into the server, type `docker ps`, and then run `docker exec <container id> <migration/seed command>`. In my case this command looks so: `docker exec <container id> yarn migrate` and `docker exec <container id> yarn seed`. Keep in mind that each update will change the id of the container so you will have to use `docker ps` again to get the id.
