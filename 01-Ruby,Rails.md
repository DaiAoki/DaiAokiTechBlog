# Ruby, Ruby on Rails

## ディレクトリが存在するか検証し、作成する
Rubyで、ディレクトリが存在するか検証し、存在しない場合作成する。  
深い階層にも対応。

### コードサンプル
```ruby
path = "path/to/hoge"
FileUtils.mkdir_p(path) unless FileTest.exist?(path)
```

### ユースケース
動的にディレクトリを作成して、ファイルを格納したい時に使用した。

```ruby
  # 省略
  save_dir += "/uploads/#{Time.now.strftime('%Y%m%d')}"
  FileUtils.mkdir_p(save_dir) unless FileTest.exist?(save_dir)
end
```

## strftimeで月日などの0埋めを行わないようにする
1. 0埋めする場合(通常)
```ruby
Date.today.strftime("%Y/%m/%d") #=> 2018/10/03
```

2. 0埋めしない場合
```ruby
Date.today.strftime("%Y/%-m/%-d") #=> 2018/10/3
```

## 使用可能なメソッド一覧を取得する

### Object#methods
`methods`で、使用可能なメソッドの一覧をシンボルの配列として取得することができる。

```ruby
hoge.methods
```

ただし、publicかprotectedなメソッドだけである。（privateなメソッドは含まれない）  
＊privateメソッドを取得したい時は、 private_methods を使用すれば良い。

### Enumerable#grep
`methods`で出力される結果は膨大なため、`grep`と複合して使用すると便利。

```ruby
hoge.methods.grep(/update/)
```

これで`update`を含むメソッドの一覧を取得することができる。

## Rakeタスクの作り方

### Rakeタスクとは
Rubyで一連の処理を記述したもの。シェルスクリプトと比較すると、書きやすく、DBに一斉に変更加えるときやメール配信などの定期実行する処理に使用すると便利。

### 基本形
```ruby
desc "taskの説明。日本語でも可"
task :taskの名前（英語） do
  #  一連の処理を記述する
  p "Hello, world"
end
```

```bash
rake -T // 定義されているrakeタスク一覧を表示
  rake taskの名前
rake taskの名前 //rakeタスクの実行
  Hello, world
```

### Rakeタスクをグループ化する
Rakeタスクをグルーピング化し名前空間を分けるには、`namespace`を使う。名前空間を分けることで、異なるnamespace間で同一名称のtaskを定義することができる。

```ruby
namespace :hoge do
  desc "hogeをbarする"
  task :bar do
    // 処理
  end
end

namespace :foo do
  desc "fooをbarする"
  task :bar do
    // 処理
  end
end
```

```bash
rake -T
  rake hoge:bar
  rake foo:bar
```

### 名前空間のネスト

```ruby
namespace :hoge do
  namespace :foo do
    desc "barする" do
    task :bar do
      # 処理
    end
  end
end
```

```bash
rake -T
  rake hoge:foo:bar
```

### 大量のデータを扱う場合にはfindではなくfind_eachを
- find
  - 全データをメモリに展開 → 処理
- find_each
  - デフォでデータを1000件ずつメモリに展開 → 処理
    - `batch_size`を指定することで件数の変更可

## 『Rubyのしくみ（オーム社）』を読んでみた

### Rubyでコードが実行されるまで
1. Rubyコード
2. 字句解析
3. 構文解析
4. コンパイル
5. YARV命令

```
字句解析：ソースコード内のテキストをトークン列に変換する
構文解析：トークン列を意味のある単位にグループ化する
コンパイル：YARV(Yet Another Ruby Virtual Machine)というRubyの仮想マシンを使って実行できる低レベルの命令に変換する
```

### 字句解析と構文解析
以下のようなプログラムがあったとする。

```ruby
10.times do |n|
  puts n
end
```

1行目を字句解析すると以下のようになる。  
```
「tINTEGER(10) 」「.」「tIDENTIFIER(times)」「keyword_do」「|」「tIDENTIFIER(n)」「|」
```

これでトークン列に変換された。  

次に構文解析。構文解析はあらかじめ用意された文法規則に一致するトークン列をグループ化する。  
method_callとbrace_blockを例に文法規則とはどういうものかをまとめる。  

