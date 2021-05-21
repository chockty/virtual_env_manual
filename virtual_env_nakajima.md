# virtual_env_manual

### <span style="color: red; ">!!Caution!!</span> 手順書を進める前に一読ください。
##### ① 手順書は前の項目を踏まえた上で進みます。項目を飛ばして進めるとエラーを発生させる可能性がありますので、飛ばさず順番通りに進めてください。
##### ②コードブロックに記載されているコードは上から順番に実行してください。
##### ③ コードブロック内の「\$」はプロンプトを意味しているので、実際に打ち込んでいただく際は「$」を外して入力してください。「#」は注意書きや補足事項が記載されておりますので、漏れなく読んでください。
##### ④指示がない限り、ゲストOSからのログインや、ディレクトリの移動は不要です。前工程と同じ状態で進めてください。
<br />

## ＜本手順書のゴール＞
本手順書では、仮想環境上にlaravelの簡易的なアプリケーションを実装頂きます。
機能としては、ユーザ登録、ユーザログイン、ユーザログアウトの3つの機能が実装されます。
仮想環境上でlaravelを起動させ、以上3点の機能が問題なく使用できることが本手順書のゴールです。
<br />

## ＜イメージ図＞
【アプリケーション】→ laravel（5.laravelの導入 / 動作確認）<br />
【&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ゲストOS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;】→ CentOS（2. PHPの導入、3. Nginxの導入、4. Mysqlの導入）<br />
【&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ホストOS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;】→MacOS（本手順書はMacPCを想定しております。）&nbsp;VirtualBox / vagrant（0. 前提条件、1. 作業用ディレクトリの作成及び初期設定）
<br />

## ＜バージョン一覧＞
| 項目 | サービス | バージョン |
|:-:|:-:|:-:|
| OS              | CentOS  | 7       |
| Language        | PHP     | 7.3     |
| Server software | Nginx   | 1.19.10 |
| Database        | MySQL   | 5.7     |
| Framework       | laravel | 6.0     |
<br />

## ＜Index＞
#### 0. 前提条件
#### 1. 作業用ディレクトリの作成及び初期設定
#### 2. PHPの導入
#### 3. Nginxの導入
#### 4. Mysqlの導入
#### 5. laravelの導入 / 動作確認

<br />

## 0. 前提条件
#### 未実施の場合は参考情報を基に実施してください
- ① Virtualbox をインストールしている。
参考：https://www.virtualbox.org/wiki/Download_Old_Builds_6_0
※本手順書では、バージョン「6.0.14」を採用しております。
<br />

- ② Vagrant をインストールしている。
参考：https://www.vagrantup.com
※本手順書では、バージョン「2.2.16」を採用しております。
<br />

- ③ CentOS 7 のboxをインストールしている。
実行コマンド：vagrant box add centos/7
（macの場合はターミナル、windowsの場合はコマンドプロントで実行してください）
参考：https://learn.hashicorp.com/tutorials/vagrant/getting-started-boxes?in=vagrant/getting-started
<br />

## 1. 作業用ディレクトリの作成及び初期設定
#### Vagrant用の作業ディレクトリを用意し、最低限の初期設定を実施します。
- ① 作業用ディレクトリの作成。
（場所の指定はありませんが、自分がアクセスしやすい場所がおすすめです。）<br />
実行コマンド：
  ```
  $ mkdir ディレクトリ名
  ```
    ##### <span style="color: red; ">!!caution!!</span> 作成したディレクトリを移動させると、後ほどゲストOSにログインができなくなる可能性があるので注意してください。
<br />

- ② ①で作成したディレクトリに入り、ゲストOSを初期化する。
実行コマンド：
  ```
  $ cd ①で作成したディレクトリ

  # 本手順書では、0. 前提条件でインストールしたCentOS 7 を指定しております。
  $ vagrant init centos/7
  
  # 実行後問題なければ以下のような文言が表示されます
  A `Vagrantfile` has been placed in this directory. You are now
  ready to `vagrant up` your first virtual environment! Please read
  the comments in the Vagrantfile as well as documentation on
  `vagrantup.com` for more information on using Vagrant.
  ```
