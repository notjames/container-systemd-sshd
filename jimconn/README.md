# notes

* Docker compose build

```
docker-compose build cnct-ci
```

* Running the container

  * The `--privileged` parameter is the hammer parameter which can be used to allow `systemd` to work the container properly. One can use `--cap-add SYSADMIN` argument, 
    which is sufficient for `systemd` to work.
  * The `-d` parameter is required to daemonize the container since `systemd` wants to run in the foreground.
  * The `--env` parameter gives the ability to manipulate aspects of the sshd. (TODO : elaborate - use @jdeathe's repo)
  * It's recommended to use `--name <name>` for the container to make it easier to reference.

```
docker run -d --rm --privileged --env='SSH_SUDO="ALL=(ALL) NOPASSWD:ALL"' --name ssh.container --env="SSH_USER=root" <builddir>_<container_name>:latest
```


