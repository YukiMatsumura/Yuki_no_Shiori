### Update Jenkins2

Dockerから起動したJenkinsにソフトウェアアップデートがあった場合にそれを適用する方法です.  

Jenkinsのアップデート情報はJenkins管理画面で通知されて`jenkins.war`のダウンロードリンクが示されます.  
サーバ上で`jenkins.war`を指定してJenkinsサーバを起動している場合はこれを置き換えるだけで済みますが, Jenkins公式の[Docker image](https://hub.docker.com/r/library/jenkins/)を使ってコンテナ上にJenkinsサーバを起動している場合は違うアプローチが必要です.  

今回のJenkins on Containerは次の内容で起動しています.  

```bash
// jenkins用のData Volume Containerを用意
docker run -v /var/jenkins_home --name j_storage ubuntu /bin/bash

// jenkins user用にディレクトリのアクセス権限を設定
docker exec j_storage chown 1000 /var/jenkins_home

// jenkinsコンテナをData Volume Container指定して起動
docker run --volumes-from j_storage -p 8080:8080 -p 50000:50000 jenkins:2.0
```

[Jenkinsコンテナの起動方法は何種類かあります](http://yuki312.blogspot.jp/2016/05/get-started-jenkins2.html).  

 - コンテナ内にデータを格納する方法
 - ホスト側のディレクトリをコンテナの`/var/jenkins_home`にマウントする方法
 - Data Volume Container（`/var/jenkins_home`）をマウントする方法

そして, コンテナにあるJenkinsのバージョンをアップデートする方法も何種類かあります.  

 - 直接`jenkins.war`を書き換える
 - 新しいJenkinsバージョンに対応したDocker imageをpullする

コンテナの思想を考えると推奨されるのは後者です. 前者だとコンテナのポータビリティを活かせません. 

```bash
// 新しいバージョンのjenkins docker imageを取得
docker pull jenkins:2.3

// 新しいバージョンのjenkins containerを起動
// 現行のjenkinsで使用していたData Volumeを指定すること
docker run --name jenkins2.3 --volumes-from j_storage -p 8080:8080 -p 50000:50000 jenkins:2.3
```

Jenkinsのバージョンアップで注意することは, Jenkinsのコンテナを起動する際にコンテナ内にデータを保存する方法だと, 新しいDocker imageでも使えるようにデータを引き継ぐ必要があるということです.  

```bash
docker cp $CONTAINER_ID:/var/jenkins_home
```

ホストディレクトリのマウントやData Volume Containerでデータ部を外からマウントしている場合は, 新しいJenkins バージョンのコンテナを起動する際にそれをマウントするだけで済みます.  

Jenkinsの起動についてはこちらもあわせてお読みください.  
[Yukiの枝折 - Get Started Jenkins2](http://yuki312.blogspot.jp/2016/05/get-started-jenkins2.html)  

以上です.  