<br />

- ③ Vagrantfileの編集
vagrantの基本設定が記述されているVagrantfileを編集します。
実行コマンド：
  ```
  $ vi Vagrantfile

  # ファイルの内容が表示されたら「i」キーを押して「INSERT」モードに切り替えてください。
  ```
  変更内容：
    (1) 以下の内容をコメントイン（#を外す）する
    ```
    config.vm.network "forwarded_port", guest: 80, host: 8080
    config.vm.network "private_network", ip: "192.168.33.19"

    # 本手順書では「ip:」を　192.168.33.19を設定しておりますが必要に応じて書き換えてください。
    ```
    既に別の仮想環境を構築済みの場合は「host:」「ip:」が構築済み仮想環境の設定と同じ値にならないように注意してください。
    vagrantを起動させる際にエラーが発生します。
    <br />

    (2) 以下の内容を編集する（コメントインも実施してください）
    ```
    config.vm.synced_folder "../data", "/vagrant_data"
    # ↓ 以下に編集
    config.vm.synced_folder "./", "/vagrant", type:"virtualbox"

    # 編集が終わりましたら、「esc」キーを押した後に「:wq」と入力してEnterを押してください。
    ```
<br />

- ④ vagrant pluginのインストール
インストールするplugin：vagrant-vbguest<br>
vagrant-vbguestを使用することによって、boxにインストールされたisoファイルのGuestAddition(ここで使われている仮想OS)を自動的に最新版へ更新してくれます。更新をしないと、Virtualboxがisoファイルを適用外と判断し、エラーが発生してしまう可能性があります。
    <br />

  実行コマンド：
  ```
  $ vagrant plugin install vagrant-vbguest --plugin-version 0.21

  # 問題なくインストールされていれば、以下のコマンドの実行後に「vagrant-vbguest」が表示されます。
  $ vagrant plugin list
  ```
  vagrant-vbguestのバージョンによっては仮想環境上に共有フォルダであるvagrantフォルダが作成されないケースがあるため、本手順書ではバージョンを指定しております。
    <br />

- ⑤ ゲストOSの起動 / ログイン
ゲストOSを起動し実際にログインしてみます。
    実行コマンド：
    ```
    $ vagrant up # 起動コマンド
    $ vagrant status # 状態確認コマンド

    # 以下のように表示されたら起動成功です。
    Current machine states:
    default                   running (virtualbox)

    # 起動に成功しましたら、以下のコマンドでログインしてください。
    $ vagrant ssh

    # ログイン成功の場合、以下の内容が表示されます。
    Welcome to your Vagrant-built virtual machine.
    [vagrant@localhost ~]$
    # ↑ ゲストOSヘログインしている状態のプロンプトです。

    # 今回は不要ですが、ログアウトしたい場合は「exit」コマンドを投入してください。
    ```
  <br />

## 2. PHPの導入
#### 起動できたゲストOSにPHPを導入していきます。
##### !!caution!! 3. の続きで、ゲストOSにログインしたまま進めます。
- ① パッケージのインストール
gitなど開発に必要なツール群をインストールします。
  実行コマンド：
  ```
  $ sudo yum -y groupinstall "development tools"
  ```
  <br />

- ② PHPのインストール
PHPを動かすためのツールをインストールします。
  実行コマンド：
  ```
  $ sudo yum -y install epel-release wget
  $ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
  $ sudo rpm -Uvh remi-release-7.rpm
  $ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml  php-fpm php-common php-devel php-mysql unzip
  
  # 以下のコマンドを実行してPHPのヴァージョン情報が表示されたらインストール成功です。
  $ php -v
  ```
  <br />

  今回はphp 7.3をインストールするために「php73」と指定をしております。「73」の箇所を変更することによって、指定したバージョンのphpをインストールすることができます。
  <br />

