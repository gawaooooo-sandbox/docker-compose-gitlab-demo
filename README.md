# Docker で GitLab EE ローカル環境を構築する

## docker-compose を使う

### docker-compose version

```bash
$ docker-compose --version
Docker Compose version v2.0.0-rc.1
```

### docker-compose up

`docker-compose.yml` が置いてあるディレクトリで実行

**image, hostname, GITLAB_OMNIBUS_CONFIG, ports, volumes の指定は作りたい環境に合わせて変更する**

```yml
version: "3"
services:
  web:
    image: "gitlab/gitlab-ee:13.12.4-ee.0"
    restart: always
    hostname: "localhost"
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://localhost:9010'
        gitlab_rails['gitlab_shell_ssh_port'] = 2022
        gitlab_rails['time_zone'] = 'Asia/Tokyo'
    ports:
      - "9010:9010"
      - "443:443"
      - "2022:22"
    volumes:
      - "./config:/etc/gitlab"
      - "./logs:/var/log/gitlab"
      - "./data:/var/opt/gitlab"
```

```bash
$ docker-compose up -d 
```

### docker-compose ps

***starting***

```bash
$ docker-compose ps
NAME                COMMAND             SERVICE             STATUS               PORTS
gitlab-test_web_1   "/assets/wrapper"   web                 running (starting)   :::2022->22/tcp, 0.0.0.0:2022->22/tcp, 80/tcp, :::443->443/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:9010->9010/tcp, :::9010->9010/tcp
```

***healthy***

```bash
$ docker-compose ps
NAME                COMMAND             SERVICE             STATUS              PORTS
gitlab-test_web_1   "/assets/wrapper"   web                 running (healthy)   0.0.0.0:2022->22/tcp, :::2022->22/tcp, 80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp, :::9010->9010/tcp, 0.0.0.0:9010->9010/tcp
```

### docker-compose logs

```bash
$ docker-compose logs -f web # docker-compose logs -f <SERVICE>
```

### ブラウザで確認

`docker-compose ps` で `STATUS` が `running (healthy)` になったら確認

`http://localhost:9010` アクセス --> `http://localhost:9010/users/password/edit?reset_password_token=<token>` にリダイレクトされる

rootユーザのパスワード変更を求められるので変更する


## Docker Engine を使う

### docker version

```bash
$ docker --version
Docker version 20.10.8, build 3967b7d
```

### docker run

**image, hostname,  ports, volumes の指定は作りたい環境に合わせて変更する**

```bash
$ docker run --detach \
--hostname gitlab.example.com \
--publish 443:443 --publish 80:80 \
--name gitlab \
--restart always \
--volume $PWD/config:/etc/gitlab \
--volume $PWD/logs:/var/log/gitlab \
--volume $PWD/data:/var/opt/gitlab \
gitlab/gitlab-ee:13.12.4-ee.0
```

`$PWD` はカレントディレクトリ  
`--volume` は 相対パスが指定できないため

### docker ps

***starting***

```bash
$ docker ps
CONTAINER ID   IMAGE                     COMMAND             CREATED          STATUS                             PORTS                                                                              NAMES
e539348e03c7   gitlab/gitlab-ee:13.12.4-ee.0   "/assets/wrapper"   56 seconds ago   Up 55 seconds (health: starting)   0.0.0.0:80->80/tcp, :::80->80/tcp, 22/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   gitlab
```

***healthy***

```bash
$ docker ps
CONTAINER ID   IMAGE                     COMMAND             CREATED          STATUS                    PORTS                                                                              NAMES
b2d1bf9eecb9   gitlab/gitlab-ee:13.12.4-ee.0   "/assets/wrapper"   16 minutes ago   Up 16 minutes (healthy)   0.0.0.0:80->80/tcp, :::80->80/tcp, 22/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   gitlab
```

### docker logs

```bash
$ docker logs -f gitlab # docker logs -f <container name>
```

### ブラウザで確認

`docker ps` で `STATUS` が `(healthy)` になったら確認

`http://localhost` アクセス --> `http://localhost/users/password/edit?reset_password_token=<token>` にリダイレクトされる

rootユーザのパスワード変更を求められるので変更する

## 参考
[GitLab Docker images](https://docs.gitlab.com/ee/install/docker.html)

[GitLab をインストールしよう! (Docker Image)](https://qiita.com/masakura/items/e29f1dd4794bcaf066ce)

[Dockerで自宅GitLab環境構築~GitLab-CIの初歩まで](https://qiita.com/ryuichi1208/items/1c08523b0ef34d05026f)

[GitLabを自分のパソコンや社内サーバで動かしてみる](https://blackbird-blog.com/gitlab-localinstall)

[docker-composeを使ってGitLab / Runner のローカル環境](https://tsyama.hatenablog.com/entry/docker-local-gitlab-and-runner)

[[Docker] CentOS 7 にインストールした Docker で GitLab 環境を構築する](https://blog.hiros-dot.net/?p=10426)