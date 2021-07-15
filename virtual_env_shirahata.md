# バージョン一覧
|  PHP  |  Nginx  |  MySQL  |  Laravel  |  OSのバージョン  |
| ----- | ------- | ------- | --------- | -------------- |
|  TD   |  TD     |  TD     |  TD       |  TD            |


# どういう流れで環境構築したか

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


###
###
###




## 5
## 5
## 5
## 5
## 5
## 5



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