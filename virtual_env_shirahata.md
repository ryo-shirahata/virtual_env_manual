# バージョン一覧
|  PHP  |  Webサーバ  |  MySQL  |  Laravel  |  OSのバージョン  |
| ----- | ---------- | ------- | --------- | -------------- |
|  7.3  |   Nginx    |   5.7   |    6.0    |    CentOS7     |



# どういう流れで環境構築したか

## 目次
以下で具体的に解説をしていきます。
此方の項目でイメージを掴みましょう。
- Vagrantの作業ディレクトリを用意
- Vagrantfileの編集
- Vagrant プラグインのインストール
- Vagrantを使用してゲストOSの起動し、ログイン

- パッケージをインストール
- PHPのインストール
- composerのインストール

- Laravelアプリケーションのコピー作成
- Webサーバのインストール
  今回はNginxを使用します。

- Apacheのインストール
- 設定ファイルの編集
- Apacheの起動
- ファイヤーウォールの設定
- 画面表示の確認
- データベースのインストール
- データベースの作成
- 動作確認

- Nginxのインストール
- nginxのconfファイルの編集
- php-fpmのconf編集
- Nginxの起動
- 権限の付与
- 動作確認


## Vagrantの作業ディレクトリを用意する
任意のディレクトリ下へ作業ディレクトリを作成します。
今回はホームディレクトリで作成します。
※ 作成したディレクトリを移動させてしまうと、ゲストOSへのログインが上手く行かなくなる場合があるので注意。
```
mkdir vagrant_test
```

作成した作業ディレクトリへ移動
```
cd vagrant_test
```

vagrantを使用するコマンド
※ LinuxのCentOSのバージョン7を指定
```
vagrant init centos/7
```

## Vagrantfileの編集
以下、三箇所を編集します。

変更点①は、# が付いているのを外します。
変更点②は、# が付いているのを外し、ipアドレスを変更します。
変更点③は、# が付いているのを外し、ホストOSとゲストOSのディレクトリ内をリアルタイムで同期するための設定です。
```
# 変更点① (26行)
config.vm.network "forwarded_port", guest: 80, host: 8080

# 変更点② (35行)
config.vm.network "private_network", ip: "192.168.33.19"

# 変更点③ (46行)
config.vm.synced_folder "../data", "/vagrant_data"
# ↓ 以下に編集
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```

### ポート
OSがデータ通信を行うために使用する港やドアといったイメージです。
今回はホストOSのポート8080番を使用します、起動が上手くいかない場合は既に8080番が使用されている可能性があります。

### ポートフォワーディング
自らの特定のポート番号への通信を別のアドレスの特定のポートへ自動的に転送することです。
ここではゲストOS（Vagrant）の80番の通信をホストOS（MacやWindowsなど）の8080へ転送する設定をしています。

### プライベートネットワーク
システムの内部での通信のために用いられるコンピュータネットワークのことです。
システム外部から内部のプライベートネットワークへの通信はできません。
過去にVagrantを使用したことがありプライベートネットワークIP 192.168.33.19 が既に使われている場合、
起動に失敗してしまうため192.168.55.19 などへ変更しましょう。


## Vagrant プラグインのインストール
Vagrantへプラグイン(拡張機能)をインストールするコマンド
今回は、初めに追加したBoxの中にインストールされているGuest Additionsというもののバージョンを、VirtualBoxのバージョンに合わせて最新化してくれるプラグイン
```
vagrant plugin install vagrant-vbguest
```
インストールの完了を確認するコマンド
```
vagrant plugin list
```
vagrant-vbguest以外にも 起動しているゲストOSの状態を全て同時に確認できるようにしてくれる vagrant-global-status、
環境構築中のゲストOSの状態を保存・巻き戻しができる sahara といったプラグインがあります。


## Vagrantを使用してゲストOSの起動
Vagrantを起動するコマンド
Vagrantfileがあるディレクトリにて実行
初回の起動には、時間がかかります。
```
vagrant up
```

