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