- ③ composerのインストール
PHPで使用するライブラリの依存関係を管理するcomposerをインストールします。
  実行コマンド：
  ```
  $ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
  $ php composer-setup.php
  $ php -r "unlink('composer-setup.php');"

  # どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動を行います
  $ sudo mv composer.phar /usr/local/bin/composer

  # 以下のコマンドを実行し、composerのバージョンが表示されたらインストール成功です。
  $ composer -v
  ```
<br />

## 3. Nginxの導入
#### PHPを動かすためのサーバソフトウェアNginxをインストールし設定していきます。
- ① インストール準備
Nginxの最新版をインストールできるように事前に設定を行います。
実行コマンド：
  ```
  $ sudo vi /etc/yum.repos.d/nginx.repo
  # ファイルの内容が表示されたら「i」キーを押して「INSERT」モードに切り替えてください。
  # 編集前のファイルは空ですが問題ございません。

  # 以下の内容を追記してください。
  (左端にスペースが入っていると上手く機能しないので、左詰で記載してください。)
  [nginx]
  name=nginx repo
  baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
  gpgcheck=0
  enabled=1

  # 編集が終わりましたら、「esc」キーを押した後に「:wq」と入力してEnterを押してください。
  ```
- ② Nginxのインストール及び起動確認
実行コマンド：
  ```
  $ sudo yum install -y nginx

  # 以下のコマンドを実行してnginxのバージョン情報が表示されましたらインストール成功です。
  $ nginx -v

  # 以下のコマンドで実際にnginxを起動してみましょう。
  $ sudo systemctl start nginx

  # 以下のコマンドでnginxが正常に起動しているか確認しましょう。
  $ sudo systemctl status nginx

  # コマンド実行の結果、「Active: active (running) since 〜」と表示されていれば起動成功です。
  ```
  http://Vagrantfileで設定したipアドレス (本手順書の場合は 192.168.33.19 ) へアクセスしてnginxのwelcome画面が表示されることを確認してください。
<br />

## 4. Mysqlの導入
#### PHPのwebフレームワークであるlaravelを導入し、使用するデータベースとしてMysqlを導入していきます。
  ##### !!caution!! 実行場所：ゲストOS内 パス：/home/vagrant
- ① Mysqlの導入
実行コマンド：
  ```
  $ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
  $ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
  $ sudo yum install -y mysql-community-server
  
  # 以下のコマンド実行後にMysqlのバージョンが表示されたらインストール成功です。
  $ mysql --version
  ```
<br />

- ② Mysqlの設定
実行コマンド：
  ```
  $ sudo systemctl start mysqld
  $ sudo systemctl status mysqld
  # コマンド実行の結果、「Active: active (running) since 〜」と表示されていれば起動成功です。

  # Mysqlの初期パスワードを確認します。
  $ sudo cat /var/log/mysqld.log | grep 'temporary password'
  （Mysqlを起動させてからでないと確認できません）

  # 上記コマンドを実行し下記のように表示されたらOKです
  # 「hogehoge」にあたる箇所がパスワードです。
  2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
  
  # 以下のコマンドでMysqlログインをします。
  $ mysql -u root -p

  Enter password:
  # 上記が表示されたら、先ほど確認したパスワードを入力してください。
  
  mysql >
  # 上記のプロンプトが表示されたらログイン成功です。
  ```
  <br />
  
  Mysqlは初回ログインの際にパスワードを設定する必要がありますので、コマンドを投入して設定します。<br />
  実行コマンド：
  ```
  mysql > set password = "新たなpassword";
  # 新しいパスワードの条件：大文字小文字の英数字 + 記号かつ8文字以上
  ```
  「Query OK,」と表示されればパスワード設定完了です。
  <br />

  この後laravelを起動させるため、データベースも作成しておきましょう。
  
  <br />

  実行コマンド：
  ```
  mysql > create database 任意のデータベース名;
  mysql > show databases;
  ```

  作成したデータベースが表示されれば作成完了です。
  「exit」コマンドを実行し、Mysqlからログアウトしましょう。
  <br />

  （補足：不要な場合は飛ばして大丈夫です）
  もしパスワードの条件を簡単なものへ変更したい場合は以下の設定を実施してください。
  <br />

  ```
  $ sudo vi /etc/my.cnf

  # 以下の通り編集してください。

  〜省略〜

  # read_rnd_buffer_size = 2M
  datadir=/var/lib/mysql
  socket=/var/lib/mysql/mysql.sock

  # 下記の一行を追加
  validate-password=OFF

  # 編集後、以下のコマンドでMysqlを再起動してください。
  $ sudo systemctl restart mysqld
  ```
  <br />

  本番環境など簡易的なパスワードを設定するのに適さない場合もあると思います。状況に応じて設定してください。
  <br />

