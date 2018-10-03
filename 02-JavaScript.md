# JavaScript

## Railsでremote trueでモーダル表示する際に、"ready page:load"、 "on"が効かない

### やろうとしたこと
- Rails + jQuery で、リンクを押した際にajax通信（非同期）で指定のidの内容を置きかえようとした。
- より具体的には、指定のidをモーダルウィンドウに置き換え、そのモーダルの中でJavaScriptのイベントを発火させようとした。

### 失敗したコード
＊ 以下、関係のあるところのみ抜粋

`hoge.html.haml`
```haml
.hoge
  -# 省略
  = link_to foo_path, remote: true do
    ボタン

#js-hoge-modal
```

`hoge_controller.rb`
```ruby
def foo
  # 適当な処理
end
```

`foo.js.haml`
```haml
:plain
  $("#js-hoge-modal").html(
    "#{escape_javascript(render partial: "hoge_modal")}"
  );
```

`_hoge_modal.html.haml`
```haml
.bar
  #js-bar
```

`hoge-modal.js`
```javascript
$(document).on("ready page:load", function() {
  $("#js-bar").on("click", function() {
    // ここの処理が実行されなかった！
  });
});
```

### 調査
jsの読み込みタイミングが怪しいと思ってjsの頭でalertしたら、案の定、remote trueで foo.js.haml を取得する前に実行され、取得してレンダリングするタイミングでは実行されなかった。  
つまり、以下のような順番で実行されていたことになる。  
1. hoge-modal.jsの ready page:load イベント発火
2. hoge.html.hamlのレンダリング
3. remote true のリンククリック
4. HogeControllerのfooアクションが呼ばれる
5. foo.js.hamlが呼ばれる（#js-hoge-modal の内容を _hoge_modal.html.hamlで置き換える）  

### 結論
- ajax:success のイベントを使用した
- on だと動かなかったため、 bind とした。（このあたりは、また別の機会にまとめる）

### 成功したコード
変更したのはhoge-modal.jsのみ。  
```javascript
$(document).on("ajax:success", function(){
  $("#js-bar").bind("click", function() {
    // ここの処理が実行されなかった！
  });
});
```

## Ajax通信で進捗を表示する

### 実現したいこと
ajax通信で進捗を表示したい。  
非同期処理で、ユーザーの待ち状態が長く続いて不安にならないようにしたい、UX高めたいという時に使える。  

### 実装方法

```javascript
$(document).ajaxStart( function() {
  // ajaxスタート時の処理
});
$(document).ajaxComplete( function() {
  // ajax完了時の処理
});
```

ajaxスタート時、画面に、background: #000000 + opacityのオーバーレイを表示しその上に進捗表示のDOMを追加  
ajax完了時に取り除くとかすれば、ユーザーの体感待ち時間減らせる。  

他にも使いようによってはいろいろできそう。

### Tips
ajaxStartと似たものでajaxSendがある。  
二つの違いは、簡単にいうと複数のajax通信が同時に走る場合に、各ajax通信の始めに実行されるのがajaxSend、一連のajax通信のうち初めに実行されるajax通信の始めに実行されるのがajaxStart。（説明くどくなったので下記にまとめる）  
```
ajaxSend : 3つのajax通信が走る場合、各ajax通信の始めに実行される => 3回実行
ajaxStart : 3つのajax通信が走る場合、初回のみ実行される => 1回実行
```

使ったことなかったけど、便利そうだから使う機会あったら使う。
