### Screen Zoom

Android N Developer Previewで搭載されたユーザ補助機能`Screen Zoom`.  
低視力者はこの機能を使うことで表示を拡大した状態で使うことができます.  
（あるいは表示を縮小して情報量を増やすことも可能です）  

この`ScreenZoom`機能はDensityDPIを変更することで表示の拡大・縮小を実現しています.  
DensityDPI値を大きくしても画面の物理サイズ(px)は変わりませんので結果的に拡大表示されているように見せることができる仕組みです.  
デフォルトが`xxhdpi`なNexus5Xも, `ScreenZoom`の設定で`xhdpi`, `xxxhdpi`に変更できます.  
開発者はAndroidN以降で端末のDensityDPIが変更される可能性を考慮する必要があります.  


### drawable-xxx

`ScreenZoom`でDensityが変わりますので, `drawable-xhdpi`, `drawable-xxhdpi`など設定によって適切なdpi設定修飾子のリソースが選択されるようになります.  
ただ, 動作確認でハマったポイントがありました.  

AndroidStudio2.1で動作確認したところ, たとえ各DPI毎のdrawableリソースを持っていたとしても, `ScreenZoom`の設定を変えてもそれらが適切に反映されませんでした. 

例): ScreenZoom値を変えても参照される`dwarable`リソースが変わらない...

<a href="https://3.bp.blogspot.com/-OxL4f1OWmYQ/VxHQwUIbOoI/AAAAAAAANjI/fmGmW1J2HN83sdB_7e8ruYaxzxth49rlACLcB/s1600/%2Bbad.png" imageanchor="1"><img border="0" height="532" src="https://3.bp.blogspot.com/-OxL4f1OWmYQ/VxHQwUIbOoI/AAAAAAAANjI/fmGmW1J2HN83sdB_7e8ruYaxzxth49rlACLcB/s640/%2Bbad.png" width="640" /></a>

ドロイド君アイコンに格納したDPIの値を記載しています. 上記は`drawalbe-xxhdpi`に格納された画像が表示されている状態です. 

原因はAndroidStudioがインストール先のデバイスを決定してから作成するapkにありました.  
実際に作成されているapkを確認すると, インストール対象デバイスのデフォルトDensityDPI以外のリソースを削除している様子でした.  
動作確認をするためには`./gradlew :app:assembleDebug`で全DensityDPI向けのdrawableを含めたapkを作成し, インストールする必要がありました. 

例): ScreenZoom値を変えると参照される`dwarable`リソースが変わる

<a href="https://1.bp.blogspot.com/-EVJHNkM3xbo/VxHQwY3oSqI/AAAAAAAANjE/PPhTFer0i_8QXhn-0ufI3ctTOJyPGX-YwCKgB/s1600/%2Bgood.jpg" imageanchor="1"><img border="0" height="534" src="https://1.bp.blogspot.com/-EVJHNkM3xbo/VxHQwY3oSqI/AAAAAAAANjE/PPhTFer0i_8QXhn-0ufI3ctTOJyPGX-YwCKgB/s640/%2Bgood.jpg" width="640" /></a>

以上です. 
