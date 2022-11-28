# ansible-configs

Ansible configs for my personal projects.

## Packages

1. **nginx**: Nginx listens on 80. Current configuration expects Jenkins to be running on 8080.
2. **jenkins**: To configure the pipeline from GitHub.
3. **certbot**: To fetch HTTPS certs.
4. **docker**: To make docker builds inside Jenkins.
5. **rabbitmq**: Experimental purpose.

## How to run a playbook

Regardless of the reusability factor of the roles, most probably you'll use roles from inside a playbook. The following command describes how you can run `docker` playbook.

    ansible-playbook -i inventory playbooks/docker.yml

So you must need to pass the inventory file along with the playbook itself. But there is much more going on here. If you open the docker playbook, you'll find against which host this playbook is going to run. You'll find jenkins. Now, inside the inventory file, you must have a name:host mapping for jenkins.

## Platform Support

Most of the configuration here are tested against Ubuntu 20.04 LTS (arm64).
