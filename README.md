# ELK stack in docker

## Services
- Nginx
- Elasticsearch
- Logstash
- Kibana

## Requirements

- Docker Engine
- Docker Compose
- 1.5 GB of RAM

By default, the stach exposes the following ports:
- 80: Nginx
- 443: Nginx
- 12345: Logstash UDP input

## ELK version

By default the stach uses `7.10.2` version of Elastic stack.<br>
To use a different version, simply change the version number inside the `.env` file.

## Bringing up the stack

Start services using Docker Compose:
```console
$ docker-compose up
```
You can also run all services in the background (detached mode) by adding th `-d` flag to the above command.

## Cleanup

Elasticsearch data is persisted inside a volume by default.

In order to entirely shutdown the stack and remove all persisted data, use the following Docker Compose command:
```console
$ docker-compose down -v
```

## Initial setup

The stack is pre-cofigured with the following **privileged** bootstrap user:
- user: *elastic*
- password: *changeme*

After installtion you should create your user in **Kibana -> Stack Management -> Security -> Users** and generate passwords for **built-in users**.

1. Initialize passwords for built-in users<br>
    ```console
    $ docker-compose exec -T elasticsearch bin/elasticsearch-setup-passwords auto --batch
    ```
    Passwords for all 6 built-in users will be randomly generated.<br>
2. Unset the bootstrap password<br>
    Remove `ELASTIC_PASSWORD` environment variable from the `elasticsearch` service inside the `docker-compose.yml` file. It is only used to initialize the keystore during the initial startup of Elasticsearch.<br>
3. Replace usernames and passwords in configuration files<br>
    `kibana/config/kibana.yml`:
    ```
    elasticsearch.username: kibana_system
    elasticsearch.password: *new password*
    ```
    `logstash/config/logstash.yml`:
    ```
    elasticsearch.username: logstash_system
    elasticsearch.password: *new password*
    ```
    Replace the password for the `elastic` user iside the Logstash pipeline file(`logstash/pipeline/logstash.conf`).<br>
    *Do not use the `logstash_system` user inside the Logstash **pipeline** file, it does not have sufficient permissions to create indices.*<br>
4. Restart Kibana and Logstash to apply changes
    ```console
    $ docker-compose restart kibana logstash
    ```