- method_call
```
method_call: ... | primary_value '.' operation2 | ...
```

例えば、10.timesがこれにあたる。  

- brace_block
```
brace_block: ... | keyword_do opt_block_param compstmt keyword_end | ...
```

keyword_doは予約語doに、opt_block_paramはブロック引数|n|に、compstmtはブロックに含まれる処理に、keyword_endは予約語endにそれぞれ一致する。  
下記処理がこれにあたる。  
```
do |n|
  puts n
end
```

### 抽象構文木（AST）
Rubyがコードをどう解析するかをRipperと使用することで確認することができる。

```ruby
require 'ripper'
require 'pp'
code = <<STR
10.times do |n|
  puts n
end
STR
puts code
pp Ripper.sexp(code)
```

↓出力結果

```
[:program,
 [[:method_add_block,
   [:call, [:@int, "10", [1, 0]], :".", [:@ident, "times", [1, 3]]],
   [:do_block,
    [:block_var,
     [:params, [[:@ident, "n", [1, 13]]], nil, nil, nil, nil, nil, nil],
     false],
    [[:command,
      [:@ident, "puts", [2, 2]],
      [:args_add_block, [[:var_ref, [:@ident, "n", [2, 7]]]], false]]]]]]]
```

これを図式化すると、木構造になっていて、抽象構文木（AST）と呼ばれている。

## gsubでバックスラッシュ"\"に置き換える

```
"'hoge".gsub(/'/, "#{Regexp.quote('\\')}\'")
```

## フレームワークを使わず、Rackを使用してCGIアプリケーションを作る

### Rackとは
WEBrickやThin、Unicornと言ったWebアプリケーションサーバとRuby on RailsなどのWebアプリケーションフレームワークのインターフェースとなるライブラリ。  
あるWebアプリケーションフレームワークはWEBrickでは動かせるけど、Thinでじゃ動かないといった問題を解決すべく、WebアプリケーションサーバとWebアプリケーションフレームワークの間で共通のインタフェースを決めたもの。（疎結合を保つことができる）

### Rackのインストール
```bash
bundle init
```

```ruby
gem "rack", "~> 2.0"
```

```bash
bundle install
  // Rubyのバージョンが低くてエラー
  rack (~> 2.0) was resolved to 2.0.4, which depends on
  ruby (>= 2.2.2)
rbenv versions
  2.2.3
  2.3.0
  2.5.0-dev
rbenv local 2.3.0
bundle install
// rackupコマンドでインストールされたRackのバージョンを確認
bundle exec rackup -v
Rack 1.3 (Release: 2.0.4)
```

### 最小限のRackアプリケーション
- callメソッドを持つオブジェクトであること
- callメソッドは引数で環境変数(Hashを受け取る)
- callメソッドの戻り値は下記三つの配列であること
　- ステータスコード
　- HTTPヘッダ
　- HTTPメッセージボディ

```ruby
require 'rack'

class RackApplication
  def call(env)
    [200, {'content-Type' => 'text/plain'}, ['Hello!']]
  end
end

run RackApplication.new
```

Rackアプリケーションの起動と動作確認

```bash
bundle exec rackup config.ru // 上記コードをconfig.ruという名前で保存している場合
  [2018-02-12 17:32:14] INFO WEBrick 1.3.1
  [2018-02-12 17:32:14] INFO ruby 2.3.0 (2015-12-25) [x86_64-darwin16]
  [2018-02-12 17:32:14] INFO WEBrick::HTTPServer#start: pid=50460 port=9292
curl -i http://localhost:9292
  HTTP/1.1 200 OK
  Content-Type: text/plain
  Transfer-Encoding: chunked
  Server: WEBrick/1.3.1 (Ruby/2.3.0/2015-12-25)
  Date: Mon, 12 Feb 2018 08:35:10 GMT
  Connection: Keep-Alive
  Hello!
```
＊ ブラウザで localhost:9292 にアクセスしても良い。

### Rackのリクエストクラスとレスポンスクラス
リクエスト情報の入ったenvの値や、レスポンスの配列を扱いやすくするために、Rack::RequestやRack::Responseというラッパークラスが用意されている