## 5. laravelの導入
- ① Laravelのプロジェクト作成
ゲストOS上でlaravelのプロジェクトを作成し、マイグレーションを実施します。<br />
    実行コマンド(1:プロジェクト作成)：
    ```
    $ cd /vagrant
    $ composer create-project laravel/laravel=6.0 --prefer-dist プロジェクト名
    ```

    本手順書では「laravel 6.0」を採用しておりますので、「laravel=6.0」と指定しております。
    「--prefer-dist」は安定稼働版をインストールするためのオプションです。
    <br />

- ② データベース連携の設定
laravelとMysqlを接続するためにlaravel側の設定を編集します。
  実行コマンド：
  ```
  $ vi /vagrant/作成したlaravelプロジェクト名/.env

  # ファイルの内容が表示されたら「i」キーを押して「INSERT」モードに切り替えてください。
  # 以下の内容の通り編集してください。
  DB_DATABASE=4.②で作成したデータベース名
  DB_PASSWORD=登録したパスワード

  # 編集が終わりましたら、「esc」キーを押した後に「:wq」と入力してEnterを押してください。
  ```
  <br />

- ③ マイグレーション実行
  実行コマンド:
  ```
  $ cd 作成したlaravelプロジェクト名
  $ php artisan migrate
  ```
  「Migration table created successfully.」と表示されればマイグレーション成功です。

  ##### !!caution!!
  必ず<span style="color: red;">ゲストOSにログインした状態で、 /home/vagrant/作成したlaravelプロジェクト名</span> のディレクトリでマイグレーションを実施してください。
  上記以外の条件でマイグレーションを実施するとエラーになりテーブルが作成されません。
<br />

- ④ nginx / php-fpmの設定
構築した環境上でlaravelを動かすための設定をします。<br />
  実行コマンド(1：nginx側の設定)：
    ```
    $ sudo vi /etc/nginx/conf.d/default.conf
    # ファイルの内容が表示されたら「i」キーを押して「INSERT」モードに切り替えてください。

    # 以下の内容の通り編集してください。
      listen       80;
      server_name  Vagranfileでコメントを外した箇所のipアドレス; # 本手順書の場合：192.168.33.19
      root /vagrant/laravelのプロジェクト名/public; # 追記
      index  index.html index.htm index.php; # 追記

    〜省略〜

    location / {
          #root   /usr/share/nginx/html; # コメントアウト
          #index  index.html index.htm;  # コメントアウト
          try_files $uri $uri/ /index.php$is_args$args;  # 追記
      }

    〜省略〜

    # 該当箇所をコメントインしてください。
    location ~ \.php$ {
      #    root           html;
          fastcgi_pass   127.0.0.1:9000;
          fastcgi_index  index.php;
          fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
          include        fastcgi_params;
      }

    # 編集が終わりましたら、「esc」キーを押した後に「:wq」と入力してEnterを押してください。
    ```
    <br />

    実行コマンド(2：php-fpm側の設定):
    ```
    $ sudo vi /etc/php-fpm.d/www.conf
    # ファイルの内容が表示されたら「i」キーを押して「INSERT」モードに切り替えてください。

    # 以下の内容の通り編集してください。
    ;24行目近辺
    user = vagrant
    〜省略〜
    group = nginx
    ```
    user を vagrantと設定することによってlaravel起動時に権限に関するエラーが発生することを防止できます。
  <br />
  
