# バージョン一覧
|  PHP  |  Nginx  |  MySQL  |  Laravel  |  OSのバージョン  |
| ----- | ------- | ------- | --------- | -------------- |
|  TD   |  TD     |  TD     |  TD       |  TD            |


# どういう流れで環境構築したか

## 目次
以下で具体的に解説をしていきます。
此方の項目でイメージを掴みましょう。
- a
- a
- a
- a
- a
- a
- a
- a


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
変更点③は、ホストOSとゲストOSのディレクトリ内をリアルタイムで同期するための設定です。
```
# 変更点①
config.vm.network "forwarded_port", guest: 80, host: 8080

# 変更点②
config.vm.network "private_network", ip: "192.168.33.19"

# 変更点③
config.vm.synced_folder "../data", "/vagrant_data"
# ↓ 以下に編集
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```

## Vagrant プラグインのインストール
Vagrantへプラグイン(拡張機能)をインストールするコマンド
※ 今回は、初めに追加したBoxの中にインストールされているGuest Additionsというもののバージョンを、VirtualBoxのバージョンに合わせて最新化してくれるプラグイン
```
vagrant plugin install vagrant-vbguest
```
インストールの完了を確認するコマンド
```
vagrant plugin list
```

## Vagrantを使用してゲストOSの起動
Vagrantを起動するコマンド
※ aVagrantfileがあるディレクトリにて実行
※ 初回の起動には、時間がかかります。
```
vagrant up
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
sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.3.rpm
sudo rpm -Uvh remi-release-7.rpm
sudo yum -y install --enablerepo=remi-php72 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
php -v
```

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
$ sudo systemctl start firewalld.service
$ sudo firewall-cmd --add-service=http --zone=public --permanent

# 新たに追加を行ったのでそれをファイヤーウォールに反映させるコマンドも合わせて実行します
$ sudo firewall-cmd --reload
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




## 5
## 5




### grep
###
###




## 条件
1．Vagrant を使用すること
2．ipは 192.168.33.19 とすること
3．OSは CentOS7
4．DBは MySQL5.7
5．Webサーバは Nginx
6．PHPのバージョンは 7.3
7．Laravelのバージョンは 6.0

スタートは vagrant用ディレクトリの作成
ゴールは インストールしたLaravelプロジェクトにログイン機能を実装して実際にログインすること


# 環境構築の所感
- 1
- 2
- 3
- 4
- 5


# 参考サイト
[Gizumoのホームページはこちら](https://gizumo-inc.jp/)