# Install Docker
#### Mac or Win install [boot2docker](http://boot2docker.io)
1. `boot2docker init`
2. `boot2docker up`
3. `eval "$(boot2docker shellinit)"`

### Linux
1. `apt-get update`
2. `apt-get install linux-image-generic-lts-trusty wget`
3. `reboot`
4. `wget -qO- https://get.docker.com/ | sh`

### Everyone
Verify `docker` is installed correctly
1. `docker run hello-world`
2. `docker pull ubuntu`