- ④ 起動確認
実行コマンド：
  ```
  $ sudo systemctl restart nginx
  $ sudo systemctl status nginx
  $ sudo systemctl start php-fpm
  $ sudo systemctl status php-fpm

  # それぞれ「sudo systemctl status」の実行結果で「Active: active (running) since 〜」と表示されていれば起動成功です。


  # 万が一Mysqlを落としてしまった場合は、以下のコマンドで起動させてください。
  $ sudo systemctl start mysqld
  $ sudo systemctl status mysqld
  ```
  再度、Vagrantfileに設定したIPアドレスへアクセスして、laravelのwelcome画面が表示されれば成功です。⑦へ進んでください。
  「403 Forbidden」のエラーが発生した場合は⑥を実施後⑦へ進んでください。
  <br />

- ⑥ extra：アクセスをしたら「403 Forbidden」のエラーが発生した場合
この場合、「SElinux」という機能によって発生しているエラーのため、SElinuxの設定を変更します。
  <br />

  実行コマンド：
  ```
  $ sudo vi /etc/selinux/config
  # ファイルの内容が表示されたら「i」キーを押して「INSERT」モードに切り替えてください。

  # This file controls the state of SELinux on the system.
  # SELINUX= can take one of these three values:
  # enforcing - SELinux security policy is enforced.
  # permissive - SELinux prints warnings instead of enforcing.
  # disabled - No SELinux policy is loaded.
  SELINUX=enforcing
  # ↓ 以下の通り編集
  SELINUX=permissive

  # 編集が終わりましたら、「esc」キーを押した後に「:wq」と入力してEnterを押してください。

  $ exit # ゲストOSからログアウト

  $ vagrant reload # vagrant 再起動
  $ vagrant ssh # ゲストOSへログイン

  $ sudo systemctl start nginx # nginx起動
  $ sudo systemctl start php-fpm # php-fpm起動
  ```
  SElinux自体を止めてしまうと、環境によっては問題が発生してしまうため、SEliunkを止めずに、何かエラーが発生した時にログを記録する設定へ変更しております。
  再度、Vagrantfileにて設定したIPアドレスにアクセスして、laravelのwelcome画面が表示されるか確認してください。

  <br />

- ⑦ 認証機能の実装
laravelに認証機能を実装します。
  実行コマンド：
  ※以下のコマンドはlaravel 6.0に対応した方法です。
  ```
  $ composer require laravel/ui "^1.0" --dev
  $ php artisan ui vue --auth
  ```
  <br />

  再度ブラウザでアクセスし、laravelのwelcome画面右上に「Login」「Register」と表示されていれば成功です。<br />
  ブラウザでの表示が確認できましたら、
  ユーザ登録、ユーザログイン、ユーザログアウトの機能を試してエラーが発生しないことを確認してください。
  <br />

## 参考(Giztech以外)
- [vagrantでNginx+PHP+php-fpmの環境を立ち上げる]('https://qiita.com/wjtnk/items/05446949a2d6018132b9')
- [Vagantを使ってLaravelを動かす]('https://qiita.com/hot-and-cool/items/076c0bb2690c5241ebcf')
- [Vagrantの重要ファイル「Vagrantfile」の読み込みとその周りの挙動をちょっと細かく追ってみた]('https://at-grandpa.hatenablog.jp/entry/2013/10/01/015501')
- [ReadDouble 認証6.x]('https://readouble.com/laravel/6.x/ja/authentication.html')
- [Homestead の PHPバージョンを変更する方法]('https://qiita.com/shosho/items/ad9fa1625e9ff86d3b43')
<br />

