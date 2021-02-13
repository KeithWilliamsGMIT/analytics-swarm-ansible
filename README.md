# Analytics Swarm Ansible

This repository shows how to setup an analytics stack running in Docker Swarm using Ansible to automate the task. This stack consists of the following technologies:

+ [Matomo](https://matomo.org/)
+ [MariaDB](https://mariadb.org/)

The services in this stack are also configured to work with the Traefik reverse proxy. The [proxy-swarm-ansible](https://github.com/KeithWilliamsGMIT/proxy-swarm-ansible) repository can be used to setup Traefik.

## Getting Started

The first step to getting started is to clone the repository:

```bash
git clone https://github.com/KeithWilliamsGMIT/analytics-swarm-ansible.git
```

Next, install Ansible and then we can use two existing Ansible roles to install Docker and initialise a Docker Swarm. These roles are defined in the `requirements.yml` file and will be installed to `~/.ansible/roles/` by default using the below commands:

```bash
cd playbooks/requirements
ansible-galaxy install -r requirements.yml
```

We can install [Docker](https://docs.docker.com/install/) on all the hosts now using the `install-docker.yml` playbook as shown below:

```bash
cd playbooks
ansible-playbook -i ./inventories/localhost install-docker.yml
```

Similarly, to initialise a Docker Swarm between the hosts we can run the `initialize-swarm.yml` playbook. Note that we set extra variables to override some defaults in the playbook. We want to skip everything except the actual initialisation of Docker Swarm.

```bash
ansible-playbook -i ./inventories/localhost initialize-swarm.yml --extra-vars="{'skip_engine': 'True', 'skip_group': 'True', 'skip_docker_py': 'True'}"
```

Finally, we can deploy the analytics stack to Docker Swarm using the `deploy-analytics.yml` playbook.

```bash
ansible-playbook -i ./inventories/localhost deploy-analytics.yml --ask-vault-pass
```

The `--ask-vault-pass` flag will prompt you for the vault password to access sensitive data. A default password used in this case, for demostraion purposes, is `password`.

## Does it work?

Once you followed the above steps we can check if the analytics services are running as expected on the docker manager.

```bash
ubuntu:~/analytics-swarm-ansible/playbooks$ sudo docker service ls
ID             NAME                MODE         REPLICAS   IMAGE                                           PORTS
668jkij274dn   analytics_mariadb   replicated   1/1        mariadb:latest                                  *:7000->80/tcp
o4gqsyz460gd   analytics_matomo    replicated   1/1        matomo:latest
```

We can now login to Matomo by navigating to `http://localhost:7000/` in a browser. We expose Matomo through port `7000` rather than port `80` as it is often taken. There may be some manual setup required after deploying the stack such as connecting to the database and creating an account. Once, this manual setup is complete you will be given a script to add to the application you wish to track.

## Further configuration

There are also a number of Ansible variables that can be overridden. These can be found in the table below:

| Variable Name | Description | Default Value |
|---------------|-------------|---------------|
| temporary_directory | A path to a directory where temporary files such as the compose file can be written to | "/tmp" |
| volume_directory | A path to a directory where service data can be persisted | "/tmp" |
| docker_network_name | The name of the new Docker overlay network | "analytics_network" |
| mariadb_version | The tag of the MariaDB Docker image to use | latest |
| matomo_version | The tag of the Matomo Docker image to use | latest |
| analytics_database_name | The name of the database for analytical data | matomo |
| analytics_database_username | The username to log into the database | matomo |
| analytics_database_password | The password to log into the database | password |
| domain_name | The domain name used to access the services | example.local |

## Using this role

To use this role add the following to your `requirements.yml` file:

```
- src: https://github.com/KeithWilliamsGMIT/analytics-swarm-ansible.git
  version: master
  name: deploy-analytics
```

## Contributing

Any contribution to this repository is appreciated, whether it is a pull request, bug report, feature request, feedback or even starring the repository. Some potential areas that need further refinement are:

+ Hardening of role
+ Publishing role to Ansible Galaxy
+ Documentation

## Conclusion

This repository demonstates how to deploy Matomo in Docker Swarm to provide analytical capabilities. MariaDB is used for the database by default to store the analytical data. After deploying the stack there is some additional manual steps such as configuring Matomo and adding the tracking script to your application. Finally, running all of these commands manually everytime we need to deploy this analytics stack would be time consuming and so Ansible is quite useful in automating this process.