# sshd+systemd container
Derived from ideas from @apo, online resources, and @jdeathe. This README is a WIP. The work in <repo>/apo is @apo's original work. The work in <jimconn> is my original work.

# build
docker-compose build cnct-ci

# run
Must use -d and --privileged and --name is suggested. Can use --cap SYSADMIN instead of --privileged. Use --env to manipulate the SSH daemon in the container. Use @jdeathe's sshd container for documentation. 

docker run --privileged -d --name ssh.root --rm jim_cnct-ci:latest

