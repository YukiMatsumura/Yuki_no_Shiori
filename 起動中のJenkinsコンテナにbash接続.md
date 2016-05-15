Jenkins2をDocker imageから起動してコンテナ運用している時, そのコンテナにアタッチしてbash操作したい場合があります.  

次のようにJenkinsコンテナを起動します.  

```bash
docker run --name jenkins2.3 --volumes-from j_storage -p 8080:8080 -p 50000:50000 jenkins:2.3
```

`docker ps`でjenkins2.3が起動していることを確認し, 次のコマンドでコンテナに接続しましょう.  bashプロンプトが表示されたら成功です.  

```bash
docker exec -it <JENKINS CONTAINER NAME> bash
```

### docker attach

`docker attach`でこのコンテナにbash接続を試みますがプロンプトは現れません.  

```bash
docker attach jenkins2.3
```

[JenkinsのDockerfile](https://github.com/jenkinsci/docker/blob/master/Dockerfile)の`ENTRYPOINT`を見ると`jenkins.sh`を実行するようになっています.  
`docker run`でコマンドも指定していませんので, `docker attach`はこれに接続することになります.  

```bash
ENTRYPOINT ["/bin/tini", "--", "/usr/local/bin/jenkins.sh"]
```

以上です.  
