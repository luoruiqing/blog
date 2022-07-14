---
title: "Docker + code-server 远程安全开发环境搭建"
subtitle: ""
layout: post
author: "luoruiqing"
catalog : true
header-style: text
tags:
  - Docker
  - Linux
  - VSCode
---



vscode-server 本身需要root权限，所以为了服务器的安全，这里使用 Docker 将开发环境运行在容器内，仅保留部分端口对外， 本例需要实现的基本功能如下：

+ [x] &nbsp;&nbsp; 浏览器开发
+ [x] &nbsp;&nbsp; 访问密码校验
+ [x] &nbsp;&nbsp; zsh + ohmyzsh(powerlevel10k) 环境安装
+ [x] &nbsp;&nbsp; Pyenv + Python3 安装
+ [x] &nbsp;&nbsp; Nvm + node16 安装
+ [x] &nbsp;&nbsp; 容器支持普通用户开发 root用户安装
+ [x] &nbsp;&nbsp; Nginx 服务代理转发(域名开发)


## 镜像选择

[`linuxserver/code-server`](https://hub.docker.com/r/linuxserver/code-server)：[https://hub.docker.com/r/linuxserver/code-server](https://hub.docker.com/r/linuxserver/code-server)


#### Dockerfile


```dockerfile

FROM linuxserver/code-server

USER root
ENV SUDO_USER=root
RUN export SUDO_USER=root
# security.ubuntu.com 默认
# mirrors.ustc.edu.cn 中科大
# mirrors.aliyun.com 阿里云
# mirrors.163.com 163
# mirrors.tuna.tsinghua.edu.cn 清华
ARG APT_SOURCE=mirrors.ustc.edu.cn


RUN cp /etc/apt/sources.list /etc/apt/sources.list.backup && \
    sed -i "s/security.ubuntu.com/${APT_SOURCE}/" /etc/apt/sources.list && \
    sed -i "s/archive.ubuntu.com/${APT_SOURCE}/" /etc/apt/sources.list && \
    sed -i "s/security-cdn.ubuntu.com/${APT_SOURCE}/" /etc/apt/sources.list && \
    apt-get clean

# 刷新包信息
RUN apt-get update

# 基础工具
RUN apt-get install -y \
    zsh \
    vim \
    wget

# 安装中文提示
RUN set -eux && \
    apt-get install -y language-pack-zh-hans && \
    locale-gen zh_CN.UTF-8 && \
    echo "export LC_ALL=zh_CN.UTF-8" >> /etc/profile

# 启动时使用中文
RUN export SUDO_USER=`whoami` && \
    echo "source /etc/profile" >> ~/.bashrc

# 命令行工具 ohmyzsh(powerlevel10k)
RUN set -eux && \
    echo y | sh -c "$(curl -fsSL https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)" && \
    # 国外镜像 git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k && \ 
    git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k && \
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting && \
    echo "ZSH_THEME=\"powerlevel10k/powerlevel10k\"" >> ~/.zshrc && \
    sed -i 's/^plugins=(/plugins=(zsh-autosuggestions zsh-syntax-highlighting z /' ~/.zshrc && \
    chsh -s `which zsh`

# 删除 apt/lists，可以减少最终镜像大小
RUN rm -rf /var/lib/apt/lists/*


# 安装pyenv/Python
# ENV PYENV_ROOT /root/.pyenv
# ENV PATH $PYENV_ROOT/shims:$PYENV_ROOT/bin:$PATH
# 基础依赖
RUN apt install -y gcc \
    make \
    build-essential \
    zlib1g-dev \
    libncurses5-dev \
    libgdbm-dev \
    libnss3-dev \
    libssl-dev \
    libreadline-dev \
    libffi-dev \
    libsqlite3-dev \
    libbz2-dev

# WARNING: The Python lzma extension was not compiled. Missing the lzma lib?
# pyenv + python
RUN curl https://pyenv.run | bash && \
    echo 'export PATH="/config/.pyenv/bin:$PATH"\neval "$(pyenv init -)"\neval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile && \
    source ~/.bash_profile && \
    pyenv update
RUN version="3.6.3"; echo $version; wget "https://mirrors.huaweicloud.com/python/$version/Python-$version.tar.xz" -P ~/.pyenv/cache/;pyenv install $version -v
RUN pyenv rehash
RUN source ~/.bash_profile && \
    python -m venv ./.venv && \
    source ./.venv/bin/activate && \
    pip install -i https://pypi.douban.com/simple --upgrade pip
    

# 安装 NVM
RUN wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash && \
    echo  '# NVM\nexport NVM_DIR="$HOME/.nvm"\n[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvmcurl' >> ~/.bash_profile && \
    source ~/.bash_profile
# 安装 node(16)
RUN nvm install 16


```

## 启动

```sh
docker run -d \ 
    --name=code-server \ 
    -e PUID=1000 \ 
    -e PGID=1000 \ 
    -e TZ=Asia/Shanghai \ 
    -e PASSWORD=vs-dev \ 
    -e DEFAULT_WORKSPACE=/config/workspace \ 
    -e SUDO_PASSWORD=vs-admin \ 
    -p <外部端口>:8443 \ 
    -v <外部存储>:/config \ 
    --restart unless-stopped \ 
    code-server
```


- `PASSWORD` : `vs-dev` : 网页登陆进入的密码
- `TZ` : `Asia/Shanghai` : 服务器的时区
- `SUDO_PASSWORD` : `vs-admin` : 执行root命令时的密码

## 修改root密码

```sh
sudo passwd root
# vs-admin
```

## Nginx代理


```conf
# 用于开启 WebSocket 代理
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
  listen 8086;
  server_name <域名>;


  location / {
            proxy_pass <容器端口>;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
        }

}
```

## 删除容器

```sh
docker stop code-server && docker rm code-server
```


## 删除镜像

```sh
docker rmi code-server
```