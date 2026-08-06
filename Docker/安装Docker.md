
zhaohao的脚本
```bash
yum install -y git

git clone https://gitee.com/ShareCoffee/Tools.git

cd Tools/install

cd Tools/install

./install_docker_ce.sh
```

[官方教程](https://docs.docker.com/engine/install/rhel/#install-using-the-repository) 
```bash

sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo


sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl start docker
#sudo systemctl enable --now docker

sudo docker run hello-world



sudo usermod -aG docker $USER_NAME
newgrp docker


```
