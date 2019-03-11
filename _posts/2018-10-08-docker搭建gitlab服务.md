---
title: docker搭建gitlab服务
categories:
 - DevOps
tags: DevOps
---

39.108.218.254/Gccf,1234

# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo systemctl restart docker
# Step 5: 配置镜像加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://u4d9eweh.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

# Step 6: 启动gitlab服务
sudo docker run --detach \
    --hostname gitlab.clear.com \
	--env GITLAB_OMNIBUS_CONFIG="external_url 'http://gitlab.clear.com:8929'; gitlab_rails['gitlab_shell_ssh_port'] = 2289; gitlab_rails['lfs_enabled'] = true;" \
    --publish 8929:8929 \
    --publish 2289:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest

# 启动后设置密码
root/Xgw123456
xuguangwu/Xgw123456