## 環境構築の所感
#### 1.コマンドをどこで実行するのか
&nbsp;&nbsp;環境構築に当たって様々なコマンドを実行しますが、「どのOS上」でかつ「どこのディレクトリ」で実行するかを正確に把握しないと、コマンドが受け付けられずエラーが発生してしまいます。
&nbsp;&nbsp;例えば、「php artisan migrate」を実行する場合、laravelにて用意したマイグレーションファイルを基に、データベースへマイグレーションを実行することが、このコマンドの役割となります。なので、laravelプロジェクトの中、かつデータベースを用意しているOS上で実行すると予測することができます。そうなると、今回の場合、ゲストOS上で作成（もしくは複製）したlaravelのディレクトリで実行すると考えることができます。
&nbsp;&nbsp;正確に把握するためには手順に沿ってコマンドを実行していくことは大切ですが、「コマンドの役割」を把握することによって、酷似したディレクトリが存在したり、どのOS上で実行すればいいかわからなくなったりした際に、回答に辿り着くヒントになると実感しました。
<br />

#### 2. 繰り返し確認の大切さ
&nbsp;&nbsp;環境構築を行っていると、些細なミスが結構先の工程で影響を及ぼす印象がありました。実行してすぐに結果が分かればいいですが、設定変更のみ行うケース、起動コマンドのみ実行して結果は返ってこないケースなど、自分が実行した内容が正確かどうかがすぐにわからないケースが多くあります。（環境構築に限った話ではないと思いますが）
&nbsp;&nbsp;例えば、nginxの設定の際に、documentrootのパスを一文字でも間違えると、laravel起動時に「file not found」と表示され、laravelのwelcome画面が表示されなくなります。しかし、手順書通りに進めていくと、そこが原因だと分かるまでにかなりの工程を踏むことになります。今回の場合は、laravel/publicが見つかってないこと、権限のエラーが発生していないことから、パスの記載ミスという予測が立てられますが、場合によっては、どこの工程が原因のエラーなのか予測を立てるまでに膨大な時間がかかってしまいます。
&nbsp;&nbsp;手順書外のエラーが発生することも結構ありますが、まず、手順書内の記載と異なることを実行していないか確認すること、そして、サービスを起動したらそのサービスのステータスを確認すること、この2つは癖づけていく必要があると思いました。
<br />

#### 3. 無知の目線
&nbsp;&nbsp;環境構築手順書作成の条件に、「他の人が手順書を見て全く同じ環境を構築できる」という条件があります。それは「環境構築についてほとんど無知でも見ればとりあえず作れる」というレベルの手順書だと考えております。そのためには、①自分の当たり前を疑うこと。②なるべく使用者の意図が介在しないようにすること。の2点が大切だと考えました。
&nbsp;&nbsp;例えば、自分はコマンドを実行する場所についてが考えられます。（①）私の場合、前後の状況や実行コマンドから想像して予測を立てて、ネットの情報で裏を取ることによって進めることができましたが、殆ど無知の場合、ゲストOSかホストOSか、どこのディレクトリか、ということを考えること自体がハードルの高い作業になる可能性があります。なので自分基準ではなくて、初めて見る人を想像して組み立てました。（②）一方、コマンドやサービスの役割はそこまで詳しく記述しませんでした。詳しく記述することによって、使用者側がこちらの意図しない予測を立てて手順書を逸脱する可能性があると考えたためです。
&nbsp;&nbsp;とは言っても、手順書によってその目的は異なってくると思いますので、今回が正解とは考えておりません。しかし、自分の基準を外して、別の人の目線を想像することは、プログラミングにおいても大切な要素（可読性、保守性に繋がる）だと思いますので、今回の経験は今後の糧になっていくと考えております。（大変ですが）