```ruby
require 'rack'

class RackApplication
  def call(env)
    request = Rack::Request.new(env)

    response = if request.path_info == '/'
                 body = "#{request.request_method}: Hello! #{request.params['name']}!"
                 Rack::Response.new(body, 200, {'Content-Type' => 'text/plain'})
               else
                 Rack::Response.new('Not Found', 404, {'Content-Type' => 'text/plain'})
               end

    # callメソッドの戻り値として扱えるインターフェースへ変換
    response.finish
  end
end

run RackApplication.new
```

```bash
curl http://localhost:9292/\?name=Ruby
  GET: Hello! Ruby!
curl http://localhost:9292/ -d "name=Ruby"
  POST: Hello! Ruby!
curl http://localhost:9292/foo\?name=Ruby
  Not Found
```

### Rackのミドルウェア機構
Basic認証とステータスコードが400/500系のエラー時のエラー画面の表示をするミドルウェア。  

- Rack::Auth::Basic：Basic認証を扱う
- Rack::ShowStatus：例外発生時やステータスコードが400・500系の場合に詳細なエラーを画面に表示する

```ruby
require 'rack'

class RackApplication
  def call(env)
    [200, {'content-Type' => 'text/plain'}, ['Hello!']]
  end
end

use Rack::ShowStatus
use Rack::Auth::Basic do |username, password|
  username == password
end

run RackApplication.new
```

### Rackのミドルウェアを作成する
Rackミドルウェアのインターフェース  
- コンストラクタでRackミドルウェア、またはRackアプリケーションのインスタンスが渡される
- callメソッドを持つ（引数や戻り値はRackアプリケーションと同じ）

＊ callメソッド中にコンストラクタで受け取ったインスタンスのcallメソッドを呼び出して、後続のミドルウェアやアプリケーションを実行する。  
　（callメソッドを呼ばない場合、後続の処理を実行せずアプリケーションの処理を中断するということになる。）

```ruby
require 'rack'

class URLFilter
  def initialize(app)
    @app = app
  end

  def call(env)
    if env['PATH_INFO'] == '/admin'
      [
        404,
        {'Content-Type' => 'text/plain'},
        ["Not Found.(PATH=#{env['PATH_INFO']})"]
      ]
    else
      @app.call(env)
    end
  end
end

class RackApplication
  def call(env)
    [
      200,
      {'content-Type' => 'text/plain'},
      ["RackApplication(PATH=#{env['PATH_INFO']})"]
    ]
  end
end

# useメソッドで登録した順に実行されていく
# URLFilter#call -> Rack::Auth::Basic#call -> RackApplication#call
use URLFilter
use Rack::Auth::Basic do |username, password|
  username == password
end

run RackApplication.new
```

## TCPソケット通信
localhostのポート20000番をopenして、ソケットを通じてメッセージを表示してみる。

```ruby
#!/usr/bin/ruby
require 'socket'

server = TCPServer.new(20000)

loop do
  socket = server.accept

  while str = socket.gets.chomp
    puts "Receive: #{str}"

    socket.puts "SERVER received '#{str}' from Client."
  end

  socket.close
end
```

↓telnetで動作確認

```bash
chmod 755 socket_server.rb
./socket_server.rb
telnet localhost 20000
//適当な文字列を入力する
```

### クライアント側も実装

```ruby
#!/usr/bin/ruby
require 'socket'

# localhostのポート20000番のソケットを開ける
socket = TCPSocket.open("localhost", 20000)

# 文字列入力を受け付ける
while line = $stdin.gets
  socket.puts line
  socket.flush

  puts socket.gets
end

socket.close
```

↓telnetを使わず、serverとclientで動作確認

```bash
chmod 755 socket_client.rb
./socket_client.rb
//適当な文字列を入力する
```

## ブロックについて

### yield

```ruby
def yield_sample
  yield
end

yield_sample do
  puts 'Hello, world!'
end    # => Hello, world!
```

yieldを呼び出すメソッドをblockなしで呼び出すとエラーになる。

```ruby
yield_sample    # => Local JumpError: no block given (yield)
```

block与えられた時のみyieldを実行する。

```ruby
def yield_sample
  yield if block_given?
end

yield_sample    # => エラーにならない

yield_sample do
  puts "Hello, world!"
end    # => Hello, world!
```

