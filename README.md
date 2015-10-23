# dockervm
Vagrantfile to run docker on a Mac

To use:

```
vagrant up && scp vagrant@192.168.33.2:*.pem $HOME/.docker/
export DOCKER_HOST=tcp://192.168.33.2:2376 DOCKER_TLS_VERIFY="1"
```

You will need the latest docker installed in your Mac following directions in docker.com .
