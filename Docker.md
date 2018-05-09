# Docker 學習筆記

## Docker 是什麼？

container，在作業系統層，做虛擬化的動作像是一個獨立開來的環境，對作業系統上的資源做切割然後利用，<br>
類似虛擬機，但是 containers 共用 kernel ，沒有自己的作業系統；<br>
虛擬機則有，並由 hypervisor 來監督，兩者開機時間差很多。

## 需求性

* 安全性：環境隔離，e.g 網路服務如果被入侵，container 就可以當作外部多出來的一層保護

## 特性

* One process in one container
* Data in the container would not be preserved：container 資料會在隨著停止運作而消失；<br>
若需要儲存，要透過第3方，如 [Volumes Component](https://kubernetes.io/docs/concepts/storage/volumes/) 的服務。

## 平台需求（適用於此筆記）

Linux 系統 or 在 https://labs.play-with-docker.com/ 試用

## 安裝與準備

將自己加入 docker 使用者群組，之後使用 docker 不用一直加 `sudo`

```shell
sudo apt-get install docker.io -y
# or curl -L get.doce  -> Adding group `docker'

sudo groupadd docker
# sudo adduser <userName> <group> or sudo usermod -G <group> <userName>
sudo adduser anntsai docker
```

## 指令

```shell
#### docker  system ####
docker info
docker version
docker --version
docker stats

#### container & images ####
docker run <container_name/id>
docker ps [-a]
docker images 
docker inspect
docker start
docker restart
docker stop <container name/id>
docker kill <container> # when no response
docker pause
docker unpause

# online: docker pull <what_you_want>:<ver>
docker pull ubuntu:16.04 # 拉線上的 image

# offline: search images
docker search [--filter "is-official=true"] ubuntu

# run a container from the image (will pull if images isn't in local)
# -p publish ports <external_host_port>/udp?:<internal_container_mapped_port>
# -i enable interactive mode
# -t enable terminal
# -d daemon mode 背景執行
# -v mount [external-folder-path]:[external-folder-path] 目錄共享
# -e entrypoint 類似 COMMAND ，只是在一開始就會被執行
# --name 為 container 自命名
# [which_bash]
# docker run [options] _image_:_tag_ [commandOnBash]
docker run -it [--name ubuntu1704] ubuntu:17.04 bash # 如果系統現在沒有那個 image ，會幫拉 interact
docker run --rm hello-world # don't enter `ps`
docker run -it -p 1234:80 --rm 
docker run -d -p 80:80 --name nginx -v /var/www:/usr/share/nginx/html nginx:alpine # 機器的port:docker的port

# docker exec [options] container_name/ID [commandOnBash]
docker exec -it nginx nginx -v 
docker exec -it nginx sh
docker exec -it nginx vi /etc/nginx/nginx.conf

# update https://docs.docker.com/engine/reference/commandline/update/ 更新已經建立的 container 參數
docker update 

# remove 
docker rm ID # remove useless container，跟VM 不一樣，通常事情做完，就會砍掉了
docker rmi _withTag/ID_[-f] # remove images

# log
docker logs <container>

# tag
docker tag <container name/id> <account>/<reponame>:<version>

# rename
docker rename <old_container_name> <new_name>

# between external & internal
docker cp /path/to/[file1] DOCKER_ID:[/]path/to/[file2] # from ex to in
docker cp DOCKER_ID:[/]path/to/[file2] /path/to/[file1] # from in to ex

hostname _hostMacheineName_ --restart always # 重要服務需要隨時可以被存取

#docker commit
docker run --hostname gitlab.example --restart always -d -p 443:443 -p 80:80 -p 2222:22 --name gitlab -v /srv/gitlab/config:/etc/gitlab -v /srv/gitlab/logs:/var/log/gitlab -v /srv/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
docker logs -f 

docker run --restart always -v /srv/gitlab/config:/etc/gitlab -v /srv/gitlab/logs:/var/log/gitlab -v /srv/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce

docker run -it -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix ubuntu-16.04-firefox
```

## 快速建置 via Dockerfile

```dockerfile
FROM <base_image>:<ver>
# 讓 port 可以從 Docker 容器外部存取
[EXPOSE <internal_port>]
# 設定（建立）工作目錄
[WORKDIR /dir]
# 複製目前目錄下的內容，放進 Docker 容器中的 ...目錄
[ADD . /dir]
# 定義環境變數
[ENV NAME World]
# 紀錄 maintainer 資訊
[MAINTAINER maintainer_name <mail_info>]
[ENTRYPOINT ["point"]]
RUN <commands_in_termainl>
[LABEL <key>=<value>] # e.g. maintainer="moby-dock@example.com"
# 當 Docker 容器啟動時，自動執行 ...
CMD ["bash"]   
```

## Reference

1. https://blog.gtwang.org/virtualization/docker-basic-tutorial/
2. https://joshhu.gitbooks.io/dockercommands/content/Containers/ContainersBasic.html