### ブロックの戻り値/引数
yieldの呼び出しで渡した値はブロックに渡される。

```ruby
def with_current_time
  yield Time.now
end

with_current_time do |now|
  puts now.year
end    # => 2018
```

ブロック引数にもデフォルト値や可変長引数を指定できる。

```ruby
# デフォルト値
def default_argument_for_block
  yield
end

default_argument_for_block do |val = 'Hi'|
  puts val
end    # => Hi


# 可変長引数
def flexible_arguments_for_block
  yield 1, 2, 3
end

flexible_arguments_for_block do |*params|
  puts params.inspect
end    # => [1, 2, 3]
```

### whileとyieldでeachを定義する

```ruby
class MyEach
  def initialize(array)
    @array = array
  end

  def each
    i = 0
    while i < @array.length
      yield @array[i]
      i += 1
    end
  end
end


hoges = MyEach.new([1,2,3])
hoges.each do |hoge|
  p hoge
end    # 1 2 3
```

### 仮引数としてブロックを受け取る
受け取ったメソッドをyieldでそのメソッド内で実行せずに、他のメソッドにProcオブジェクトとして渡したい時は、ブロックを仮引数として受け取る。

```ruby
def block_sample(&block)
  block.call if block
end

block_sample do
  puts 'Hello!'
end    # => Hello
```

### オブジェクトをブロックとして渡す
```
people = %w(Dai Aoki)
block = Proc.new {|name| puts name }

people.each &block    # => Dai Aoki



p1 = Proc.new {|val| val.upcase}
p2 = :upcase.to_proc

p1.call('hello')    # => HI
p2.call('hello')    # => HI



people.map {|person| person.upcase}    # => ["Dai", "Aoki"]

# ブロック引数に対して引数なしのメソッド呼び出しを行いたいときには、container.map(&:method_name)の記法が用いられる
people.map(&:upcase)    # => ["Dai", "Aoki"]
```

### 繰り返し以外に用いられるブロック
`前処理 -> 本質的な処理 -> 後処理` というように、前後の処理を共通化して、本質的な処理に集中できるようにするパターンの処理にも向いている。

- ファイルのオープン/クローズ
- DBへの接続/切断
- トランザクションの開始と終了
- ロックと解放

```ruby
def write_with_lock
  File.open 'time.txt', 'w' do |f|
    f.flock File::LOCK_EX    # 前処理
    yield f    # 本質的な処理
    f.flock File::LOCK_UM    # 後処理
  end
end

write_with_lock do |f|
  f.puts Time.now
end
```

## .env + .gitignore + YAMLで非公開の環境変数を管理する

### 1. .envファイルを作成する

```bash
HOGE="HOGE_VALUE"
FOO="FOO_VALUE"
BAR="BAR_VALUE"
```

### 2. .envファイルを.gitignoreに追加する

```bash
.env
```

### 3. .envファイルに定義された環境変数を参照する
initializerや、起動時に読み込まれるファイル  
```ruby
require './config/setting'
```

/config/setting.rb  
```ruby
require 'yaml'
require 'erb'
require 'dotenv/load'

$conf = YAML.load(ERB.new(File.read('config/config.yml')).result)
```

/config/config.yml  
```ruby
sample:
  hoge: <%= ENV['HOGE'] %>
  foo:  <%= ENV['FOO'] %>
  bar:  <%= ENV['BAR'] %>
```

グローバル変数$confに.env + YAMLに記述した環境変数が格納されるので参照する。  
```ruby
p $conf['sample']['hoge']    #=> "HOGE_VALUE"
p $conf['sample']['foo']     #=> "FOO_VALUE"
p $conf['sample']['bar']     #=> "BAR_VALUE"
```

## 特定のディレクトリ以下のファイルをすべてrequireする
下記のようなディレクトリ構造  
```
modules
├hoge.rb
├foo.rb
└bar.rb
```

```ruby
Dir[File.expand_path('../modules', __FILE__) << '/*.rb'].each do |file|
  require file
end
```

