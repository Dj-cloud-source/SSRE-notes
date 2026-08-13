
zhaohao的脚本
```bash
yum install -y git

git clone https://gitee.com/ShareCoffee/Tools.git

cd Tools/install


./install_docker_ce.sh
```

[官方教程](https://docs.docker.com/engine/install/rhel/#install-using-the-repository) 
```bash

dnf -y install dnf-plugins-core

dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

yum list docker-ce --showduplicates


yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

systemctl start docker
#sudo systemctl enable --now docker


systemctl status docker
docker version
docker info


docker run hello-world



usermod -aG docker $USER_NAME
newgrp docker
```
