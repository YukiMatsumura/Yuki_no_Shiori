GitLab公式ページ: https://about.gitlab.com/
GitLab Documentation: http://docs.gitlab.com/

GitLab Docker images:
http://docs.gitlab.com/omnibus/docker/

gitlab/gitlab-ce: https://hub.docker.com/r/gitlab/gitlab-ce/
GitLab Community Edition docker image based on the Omnibus package

gitlab/gitlab-ee: https://hub.docker.com/r/gitlab/gitlab-ee/
GitLab Enterprise Edition docker image based on the Omnibus package

環境はMacOS.

docker run --d \
    --hostname gitlab.example.com \
    --publish 10443:443 --publish 10080:80 --publish 10022:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest

Local location ＆ Container location ＆ Usage  

/srv/gitlab/data : /var/opt/gitlab ... For storing application data
/srv/gitlab/logs : /var/log/gitlab ... For storing logs
/srv/gitlab/config : /etc/gitlab ... For storing the GitLab configuration files

これでGitLabサーバが起動してくるのでdocker machineのIPアドレス:10080にアクセスします.  
  ex) http:////192.168.99.100:10080

イメージ起動直後にアクセスしてもレスポンスエラーが返されるので, しばらく待ってからアクセスします.  
イメージにプリインされているrootユーザの情報は下記です.  

 username: `root`
 password: `5iveL!fe`


GitLabのコンフィグレーションファイル /etc/gitlab/gitlab.rb
sudo docker exec -it gitlab vi /etc/gitlab/gitlab.rb

external_url "http://localhost"
SMTPの設定を行う. http://h-otter.hatenablog.jp/entry/2015/07/31/220511


Gmailを使う場合, 安全性の低いアプリからのアクセスを有効にする.
安全なアプリからの接続とする方法があるかもしれない（TODO）
http://www.tis2010.jp/knowledge/?pid=2


GitLab Configuration: http://docs.gitlab.com/omnibus/settings/configuration.html

Enable https

external_url "https://gitlab.example.com"
omnibus-gitlab will look for key and certificate files called /etc/gitlab/ssl/gitlab.example.com.key
and /etc/gitlab/ssl/gitlab.example.com.crt, respectively. 



GitHubの"Pull Request"は, GitLab上では"Merge Request"と呼ばれる.  