## AWSのEC2インスタンスのRubyバージョンアップ(rbenvでバージョン管理)
1.  ruby-buildのバージョンアップ
```bash
cd ~/.rbenv/plugins/ruby-build/
git fetch
git pull
```
2. autoconfのインストール
```bash
sudo yum install autoconf
Complete!
```
3. bisonのインストール
```bash
sudo yum install bison
Complete!
```
4. Rubyのインストール
```bash
rbenv install 2.5.0-dev
bundle install
rbenv exec gem install rails
```

-----
ここからRails

## includesしてN+1問題を解決
### N+1問題とは？
N+1問題とは簡単にいうと、必要以上にSQLのクエリが走ること。Controllerでとってきた値をviewでeachで回す時なんかに、注意しないと発生してしまう。  
例えば、@users = User.allして、  
```ruby
- @users.each do |user|
　= user.friends.count
```

とかってやると、eachで回るたびに、userの数だけfriendテーブルに対するクエリが発行される。  
最初に発行したUser.allのクエリを足して、`N+1`

### 対策方法
対策方法はシンプルで、事前にincludesしておけばよい。  
```ruby
@users = User.all.includes(:friends)
```

## ajaxでPATCHリクエスト送信時、InvalidAuthenticityToken

### ActionController::InvalidAuthenticityToken
RailsでajaxでPATCHリクエストを送信する際に、**ActionController::InvalidAuthenticityToken**のエラーが発生し、通信が失敗した。  
原因は、クロスサイトリクエストフォージェリ対策のトークンが不正ということ。

### 解決策
ajax通信送信前にリクエストヘッダーにクロスサイトリクエストフォージェリートークンをセットする。

```javascript
$.ajax({
  async: true,
  type: "PATCH",
  url: /hoge
  dataType: "text",
  timeout: 5000,
  beforeSend: function(xhr) {
    xhr.setRequestHeader("X-CSRF-Token", $("meta[name="csrf-token"]").attr("content"))
  },
  error: function (data) {
    // 失敗時の処理
  },
  success: function (data) {
    // 成功時の処理
  }
});
```

`beforeSend`で、  
```javascript
xhr.setRequestHeader("X-CSRF-Token", $("meta[name="csrf-token"]").attr("content"))
```

## highlight + deleteで特定の文字を装飾する

### highlight + delete
Railsのhighlight関数はデフォだと特定の文字列をstrongタグで囲む。  
ハイライトする文字列に正規表現を、highlighterに任意のタグを指定することで自由に装飾可能。  
> highlight(文字列, ハイライトする文字列 [, :highighter => ハイライトする文字列を置き換える形式)  

```ruby
def highlight_span(text)
  highligth(text, Regexp.new(/`.*`/), highlighter:  '<span class="util-highlight-span">\1</span>').delete('`').html_safe
end
```

## ActiveRecordのhas_manyのdependentオプションまとめ
dependentオプションは、簡単にいうと親レコードが削除された場合に子レコードはどうするかを指定する。  

```ruby
class Article < ActiveRecord::Base
  has_many :comments
end

class Comment < ActiveRecord::Base
  belongs_to: article
end
```

1. 記事が削除されたらコメントも削除(destroyメソッドが実行される)
```ruby
has_many :comments, dependent: :destroy
```

2. 記事が削除されたらコメントも削除(直接DBから削除される)
```ruby
has_many :comments, dependent: :delete_all
```


3. 記事が削除されたらコメントの外部キーをnullにする
```ruby
has_many :comments, dependent: :nullify
```

[RAILS GUIDES　Active Record の関連付け (アソシエーション)](https://railsguides.jp/association_basics.html)

## ActiveRecordのhavingでhas_manyな子レコードのうち、2件以上あるものを抽出する

```ruby
class Hoge < ActiveRecord::Base
  has_many :bars
end

class Bar < ActiveRecord::Base
  belongs_to :hoge
end
```

上記のようなリレーションのモデルがあった時に、共通の外部キーhoge_idを持つbarのレコードを抽出したかった。  
通常のSQLだと以下のようにかける。  

```sql
SELECT *
FROM db_name.bars
GROUP BY hoge_id
HAVING count(*) >= 2
```

ActiveRecordのメソッドを使うと以下のようになる  

```ruby
Bar.group(:hoge_id).having(['count(*) >= ?', 2])
```

countとかの集計関数と組み合わせる時に、havingを使うと便利。
