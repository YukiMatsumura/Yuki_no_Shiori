## Get Started Jenkins2

### Docker Image

Jenkins2.0のOfficial docker image(LTS/安定版)がDocker Hubで公開されています.  

 - [Docker Hub - Jenkins](https://hub.docker.com/_/jenkins/)
 - [Official Jenkins LTS docker image](https://jenkins.io/blog/2014/08/12/official-jenkins-lts-docker-image/)

Weekly buildベースのDocker imageも公開されていますが, 今回はLTS版のDocker Imageを使ってDocker + Jenkins2.0な環境を構築します(構築はMacOS上で実施しています).  

 - [Docker Hub - Jenkinsci/jenkins](https://hub.docker.com/r/jenkinsci/jenkins/)

JenkinsのDocker imageはGitHub上のレポジトリで管理されています.  

 - [GitHub - jenkinsci/docker](https://github.com/jenkinsci/docker)

本稿はすでにDocker環境を構築できており, 使用できる状態にあることを前提に進め, Docker環境構築手順については省略します.  


### docker pull

docker pullでJenkins2のdocker imageを取得しましょう. 現時点で公開されている最新の[tag](https://hub.docker.com/r/library/jenkins/tags/)を指定します.  

```shell
docker pull jenkins:2.0
```

上記はJenkins2.0が最新であった場合の例です.  
下記のようなメッセージが表示されて, 正しくDocker imageをpullできたことを確認します.  

```shell
Digest: sha256:e6f668256b6a048e7041d0bb636425fdd45f50559f23633cb94a9baea59578ee
Status: Downloaded newer image for jenkins:2.0
```

Docker image取得後にJenkinsのソフトウェアバージョンをアップデートする方法は別稿に記載します.  
これでJenkinsを実行できる環境は整いました. Dockerを使えば手軽に環境構築できますね.  


### docker run

JenkinsのDocker imageを起動する方法はいくつかありますが,  
まずはJenkinsのコンテナにある2つの重要なPortを確認しておきます.  

8080:  
Jenkins serverのListen portです. 

50000:  
JenkinsのSlave server用Portです. Slave agent接続時に使用されます.  

`docker run`で起動する際にはこれら2つのPortのポートフォワーディングを指定します.  

```shell
docker run -p 8080:8080 -p 50000:50000 jenkins:2.0
```

デフォルトのSlave agent portを50000から変更したい場合は環境変数`JENKINS_SLAVE_AGENT_PORT`を指定します.  

```shell
FROM jenkins:2.0
ENV JENKINS_SLAVE_AGENT_PORT 50001
```

または`run`コマンドの引数にこれを指定できます.  

```shell
docker run -e JENKINS_SLAVE_AGENT_PORT 50001 ...
```

これでJenkins serverを起動することができました


### run with Host Directory

先ほどの方法でJenkinsを起動するとデータがコンテナ内に保存されます.  
つまり, Jenkinsのデータは永続化されず, コンテナを破棄するとデータは失われます.  
もしコンテナを破棄してデータを引き継ぎたい場合やバックアップしたい場合は,  一手間必要です.  

```bash
docker cp $CONTAINER_ID:/var/jenkins_home
```

Jenkinsのデータは`/var/jenkins_home`に作成されます.  

```bash
# Jenkins home directory is a volume, so configuration and build history 
# can be persisted and survive image upgrades
VOLUME /var/jenkins_home
```

そこで, ホストのディレクトリをJenkins コンテナの`/var/jenkins_home`にマウントする方法があります.  

```shell
docker run -p 8080:8080 -p 50000:50000 -v </host/volume>:/var/jenkins_home jenkins
```

これでコンテナ破棄でデータが消えることはありませんし, バックアップも容易です.  


### run with Data Volume Container

ホストのディレクトリをマウントする方法だとコンテナの旨味を活かしきれていません. ホストのボリュームに依存するとコンテナのポータビリティ性を欠いてしまいます. 
[Data Volume Container](https://docs.docker.com/engine/userguide/containers/dockervolumes/)を使ってこの課題を解決していきます.  

まず, Data Volume Container（`j_storage`）を作成することからはじめます.  

```bash
docker run -v /var/jenkins_home --name j_storage ubuntu /bin/bash
```

`-v`でData Volumeのパスに`/var/jenkins_home`を指定して作成します.  
この状態で作成されている`jenkins_home`フォルダのアクセス権限は次の状態です.  

```bash
drwxr-xr-x 14 root root  4096 May 14 15:26 jenkins_home
```

これではJenkinsユーザに書き込み権限がないのでJenkinsコンテナを起動することができません.  

```bash
touch: cannot touch ‘/var/jenkins_home/copy_reference_file.log’: Permission denied
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
```

Data Volume Container（`j_storage`）の`/var/jenkins_home`のアクセス権限を変更します.  

```bash
docker exec j_storage chown 1000 /var/jenkins_home
```

Jenkinsユーザのuidは`1000`で定義されていますので`chown`でそれを指定します.  

```bash
ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000

# Jenkins is run with user `jenkins`, uid = 1000
# If you bind mount a volume from the host or a data container, 
# ensure you use the same uid
RUN groupadd -g ${gid} ${group} \
    && useradd -d "$JENKINS_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user}
```

設定できたらData Volumeを指定してJenkinsサーバを起動しましょう.  

```bash
docker run --volumes-from j_storage -p 8080:8080 -p 50000:50000 jenkins:2.0
```

これでJenkinsのデータについてもポータビリティを確保できました.  

以上です.  
