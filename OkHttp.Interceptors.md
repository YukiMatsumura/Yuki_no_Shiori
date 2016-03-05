## OkHttp Interceptors

本稿ではOkHttpのApplication InterceptorとNetwork Interceptorの役割について書きました.  
内容は[OkHttp公式wiki - Interceptors](https://github.com/square/okhttp/wiki/Interceptors)と被っていますが, メモとして残します.  


### Interceptor

OkHttpでは処理に割り込めるポイントが2つあり, 必要に応じて使い分けます.

![OkHttp Interceptor](https://raw.githubusercontent.com/YukiMatsumura/Yuki_no_Shiori/master/image/okhttp_interceptor.png)

アプリケーションとOkHttpとのコミュニケーションに割り込むInterceptorを`ApplicationInterceptor`, OkHttpとネットワークとのコミュニケーションに割り込むInterceptorを`NetworkInterceptor`と呼びます.  

これらは`OkHttpClient.Builder`を経由して設定できます.  

```java
OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(new LoggingInterceptor())
    .addNetworkInterceptor(new LoggingInterceptor())
    .build();
```


### 挙動の違い

OkHttpはアプリケーションが楽できるように様々な処理を代行します.  

Redirect, Retryの処理はアプリとのコミュニケーションなしに, OkHttpに閉じて行われます.  
そういったアプリとのコミュニケーションが発生しないケースでは`ApplicationInterceptor`が使えません.  

Redirect, Retryの処理はネットワークとのコミュニケーションになります.  ここへ割り込むには`NetworkInterceptor`が使えます.  
ただし, Cacheレスポンスが働く場合はネットワークとのコミュニケーションが発生しません.  
そういったネットワークとのコミュニケーションが発生しないケースでは`NetworkInterceptor`が使えません.  

各Interceptorのポイントを下記に纏めます.  

`ApplicationInterceptor`  

 - RedirectやRetryといった中間応答には反応できない  
 - Cacheレスポンスにも反応する  
 - アプリケーションが発行するオリジナルリクエストが監視対象  

`NetworkInterceptor`  

 - RedirectやRetryといった中間応答にも反応できる   
 - Cacheレスポンスには反応できない
 - ネットワークへ発行されるリクエストが監視対象  

監視対象リクエストについて.  OkHttpはアプリからのリクエストを加工して通信します.  
この時, アプリからリクエストされたオリジナルの内容は`ApplicationInterceptor`が補足します.  
その後, OkHttpが加工したリクエストは`NetworkInterceptor`が補足します.  

```java
client.newCall(new Request.Builder()
        .url(url)
        .build())
        .execute();
```

上記の単純なGETリクエストを投げるコードで, `ApplicationInterceptor`, `NetworkInterceptor`それぞれに[HttpLoggingInterceptor](https://github.com/square/okhttp/tree/master/okhttp-logging-interceptor)を設定した場合のログ出力を確認します.  

```text
// ApplicationInterceptorによる出力
--> GET http://127.0.0.1:54817/ http/1.1
--> END GET

// NetworkInterceptorによる出力
--> GET http://127.0.0.1:54817/ http/1.1
Host: 127.0.0.1:54817
Connection: Keep-Alive
Accept-Encoding: gzip
User-Agent: okhttp/3.2.0
--> END GET
```

リクエストを加工する工程がみて取れました.  

他にも, OkHttpにCookie管理を任せてCookieヘッダをログで確認したい場合は`NetworkInterceptor`を使います.  

```java
CookieManager cookieManager
    = new CookieManager(null, ACCEPT_ORIGINAL_SERVER);
CookieHandler.setDefault(cookieManager);

server.enqueue(new MockResponse()
    .addHeader("Set-Cookie: a=hoge;")
    .addHeader("Set-Cookie: b=foo;"));
server.enqueue(new MockResponse());

client = client.newBuilder()
    .cookieJar(new JavaNetCookieJar(cookieManager))
    .build();

requestGet(urlWithIpAddress(server, "/"));
requestGet(urlWithIpAddress(server, "/"));
```

```text
--> GET http://127.0.0.1:56899/ http/1.1
Host: 127.0.0.1:56899
Connection: Keep-Alive
Accept-Encoding: gzip
Cookie: a=hoge; b=foo
User-Agent: okhttp/3.2.0
--> END GET
```

以上です.  