### mountedエラーが発生した場合
以下のようなエラーが発生する場合があります。
```
The following SSH command responded with a non-zero exit status.
Vagrant assumes that this means the command failed!

umount /mnt

Stdout from the command:

Stderr from the command:

umount: /mnt: not mounted
```
以下の記事を参考にしてください。
[参考記事](https://qiita.com/mao172/items/f1af5bedd0e9536169ae)

### 仮想マシンの再起動
以下コマンドで一度、起動停止をして再度起動確認をしましょう。
```
vagrant halt
```
Vagrantの起動
```
vagrant up
```
以下のようなログが確認出来れば次へ進みましょう。
```
[default] GuestAdditions 6.0.14 running --- OK.
```


## ゲストOSへのログイン
作成したディレクトリで下記のコマンドを実行
```
vagrant ssh
```
以下のような表記が確認できればゲストOSにログインしていることになります。
```
Welcome to your Vagrant-built virtual machine.
[vagrant@localhost ~]$
```
※ ログアウトの際は、
```
exit
```
コマンドを使用してください。

## vagrant情報の確認
以下のコマンドで確認
```
vagrant ssh-config
```

## ホストOS・ゲストOSの見極め
ホストOS
```
ユーザー名noMacbook:~ ユーザー名$
```
ゲストOS
```
[vagrant@localhost ~]$
```



## パッケージをインストール
グループパッケージをインストールするコマンド
※ gitなどの開発に必要なパッケージを一括でインストール
```
sudo yum -y groupinstall "development tools"
```

### sudoコマンド
rootユーザー(システム管理者)の権限を借りるコマンドです。
ゲストOSにログインした場合、ログインユーザーはvagrantとなります。
vagrantというユーザーは、rootユーザーによって作成されたユーザーであり、
rootユーザーにしか許されていない操作を実行する際には、rootユーザーの権限を一時的に借りる必要があります。
通常時は一般ユーザーで作業を行い、必要なときだけsudoコマンドを使用してrootユーザーの権限を借りるようにしましょう。

### yumコマンド
CentOSを代表とするLinuxOSのRedHat系ディストリビューションと呼ばれる種類のOSで使用されているパッケージ管理ツール・コマンドです。
Macの brew に近いイメージです。
#### -yオプション
インストール実行途中に yes/no と聞かれた場合に全て自動でyesと回答してくれます。
大量のパッケージをインストールしようとした場合、個々のパッケージのインストールごとに
yes/no の入力を求められてスクリプトが停止してしまい面倒であるため、忘れずに付けるようにしましょう。

### groupinstall
まとめてパッケージのインストールを行うことが可能です。
他のグループパッケージでも対応可能です。


## PHPのインストール
yumコマンドを使用してPHPをインストールした場合、古いバージョンのPHPがインストールされてしまいます。
今回はバージョン7.3でインストールする為、外部パッケージツールをダウンロードして、そこからPHPをインストールしていきます。
```
sudo yum -y install epel-release wget
sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
sudo rpm -Uvh remi-release-7.rpm
sudo yum -y install --enablerepo=remi-php72 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
php -v
```
バージョンが表示されれば問題ありません。

### 拡張モジュール
PHPのインストールと同時に、PHPアプリケーションを動かす上で必要となるモジュール(拡張機能)をインストールしています。
PDO接続行うために、php-mbstring は PHPで日本語などのマルチバイト文字を扱うために必要となります。

### wget
URLを指定することで指定先URLのファイルをダウンロードすることが可能です。

### rpm
Linuxディストリビューション(CentOSやFedora)内で使用できるパッケージ管理ツールです。
yumと異なる点としては、yumがパッケージごとの **依存関係** を管理してくれることに対して、
rpmはパッケージ自体を管理することはできますがその管理対象のパッケージの依存関係までは管理することはできません。


## composerのインストール
PHPのパッケージ管理ツールであるcomposerをインストール
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

# どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動を行います
sudo mv composer.phar /usr/local/bin/composer
composer -v
```

### Composer
Composerは RedHat系のLinuxディストリビューションで使用できるOS用パッケージ管理ツールの yum とは異なり、PHPのパッケージの **依存関係** を管理・解決するツールです。


## 依存関係とは
依存関係管理・解決ツールがない場合、
パッケージAを使用するために必要なパッケージB、パッケージC、
さらにパッケージB、パッケージCが必要とするパッケージD、パッケージEを自分で探して個別にインストールしなければいけなくなります。
依存関係管理・解決ツールは、パッケージAをインストール時に、複雑な依存関係にあるパッケージをすべて同時にインストールしてくれます。



## Laravelアプリケーションのコピー作成
始めに作成したディエクトりへ移動し、laravel_appのコピーを配下へ作成してください。
※ ゲストOSにログインしている人は一旦 exit コマンドを実行してログアウトしてください。
```
cd vagrant_test
cp -r laravel_appディレクトリまでの絶対パス ./
```
私の場合 ホーム/gizumo/laravel_app ですので以下コマンドです。
```
cp -r ~/gizumo/laravel_app ./
```


## Webサーバのインストール
今回は Apache と nginx の二つを練習します。
本件は nginx のインストールですので以下は飛ばして構いません。

### Webサーバとは
Webサーバとはユーザーからのリクエストの内容に応じてアプリケーションサーバに内容を伝達、またアプリケーションサーバが実行した結果をユーザーに返す役割を担っています。


## Apacheのインストール
WebサーバソフトウェアであるApacheをインストールします。
※ ゲストOSにログインしていない方は vagrant_test ディレクトリにて vagrant ssh コマンドを実行してログインしてください。
バージョンの確認まで出来ていれば大丈夫です。
```
sudo yum -y install httpd
httpd -v
```


## Apacheの設定ファイルを編集
編集には vi コマンドを使用します。
```
sudo vi /etc/httpd/conf/httpd.conf
```
以下、三箇所を編集します。

変更点①は、ルートを修正します。
変更点②は、ルートを修正し、.htaccess というファイルの使用を有効化する設定をします。
変更点③は、vagrantユーザーでApacheを実行できるように設定します。
```
# 省略

# 変更点①
DocumentRoot "/var/www/html"
# ↓ 以下に編集
DocumentRoot "/vagrant/laravel_app/public"

# 省略

# 変更点②
<Directory "/var/www/">
  AllowOverride None
  Require all granted
</Directory>
# ↓ 以下に編集
<Directory "/vagrant/laravel_app/public">
  AllowOverride All
  Require all granted
</Directory>

# 省略

# 変更点③
User apache
Group apache
# ↓ 以下に編集
User vagrant
Group vagrant
```

### DocumentRoot
192.168.33.19 に対してリクエストがあった場合に、サーバ内のどこのディレクトリを参照するか決める指定です。

### Directory
指定したディレクトリに対して個別の設定をすることができます。

### AllowOverride
Directoryに指定したディレクトリ内の .htaccess というファイルの使用を有効化する設定です。
.htaccess というファイルを配置することでディレクトリごとにWebサーバの設定を変更することができます。

### Require
Directoryに指定したディレクトリへのアクセスを制御します。
今回は all granted と指定されているので全てのipアドレスからのアクセスを許可しています。

### User, Group
Apacheを起動するユーザーとグループを指定します。
今回は vagrant というユーザーでゲストOS内にログインしているため apache を vagrant に変更してvagrantユーザーでApacheを実行できるように設定しました。


## Apacheの起動
以下のコマンドを実行して起動をします。
```
sudo systemctl start httpd
```
以下のコマンドでApacheが正常に起動しているか確認してください。
```
sudo systemctl status httpd
```
以下、実行結果の例になります。
```
 ● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since 土 XXXX-XX-XX XX:XX:XX UTC; 2s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 16536 (httpd)
   Status: "Processing requests..."
   CGroup: /system.slice/httpd.service
           ├─16536 /usr/sbin/httpd -DFOREGROUND
           ├─16537 /usr/sbin/httpd -DFOREGROUND
           ├─16538 /usr/sbin/httpd -DFOREGROUND
           ├─16539 /usr/sbin/httpd -DFOREGROUND
           ├─16540 /usr/sbin/httpd -DFOREGROUND
           └─16541 /usr/sbin/httpd -DFOREGROUND

 X月 XX XX:XX:XX localhost.localdomain systemd[1]: Starting The Apache HTTP Server...
 X月 XX XX:XX:XX localhost.localdomain httpd[16536]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using ... message
 X月 XX XX:XX:XX localhost.localdomain systemd[1]: Started The Apache HTTP Server.
 Hint: Some lines were ellipsized, use -l to show in full.
```


## ファイヤーウォールの設定
起動状態のままホストOS側からアクセスできるようにしてあげましょう。
ファイヤーウォールに対してこの80ポートを経由したhttp通信によるアクセスを許可するためのコマンドを実行します。
```
# ファイヤーウォールの起動
sudo systemctl start firewalld.service
sudo firewall-cmd --add-service=http --zone=public --permanent

# 新たに追加を行ったのでそれをファイヤーウォールに反映させるコマンドも合わせて実行します
sudo firewall-cmd --reload
```

### ファーヤーウォール
ネットワークの通信において、その通信をさせるかどうかを判断し許可するまたは拒否する仕組み

### ポート
httpという通信を行うためのポートと呼ばれる窓口番号


## ブラウザの確認
http://192.168.33.19 と入力し確認してみてください。
もしまだ表示できないようであれば、一度以下のコマンドを実行してください。
```
sudo systemctl restart httpd
```


## それでも表示されない場合
Laravelのwelcomeページが表示されず、 Forbidden 403 というエラーが出た場合の対処法を以下に記載します。
viエディタを使用してSELinuxの設定を変更します。
「SELinux コンテキスト」の不一致によりエラーが出ているので、SELinuxを無効化します。
```
sudo vi /etc/selinux/config
```
下記の箇所を編集してください。
```
SELINUX=enforcing
# ↓ 以下に編集
SELINUX=disabled
```
上記で以下が非常に間違え易く危険な為に注意してください。
場合によって**カーネルパニック**を引き起こしてしまいます。
```
SELINUXTYPE
```
設定を反映させるためにゲストOSを再起動する必要があるので、ゲストOSをから一度ログアウトして下記コマンドを実行してください。
```
exit
vagrant reload
```
リロードが完了したら再度ゲストOSにログインしましょう。
```
vagrant ssh
```
再度Apacheを起動します。
```
sudo systemctl start httpd
```
再度ブラウザの確認してください。


## データベースのインストール
今回インストールするデータベースはMySQLとなります。versionは5.7を使用します。
centos7は、デフォルトでmariaDBというデータベースがインストールされていますが、
MariaDBはMySQLと互換性があるDBなので気にせず、MySQLのインストールを進めていきます。
rpmに新たにリポジトリを追加し、インストールを行います。
```
sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
sudo yum install -y mysql-community-server
mysql --version
```
versionの確認ができましたらインストール完了です。
次にMySQLを起動し接続を行います。
```
sudo systemctl start mysqld
mysql -u root -p
Enter password:
```
MacあるいはWindowsにMySQLをインストールしたときは、
何も入力せずに接続が可能だったかと思いますが今回はデフォルトでrootにパスワードが設定されてしまっています。
まずはpasswordを調べ、接続しpassswordの再設定を行っていく必要があります。
```
sudo cat /var/log/mysqld.log | grep 'temporary password'
2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
```
hogehoge と記載されている箇所に存在するランダムな文字列がパスワードとなります。
※ 今回は、比較的簡単な方法でパスワードの再設定を行いますが、セキュリティ的によろしくはないため本番環境と呼ばれる環境でこの方法で再設定するのは避けてください。

### grep
grep は 左辺の実行結果の中から、grepで指定した文字列と一致する行や文字列を持つファイルやディレクトリのみを出力するコマンドです。
以下のコマンドと組合わせて右辺に使用します。

### |
| はパイプラインといって 左辺 | 右辺 と指定してコマンドを実行することで、左辺の実行結果を右辺に引き渡して右辺を実行することができます。

## パスワードの変更
先程出力したランダム文字列をコピー後、再度以下のコマンドを実行し、パスワード入力時にペーストしてください。
```
mysql -u root -p
Enter password:
mysql >
```
次に接続した状態でpasswordの変更を行います。
```
mysql > set password = "新たなpassword";
```
新たなpasswordには、必ず大文字小文字の英数字 + 記号かつ8文字以上の設定をする必要があります。
MySQL5.7のパスワードポリシーは厳格で開発段階では手間となってしまうため、
以下の設定を行いシンプルなパスワードに初期設定できるようにMySQLの設定ファイルを変更します。
```
sudo vi /etc/my.cnf
```
```
# 省略

[mysqld]

# 省略

# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# 下記の一行を追加
validate-password=OFF
```
編集後はMySQLサーバの再起動が必要です。
```
sudo systemctl restart mysqld
```
再度MySQLにログインしてパスワードの初期設定を行えば、簡単なパスワードで登録ができます。
以上で、MySQLの導入と設定が完了となります。


## データベースの作成
実際にLaravelのTodoアプリケーションを動かす上で使用するデータベースの作成を行います。
```
mysql > create database laravel_app;
```
Query OKと表示されたら作成は完了となります。


## Laravelを動かす
laravel_appディレクトリ下の .env ファイルの内容を以下に変更してください。
```
DB_PASSWORD=
# ↓ 以下に編集
DB_PASSWORD=登録したパスワード
```
laravel_appディレクトリに移動してマイグレーションを実行します。
```
php artisan migrate
```
マイグレーションが問題なく実行できた後、
ブラウザ上でユーザー登録ができればローカルで動かしていたLaravelを仮想環境上で全く同じように動かすことができたということになります。


## Nginxを用いた動作確認
Nginx は Apache と同様にWebサーバです。
本来一つのサーバではApacheかNginxどちらか片方しか使用しませんが、今回は練習としてApache、Nginx両方のパターンで試してみましょう。
Nginxの方がApacheよりもWebサーバとしてのシェア率が高く、使用する機会も多いと思われますので、設定方法について学んでいきます。


## Apacheの起動状態を停止
まず起動状態であるApacheを一度停止します。
```
sudo systemctl stop httpd
```
以下コマンドで停止の確認をします。
```
sudo systemctl status httpd
```

### モジュールのインストール
Nginxを使用してPHPアプリケーションを動かす上で必ず *php-fpm* というPHPのモジュールが必要になります。
PHPインストールの箇所で同時にインストールを済ませています。
Apacheは単体で機能し、Nginxは php-fpmというモジュールとセットで機能するということを覚えてください。
これはApacheが mod_php というモジュールを使用してApacheの中でPHPを実行する環境を整えておりアプリケーションサーバの役割を兼ねているためです。


## Nginxのインストール
Nginxの最新版をインストールしていきます。
以下はゲストOS側での操作になるので注意しましょう。
未ログインの肩は以下コマンドを参照してください。
```
vagrant ssh
```
viエディタを使用して以下のファイルを作成します。
```
sudo vi /etc/yum.repos.d/nginx.repo
```
以下を記述してください。
```
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```
保存して以下のコマンドを実行しNginxのインストールを実行します。
```
sudo yum install -y nginx
nginx -v
```
バージョンの確認が出来れば、以下コマンドで起動しましょう。
```
sudo systemctl start nginx
```
ブラウザにて http://192.168.33.19 と入力し、画面表示を確認してください。
**Welcome to nginx!** と表示されていれば大丈夫です。


## Laravelを動かす(nginxのconf編集)
Apache同様に設定ファイルを編集します。
php-fpmにも設定ファイルが存在しているのでこちらも編集を行います。
```
sudo vi /etc/nginx/conf.d/default.conf
```
編集範囲が広いので随時、保存と確認をしながら作業をしてください。
```
server {
  listen       80;
  server_name  192.168.33.19; # Vagranfileでコメントを外した箇所のipアドレスを記述してください。
  # ApacheのDocumentRootにあたります
  root /vagrant/laravel_app/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  # 省略

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }

  # 省略
```
Nginxの設定ファイルの変更は、以上です。


## Laravelを動かす(php-fpmのconf編集)
次に php-fpm の設定ファイルを編集していきます。
```
sudo vi /etc/php-fpm.d/www.conf
```
変更箇所は以下になります。
```
;24行目近辺
user = apache
# ↓ 以下に編集
user = nginx

group = apache
# ↓ 以下に編集
group = nginx
```
設定ファイルの変更に関しては、以上となります。
では早速起動しましょう(Nginxは再起動になります)。
```
sudo systemctl restart nginx
sudo systemctl start php-fpm
```
再度ブラウザにて、 http://192.168.33.19 を入力して確認してください。
画面は表示されますが、以下のようなLaravelのエラーが表示されると思います。
```
The stream or file "/vagrant/laravel_app/storage/logs/laravel.log" could not be opened: failed to open stream: Permission denied
```
ファイルとディレクトリの実行 user と group に nginx が許可されていないため起きているエラーです。


## nginxへ権限の付与
以下のコマンドを実行して nginx というユーザーでもログファイルへの書き込みができる権限を付与してあげましょう。
```
cd /vagrant/laravel_app
sudo chmod -R 777 storage
```


## パーミッション
**パーミッション** とはファイルやディレクトリに対する **操作権限** です。
ls -la コマンドを実行して実際にどのような操作権限が付与されているか確認してみます。
```
cd /vagrant/laravel_app/storage
ls -la

合計 0
drwxrwxrwx 1 vagrant vagrant 160  2月 28 11:48 .
drwxr-xr-x 1 vagrant vagrant 928  2月 28 11:48 ..
drwxrwxrwx 1 vagrant vagrant 128  2月 28 11:48 app
drwxrwxrwx 1 vagrant vagrant 224  2月 28 11:48 framework
drwxrwxrwx 1 vagrant vagrant 128  2月 28 11:48 logs
```
上記中で、パーミッションを指しているのが **drwxrwxrwx** の部分です。
一番左の **d** はディレクトリであることを意味しており、ファイルの場合は - が表示されます。

パーミッションは残りのrwxrwxrwxの部分です。
アルファベットのrとwとxはそれぞれ
```
r : 読み
w : 書き
x : 実行
```
を意味しており、3文字ごとにそれぞれ以下の対象についての権限を表しています。
```
rwx : rwx : rwx
所有者(u)のアクセス権 : グループユーザー(g)のアクセス権 : その他のユーザー(o)のアクセス権
```
許可を与えているものはそれぞれ rwx のアルファベットで表現され、許可が与えられていないものは - という表記になります。
ターミナル上でも、アプリケーション実行時でも Permission denied というエラーが表示された場合は、
大抵がこの権限の問題ですので適切に権限やユーザー・グループの変更を行う必要があります。

### chmod
chmodはファイルやディレクトリのパーミッションを変更するコマンドです。

### chown (CHange OWNer)
chown は パーミッションユーザーとグループを変更するコマンドです。

### 8進数での表現方法
|  r  |  w  |  x  |  ２進数  |  ８進数  |
| --- | --- | --- | ------- | ------- |
|  0  |  0  |  0  |   000   |    0    |
| --- | --- | --- | ------- | ------- |
|  0  |  0  |  1  |   001   |    1    |
| --- | --- | --- | ------- | ------- |
|  0  |  1  |  0  |   010   |    2    |
| --- | --- | --- | ------- | ------- |
|  0  |  1  |  1  |   011   |    3    |
| --- | --- | --- | ------- | ------- |
|  1  |  0  |  0  |   100   |    4    |
| --- | --- | --- | ------- | ------- |
|  1  |  0  |  1  |   101   |    5    |
| --- | --- | --- | ------- | ------- |
|  1  |  1  |  0  |   110   |    6    |
| --- | --- | --- | ------- | ------- |
|  1  |  1  |  1  |   111   |    7    |
上記図を参照して、
chmod 755 と指定した場合は rwxr-xr-x
逆に rwxr-x--x の権限を付与したい場合は chmod 751 と指定します。


## 権限の確認
意図的にアプリケーション実行エラーを起こしてlaravel.logに画面と同じエラーが表示されているか見てみましょう。
```
cd /vagrant/laravel_app
vi routes/web.php
```
以下のように変更しましょう。
```
Route::get('/', function () {
    return view('welcome');
});

↓ # ; を消します

Route::get('/', function () {
    return view('welcome')
});
```
編集が完了しましたら次は以下のコマンドを実行します。
```
tail -f storage/logs/laravel.log
```
では、再度 http://192.168.33.19 のURLにアクセスしてみます。
```
syntax error, unexpected '}', expecting ';'
```
上記の syntax error が確認出来たかと思います。
確認が完了しましたら、Ctrl + c でtailの実行モードを終了しましょう。
では変更した routes/web.php の内容を元に戻して再度 http://192.168.33.19 にアクセスして正常にLaravelのWelcome画面の表示をしてください。

### tail
指定したファイルの末数行を出力するコマンドです。
-f オプションを付けて実行した場合は、ファイルの変更を監視し変更があれば常時出力するモードとなります。


## エラーの確認
Apacheの設定ファイルに記述ミスなどが合った場合に CentOSでは /etc/httpd/logs/error_log というファイルにエラーの内容が書き込まれます。
Nginxの場合は /var/log/nginx/error.log というファイルにエラー内容が書き込まれるようになっています。

このように画面に出力されるエラー内容だけではなくログファイルに書き込まれたエラー内容を見て、
エラーやバグを解消していくことも多いので tail コマンドの使用には慣れましょう。

以上でVagrantを使用した仮想環境構築は終了となります。


## LAMP環境
**LAMP環境** とは、Webアプリケーションを開発するための、代表的なシステムの頭文字を取って命名されています。

- L inux
- A pache
- M ySQL（または、MariaDBなど）
- P HP（または、Pythonなど）

今回のServer Lessonでは全て利用しており、LAMP環境での環境構築ということになります。
LAMP環境は全て **オープンソースソフトウェア** を活用しているため、
誰でも自由かつ無料で開発環境を構築し、Webアプリケーションの開発に取り掛かることができます。
コストが抑えられる点や汎用性が高い点などから、多くの企業で利用されている開発環境だと言えます。

また、他にも XAMPP や MAMP といった環境構築パッケージもあります。

### XAMPP
- X クロスプラットフォーム
- A pache
- M ySQL（または、MariaDBなど）
- P HP
- P erl

**クロスプラットフォーム** とは、macOSやWindowsなどの様々なOSでも同じ仕様で動作するプログラムのことです。
近年は動作環境が多様化しているため、クロスプラットフォームでの開発環境が注目されています。

### MAMP
- M acintosh
- A pache
- M ySQL（または、MariaDBなど）
- P HP（または、Pythonなど）

MAMPはMacに特化したパッケージです。
無料版と有償版があり用途によっては、拡張版である有償版をインストールする必要があります。


### 以下はApacheの際に操作した内容です。
nginxからインストールした場合は以下を参照してください。

## ファイヤーウォールの設定
起動状態のままホストOS側からアクセスできるようにしてあげましょう。
ファイヤーウォールに対してこの80ポートを経由したhttp通信によるアクセスを許可するためのコマンドを実行します。
```
# ファイヤーウォールの起動
sudo systemctl start firewalld.service
sudo firewall-cmd --add-service=http --zone=public --permanent

# 新たに追加を行ったのでそれをファイヤーウォールに反映させるコマンドも合わせて実行します
sudo firewall-cmd --reload
```

### ファーヤーウォール
ネットワークの通信において、その通信をさせるかどうかを判断し許可するまたは拒否する仕組み

### ポート
httpという通信を行うためのポートと呼ばれる窓口番号


## ブラウザの確認
http://192.168.33.19 と入力し確認してみてください。
もしまだ表示できないようであれば、一度以下のコマンドを実行してください。
```
sudo systemctl restart httpd
```


## それでも表示されない場合
Laravelのwelcomeページが表示されず、 Forbidden 403 というエラーが出た場合の対処法を以下に記載します。
viエディタを使用してSELinuxの設定を変更します。
「SELinux コンテキスト」の不一致によりエラーが出ているので、SELinuxを無効化します。
```
sudo vi /etc/selinux/config
```
下記の箇所を編集してください。
```
SELINUX=enforcing
# ↓ 以下に編集
SELINUX=disabled
```
上記で以下が非常に間違え易く危険な為に注意してください。
場合によって**カーネルパニック**を引き起こしてしまいます。
```
SELINUXTYPE
```
設定を反映させるためにゲストOSを再起動する必要があるので、ゲストOSをから一度ログアウトして下記コマンドを実行してください。
```
exit
vagrant reload
```
リロードが完了したら再度ゲストOSにログインしましょう。
```
vagrant ssh
```
再度Apacheを起動します。
```
sudo systemctl start httpd
```
再度ブラウザの確認してください。


## データベースのインストール
今回インストールするデータベースはMySQLとなります。versionは5.7を使用します。
centos7は、デフォルトでmariaDBというデータベースがインストールされていますが、
MariaDBはMySQLと互換性があるDBなので気にせず、MySQLのインストールを進めていきます。
rpmに新たにリポジトリを追加し、インストールを行います。
```
sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
sudo yum install -y mysql-community-server
mysql --version
```
versionの確認ができましたらインストール完了です。
次にMySQLを起動し接続を行います。
```
sudo systemctl start mysqld
mysql -u root -p
Enter password:
```
MacあるいはWindowsにMySQLをインストールしたときは、
何も入力せずに接続が可能だったかと思いますが今回はデフォルトでrootにパスワードが設定されてしまっています。
まずはpasswordを調べ、接続しpassswordの再設定を行っていく必要があります。
```
sudo cat /var/log/mysqld.log | grep 'temporary password'
2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
```
hogehoge と記載されている箇所に存在するランダムな文字列がパスワードとなります。
※ 今回は、比較的簡単な方法でパスワードの再設定を行いますが、セキュリティ的によろしくはないため本番環境と呼ばれる環境でこの方法で再設定するのは避けてください。

### grep
grep は 左辺の実行結果の中から、grepで指定した文字列と一致する行や文字列を持つファイルやディレクトリのみを出力するコマンドです。
以下のコマンドと組合わせて右辺に使用します。

### |
| はパイプラインといって 左辺 | 右辺 と指定してコマンドを実行することで、左辺の実行結果を右辺に引き渡して右辺を実行することができます。

## パスワードの変更
先程出力したランダム文字列をコピー後、再度以下のコマンドを実行し、パスワード入力時にペーストしてください。
```
mysql -u root -p
Enter password:
mysql >
```
次に接続した状態でpasswordの変更を行います。
```
mysql > set password = "新たなpassword";
```
新たなpasswordには、必ず大文字小文字の英数字 + 記号かつ8文字以上の設定をする必要があります。
MySQL5.7のパスワードポリシーは厳格で開発段階では手間となってしまうため、
以下の設定を行いシンプルなパスワードに初期設定できるようにMySQLの設定ファイルを変更します。
```
sudo vi /etc/my.cnf
```
```
# 省略

[mysqld]

# 省略

# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# 下記の一行を追加
validate-password=OFF
```
編集後はMySQLサーバの再起動が必要です。
```
sudo systemctl restart mysqld
```
再度MySQLにログインしてパスワードの初期設定を行えば、簡単なパスワードで登録ができます。
以上で、MySQLの導入と設定が完了となります。


## データベースの作成
実際にLaravelのTodoアプリケーションを動かす上で使用するデータベースの作成を行います。
```
mysql > create database laravel_app;
```
Query OKと表示されたら作成は完了となります。


## Laravelを動かす
laravel_appディレクトリ下の .env ファイルの内容を以下に変更してください。
```
DB_PASSWORD=
# ↓ 以下に編集
DB_PASSWORD=登録したパスワード
```
laravel_appディレクトリに移動してマイグレーションを実行します。
```
php artisan migrate
```
マイグレーションが問題なく実行できた後、
ブラウザ上でユーザー登録ができればローカルで動かしていたLaravelを仮想環境上で全く同じように動かすことができたということになります。



# 環境構築の所感
- ディレクトリの位置やファイル名は特に意識が必要、設定ファイルへ反映させる際はコピペを推奨
- 操作する際は現在ゲストなのか、ホストなのか見極めてから動かす
- webサーバの起動状態を確認
- 設定ファイルなどを編集後はbuildだけでなく、起動停止をさせて再起動するなどが必要(Docker)
- 見るより動かした方が理解が早いことを痛感、不安な場合は確認必須
- 動作確認時にカーネルパニック(ブルースクリーンみたいなもの)を起こしてしまった。
  403 Forbiddenエラー解消時に SELINUXTYPE の方を disabled(無効化) してしまった。
  上記により vagrant ssh によるログインが不可能になりクラッシュしてしまった。



# 参考サイト
[Gizumoのホームページはこちら](https://gizumo-inc.jp/)