
- [Install docker](#install-docker)
- [Get docker image](#get-docker-image)
  * [Pull from docker hub](#pull-from-docker-hub)
  * [Build by your self](#build-by-your-self)
- [Run](#run)
- [Update dockerhub](#update-dockerhub)

Please follow step2 at https://lomosw.lomorage.com/en/index.html to initialize the systeam and create users once you have lomo-docker setup successfully.

# Install docker

Please follow [instruction](https://docs.docker.com/engine/install/) on docker offical site to install docker

note: If you are using OMSC, you probably need to change "id=osmc" in /etc/os-release to "id=raspbain"

```
sudo apt install -y ca-certificates
sudo update-ca-certificates --fresh
curl -fSLs https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
sudo systemctl start docker
sudo docker info
```

# Get docker image

You can either pull docker image from docker hub, or build yourself.

## Pull from docker hub

If using arm:

```
sudo docker pull lomorage/raspberrypi-lomorage:latest
```

If using arm64:

```
sudo docker pull lomorage/arm64-lomorage:latest
```

If using x86/amd64:

```
sudo docker pull lomorage/amd64-lomorage:latest
```

## Build by your self

```
# build for arm
sudo docker build --build-arg DEBIAN_FRONTEND=noninteractive --build-arg DUMMY=`date +%s` -t lomorage/raspberrypi-lomorage .

# build for arm64
sudo docker build -f Dockerfile.arm64 --build-arg DEBIAN_FRONTEND=noninteractive --build-arg DUMMY=`date +%s` -t lomorage/arm64-lomorage .

# build for x86/amd64
sudo docker build -f Dockerfile.amd64 --build-arg DEBIAN_FRONTEND=noninteractive --build-arg DUMMY=`date +%s` -t lomorage/amd64-lomorage .
```

# Run

You have two options:

## Option 1

use run.sh, download it first:

```
wget https://raw.githubusercontent.com/lomorage/lomo-docker/master/run.sh
```

You can specify the media home directory and lomo directory(**make sure to use absolute directory here**), otherwise it will use the default, you **MUST** specify the host.

```
run.sh [-m {media-dir} -k {backup-dir} -b {lomo-dir} -d -u -p {lomod-port} -i {image-name}] -t vlan-type -s subnet -g gateway -n network-interface -a vlan-address


You can use either use macvlan or ipvlan which makes MDNS service discovery work.
But macvlan and ipvlan are only support on Linux, so if you are on Windows or Mac, you can't use it.

Command line options:
    -m  DIR         path of media directory used for media assets, default to "/media/primary", optional
    -k  DIR         path of media backup directory used for media assets, default to "/media/backup", optional
    -b  DIR         path of lomo directory used for db and log files, default to "/home/jeromy/lomo", optional
    -s  SUBNET      Subnet of the host network(like 192.168.1.0/24), required when using vlan
    -g  GATEWAY     gateway of the host network(like 192.168.1.1), required when using vlan
    -n  NETWORK_INF network interface of the host network(like eth0), required when using vlan
    -t  VLAN_TYPE   vlan type, can be "macvlan" or "ipvlan", required when using vlan
    -a  VLAN_ADDR   vlan address to be used(like 192.168.1.99), required when using vlan
    -p  LOMOD_PORT  lomo-backend service port exposed on host machine, default to "8000", optional
    -i  IMAGE_NAME  docker image name, for example "lomorage/raspberrypi-lomorage:[tag]", default "lomorage/raspberrypi-lomorage:latest", optional
    -e  ENV_FILE    environment variable file passed into container, optional
    -d              Debug mode to run in foreground, default to 0, optional
    -u              Auto upgrade lomorage docker images, default to 0, optional

Examples:
    # assuming your hard drive mounted in /media, like /media/usb0, /media/usb1
    ./run.sh -m /media/usb0 -k /media/usb1 -b /home/pi/lomo -s 192.168.1.0/24 -g 192.168.1.1 -n eth0 -t macvlan -a 192.168.1.99 -u

    # or if you don't use vlan
    ./run.sh -m /media -b /home/pi/lomo -u
```

You can add the command in "/etc/rc.local" before "exit 0" to make it run automatically after system boot.

You can set environment variables in the container by using env file, if you use `-e ENV_FILE`.

For example you can have the following in the env file (check "[lomod.env](lomod.env)" to customize the webpage footer.

```
LOMOW_FOOT_HTML=<a href=https://beian.miit.gov.cn>粤ICP备168xxxxxxx号</a>
```

Or you can disable disk mount monitor is having some problem when monitoring the disk status

```
LOMOD_DISABLE_MOUNT_MONITOR=1
```

## Option 2

You can use docker compose, if you are on OSX or Windows, use "[docker-compose.yml](docker-compose.yml)", if you are on Linux, you can use "[docker-compose.vlan.yml](docker-compose.vlan.yml)" with which MDNS works.
Make sure to modify "[.env](.env)" in your env.

You can remove line `- ${HOME_MEDIA_BAKUP_DIR}:/media/backup` if you don't want to set backup directory.

```
# on OSX or Windows
docker-compose up

# on Linux
sudo docker-compose -f docker-compose.vlan.yml up
```

You can use [environment variables](https://docs.docker.com/compose/environment-variables/#the-env_file-configuration-option) in the container with docker compose as well.

For example you can have the following in the env file to customize the webpage footer.

```
LOMOW_FOOT_HTML="<a href=\"https://beian.miit.gov.cn\">粤ICP备168xxxxxxx号</a>"
```

Or you can disable disk mount monitor is having some problem when monitoring the disk status

```
LOMOD_DISABLE_MOUNT_MONITOR=1
```

# Update dockerhub

Retag and then push:

arm:

```
sudo docker tag lomorage/raspberrypi-lomorage:latest lomorage/raspberrypi-lomorage:latest
sudo docker push lomorage/raspberrypi-lomorage:latest
```

arm64:

```
sudo docker tag lomorage/arm64-lomorage:latest lomorage/arm64-lomorage:latest
sudo docker push lomorage/arm64-lomorage:latest
```

x86/amd64:

```
sudo docker tag lomorage/amd64-lomorage:latest lomorage/amd64-lomorage:latest
sudo docker push lomorage/amd64-lomorage:latest
```
