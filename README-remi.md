# Remi パッケージとか SCL について調べてみた（初心者向け？）

php を入れる場合、いつもは CentOS 7.x と [webtatic リポジトリ](https://webtatic.com/) の組み合わせを使っています。ただ、OS は上げられないけど PHP はできれば最新にしたい的な案件に備えて、remi リポジトリの方も調べてみました。今回の対象のディストロは CentOS6.x です。単なる作業ログであり、知見はあんまりないかもしれないですが、yum / rpm コマンドの使い方など、初心者の方には参考になる部分もあるかもしれません。

## Remi リポジトリとは

Remi Collet 氏が個人で管理しているリポジトリです。[Software Collections](https://www.softwarecollections.org/en/) という仕組みを使用して、複数の PHP バージョンを共存する仕組みを提供してくれます。Software Collections は [Red Hat](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/6/html/developer_guide/sect-rhscl-overview) でも採用されており、これを使って PHP 以外にも Perl や Python などさまざまなコンポーネントが提供されています。

# つらつらと作業ログ

vagrant + VirtualBox で作業を行いました。

### centos/6 box で vagrant up した素の状態(2018/04/06)

```bash
[vagrant@cent69-bare ~]$ rpm -qa | grep -E '(php|release)'
centos-release-6-9.el6.12.3.x86_64
```

### 諸般の都合により、git と ansible を入れました。

```bash
[vagrant@cent69-bare ~]$ sudo yum install git epel-release
[vagrant@cent69-bare ~]$ sudo yum install ansible
[vagrant@cent69-bare ~]$ rpm -qa | grep -E '(php|release)'
epel-release-6-8.noarch
centos-release-6-9.el6.12.3.x86_64
```

（CentOS 6.x では、ansible が epel リポジトリにしかないのでここで入れてますが、ansible が不要な場合でも、remi は epel に依存するので結局入れることになります。）

### 標準の php を導入

まず、CentOS が提供している php を入れてみます。

```bash
[vagrant@cent69-bare ~]$ sudo yum install php
================================================================================
 Package            Arch        Version                      Repository    Size
================================================================================
Installing:
 php                x86_64      5.3.3-49.el6                 base         1.1 M
Installing for dependencies:
 apr                x86_64      1.3.9-5.el6_9.1              updates      124 k
 apr-util           x86_64      1.3.9-3.el6_0.1              base          87 k
 apr-util-ldap      x86_64      1.3.9-3.el6_0.1              base          15 k
 httpd              x86_64      2.2.15-60.el6.centos.6       updates      836 k
 httpd-tools        x86_64      2.2.15-60.el6.centos.6       updates       80 k
 mailcap            noarch      2.1.31-2.el6                 base          27 k
 php-cli            x86_64      5.3.3-49.el6                 base         2.2 M
 php-common         x86_64      5.3.3-49.el6                 base         530 k
```

php は 5.3.3 が標準リポジトリ(base)から落ちてきます。

```bash
[vagrant@cent69-bare ~]$ rpm -qa | grep -E '(php|release)' | sort
centos-release-6-9.el6.12.3.x86_64
epel-release-6-8.noarch
php-5.3.3-49.el6.x86_64
php-cli-5.3.3-49.el6.x86_64
php-common-5.3.3-49.el6.x86_64
```

### remi リポジトリを探す

```bash
[vagrant@cent69-bare ~]$ yum search remi
No matches found for: remi
```

remi は標準リポジトリ や epel の範囲からは見つかりません。Remi の本家 である https://rpms.remirepo.net/ から取ってくる必要があります。

remi を入れる前に、Remi の [Configuration Wizard](https://rpms.remirepo.net/wizard/) が便利なので見ておくとよいです。CentOS6 に php-5.6 と、場合によってはそれ以外のバージョンも同時に入れたいなぁという条件でご相談した結果は以下の通り：

![remi-wizard.png](https://github.com/hotta/doc-ja/raw/master/images/remi-wizard.png)


### remi リポジトリの導入

remi が依存する epel はすでに導入済みなので、ウィザードに従って remi を入れます。

```bash
[vagrant@cent69-bare ~]$ sudo yum install http://rpms.remirepo.net/enterprise/remi-release-6.rpm
[vagrant@cent69-bare ~]$ rpm -qa | grep -E '(php|release)' | sort
centos-release-6-9.el6.12.3.x86_64
epel-release-6-8.noarch
php-5.3.3-49.el6.x86_64
php-cli-5.3.3-49.el6.x86_64
php-common-5.3.3-49.el6.x86_64
remi-release-6.9-2.el6.remi.noarch
```

remi-release が増えました。この時点の yum リポジトリの設定ファイルを見てみます。

```bash
[vagrant@cent69-bare ~]$ ls /etc/yum.repos.d/
CentOS-Base.repo       CentOS-Vault.repo  remi-php70.repo  remi-safe.repo
CentOS-Debuginfo.repo  epel.repo          remi-php71.repo
CentOS-fasttrack.repo  epel-testing.repo  remi-php72.repo
CentOS-Media.repo      remi-php54.repo    remi.repo
```

なんかたくさん入ってますが、すべてが有効になっているわけではありません。現時点で有効なリポジトリは以下の通りです。

```bash
[vagrant@cent69-bare ~]$ yum repolist enabled
リポジトリー ID  リポジトリー名                                              状態
base             CentOS-6 - Base                                             6,706
*epel            Extra Packages for Enterprise Linux 6 - x86_64              2,475
extras           CentOS-6 - Extras                                              53
remi-safe        Safe Remi''s RPM repository for Enterprise Linux 6 - x86_64 2,328
updates          CentOS-6 - Updates                                          1,179
repolist: 22,741
```

『状態』欄に表示されているのは、リポジトリに含まれるパッケージの数です。

同様に、インストールはされているが無効になっているリポジトリを表示してみます。CentOS 6.x は歴史が長いので、リストも長いです。

```bash
[vagrant@cent69-bare ~]$ yum repolist disabled
リポジトリー ID             リポジトリー名
C6.0-base                   CentOS-6.0 - Base
C6.0-centosplus             CentOS-6.0 - CentOSPlus
C6.0-contrib                CentOS-6.0 - Contrib
C6.0-extras                 CentOS-6.0 - Extras
C6.0-updates                CentOS-6.0 - Updates
C6.1-base                   CentOS-6.1 - Base
C6.1-centosplus             CentOS-6.1 - CentOSPlus
C6.1-contrib                CentOS-6.1 - Contrib
C6.1-extras                 CentOS-6.1 - Extras
C6.1-updates                CentOS-6.1 - Updates
C6.2-base                   CentOS-6.2 - Base
C6.2-centosplus             CentOS-6.2 - CentOSPlus
C6.2-contrib                CentOS-6.2 - Contrib
C6.2-extras                 CentOS-6.2 - Extras
C6.2-updates                CentOS-6.2 - Updates
C6.3-base                   CentOS-6.3 - Base
C6.3-centosplus             CentOS-6.3 - CentOSPlus
C6.3-contrib                CentOS-6.3 - Contrib
C6.3-extras                 CentOS-6.3 - Extras
C6.3-updates                CentOS-6.3 - Updates
C6.4-base                   CentOS-6.4 - Base
C6.4-centosplus             CentOS-6.4 - CentOSPlus
C6.4-contrib                CentOS-6.4 - Contrib
C6.4-extras                 CentOS-6.4 - Extras
C6.4-updates                CentOS-6.4 - Updates
C6.5-base                   CentOS-6.5 - Base
C6.5-centosplus             CentOS-6.5 - CentOSPlus
C6.5-contrib                CentOS-6.5 - Contrib
C6.5-extras                 CentOS-6.5 - Extras
C6.5-updates                CentOS-6.5 - Updates
C6.6-base                   CentOS-6.6 - Base
C6.6-centosplus             CentOS-6.6 - CentOSPlus
C6.6-contrib                CentOS-6.6 - Contrib
C6.6-extras                 CentOS-6.6 - Extras
C6.6-updates                CentOS-6.6 - Updates
C6.7-base                   CentOS-6.7 - Base
C6.7-centosplus             CentOS-6.7 - CentOSPlus
C6.7-contrib                CentOS-6.7 - Contrib
C6.7-extras                 CentOS-6.7 - Extras
C6.7-updates                CentOS-6.7 - Updates
C6.8-base                   CentOS-6.8 - Base
C6.8-centosplus             CentOS-6.8 - CentOSPlus
C6.8-contrib                CentOS-6.8 - Contrib
C6.8-extras                 CentOS-6.8 - Extras
C6.8-updates                CentOS-6.8 - Updates
base-debuginfo              CentOS-6 - Debuginfo
c6-media                    CentOS-6 - Media
centosplus                  CentOS-6 - Plus
contrib                     CentOS-6 - Contrib
epel-debuginfo              Extra Packages for Enterprise Linux 6 - x86_64 - Debug
epel-source                 Extra Packages for Enterprise Linux 6 - x86_64 - Source
epel-testing                Extra Packages for Enterprise Linux 6 - Testing - x86_64
epel-testing-debuginfo      Extra Packages for Enterprise Linux 6 - Testing - x86_64 - Debug
epel-testing-source         Extra Packages for Enterprise Linux 6 - Testing - x86_64 - Source
fasttrack                   CentOS-6 - fasttrack
remi                        Remi's RPM repository for Enterprise Linux 6 - x86_64
remi-debuginfo              Remi's RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
remi-php54                  Remi's PHP 5.4 RPM repository for Enterprise Linux 6 - x86_64
remi-php55                  Remi's PHP 5.5 RPM repository for Enterprise Linux 6 - x86_64
remi-php55-debuginfo        Remi's PHP 5.5 RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
remi-php56                  Remi's PHP 5.6 RPM repository for Enterprise Linux 6 - x86_64
remi-php56-debuginfo        Remi's PHP 5.6 RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
remi-php70                  Remi's PHP 7.0 RPM repository for Enterprise Linux 6 - x86_64
remi-php70-debuginfo        Remi's PHP 7.0 RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
remi-php70-test             Remi's PHP 7.0 test RPM repository for Enterprise Linux 6 - x86_64
remi-php70-test-debuginfo   Remi's PHP 7.0 test RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
remi-php71                  Remi's PHP 7.1 RPM repository for Enterprise Linux 6 - x86_64
remi-php71-debuginfo        Remi's PHP 7.1 RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
remi-php71-test             Remi's PHP 7.1 test RPM repository for Enterprise Linux 6 - x86_64
remi-php71-test-debuginfo   Remi's PHP 7.1 test RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
remi-php72                  Remi's PHP 7.2 RPM repository for Enterprise Linux 6 - x86_64
remi-php72-debuginfo        Remi's PHP 7.2 RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
remi-php72-test             Remi's PHP 7.2 test RPM repository for Enterprise Linux 6 - x86_64
remi-php72-test-debuginfo   Remi's PHP 7.2 test RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
remi-safe-debuginfo         Remi's RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
remi-test                   Remi's test RPM repository for Enterprise Linux 6 - x86_64
remi-test-debuginfo         Remi's test RPM repository for Enterprise Linux 6 - x86_64 - debuginfo
repolist: 0
```

remi-safe は有効ですが、remi は無効になっています。

### php56 の導入

次に、php56 を入れます。php56 は remi-safe 配下で管理されているのでそのまま入ります。

```
vagrant@cent69base:~$ sudo yum install php56
```

入れた後はこうなりました：

```
vagrant@cent69base:~$ rpm -qa | grep -E '(php|release)' | sort
centos-release-6-9.el6.12.3.x86_64
epel-release-6-8.noarch
php-5.3.3-49.el6.x86_64
php-cli-5.3.3-49.el6.x86_64
php-common-5.3.3-49.el6.x86_64
php56-2.3-1.el6.remi.x86_64
php56-php-cli-5.6.35-1.el6.remi.x86_64
php56-php-common-5.6.35-1.el6.remi.x86_64
php56-php-pear-1.10.5-5.el6.remi.noarch
php56-php-pecl-jsonc-1.3.10-1.el6.remi.x86_64
php56-php-pecl-zip-1.15.2-1.el6.remi.x86_64
php56-php-process-5.6.35-1.el6.remi.x86_64
php56-php-xml-5.6.35-1.el6.remi.x86_64
php56-runtime-2.3-1.el6.remi.x86_64
remi-release-6.9-2.el6.remi.noarch
```

php-5.3 と php-5.6 が共存しています。

### それでも php-5.3 が動いている

```
vagrant@cent69base:~$ which php
/usr/bin/php
vagrant@cent69base:~$ rpm -qf `which php`
php-cli-5.3.3-49.el6.x86_64
vagrant@cent69base:~$ ls -l /usr/bin/php
-rwxr-xr-x. 1 root root 3273840  3月 22 12:29 2017 /usr/bin/php
vagrant@cent69base:~$ php -v
PHP 5.3.3 (cli) (built: Mar 22 2017 12:27:09)
Copyright (c) 1997-2010 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2010 Zend Technologies
```

でも見えているのは 5.3 の方。モジュール版の方でも試してみます。テキストブラウザとして w3m を使います。

```
vagrant@cent69base:~$ sudo sh -c 'echo "<?php phpinfo();" > /var/www/html/phpinfo.php'
vagrant@cent69base:~$ sudo /sbin/service httpd start
Starting httpd:                                            [  OK  ]
vagrant@cent69base:~$ sudo yum install w3m
vagrant@cent69base:~$ w3m http://localhost/phpinfo.php
PHP Logo

PHP Version 5.3.3
（以下略 ... "Q" で w3m を終了）
```

### php56 の構造

/usr/bin/php はシンボリックリンクでもなく、切り替えて使うことは考慮されていません。一方、php56 は以下のような仕組みとなっています。

```
vagrant@cent69base:~$ rpm -ql php56
(contains no files)
vagrant@cent69base:~$ rpm -ql php56-runtime
```

php56 には何も入っていませんが、php56-runtime には大量に入っていますので内容を見てみます。

```
vagrant@cent69base:~$ rpm -ql php56-runtime | head -6
/etc/opt/remi/php56
/etc/scl/prefixes/php56
/opt/remi
/opt/remi/php56
/opt/remi/php56/enable
/opt/remi/php56/root
vagrant@cent69base:~$ ls -F /etc/opt/remi/php56/
X11/  pear/      php.d/   pki/  skel/       xdg/
opt/  pear.conf  php.ini  pm/   sysconfig/  xinetd.d/
```

/etc/opt/remi/php56/ 配下には、本来 /etc 直下にあるファイル／ディレクトリが見えます。

```
vagrant@cent69base:~$ ls -F /opt/remi/php56/
enable  root/
vagrant@cent69base:~$ ls -F /opt/remi/php56/root/
bin/   dev/  home/  lib64/  mnt/  proc/  sbin/     srv/  tmp/  var/
boot/  etc/  lib/   media/  opt/  root/  selinux/  sys/  usr/
```

/opt/remi/php56/root/ は / 直下の構造と同じであり、chroot 的な意図が見て取れます。

### Software Collections

```
vagrant@cent69base:~$ cat /etc/scl/prefixes/php56
/opt/remi
```

このファイルで remi リポジトリとしての起点を規定しています。"scl" は Software Collections のコマンド名でもあります。

```
vagrant@cent69base:~$ scl -l
php56
vagrant@cent69base:~$ rpm -qf `which scl`
scl-utils-20120927-29.el6_9.x86_64
vagrant@cent69base:~$ rpm -qi scl-utils | grep ^Summary
Summary     : Utilities for alternative packaging
vagrant@cent69base:~$ scl --help
usage: scl <action> [<collection>...] <command>
   or: scl -l|--list [<collection>...]
   or: scl register <path>
   or: scl deregister <collection> [--force]

Options:
    -l, --list            list installed Software Collections or packages
                          that belong to them
    -h, --help            display this help and exit

Actions:
    enable                calls enable script from Software Collection
                          (enables a Software Collection)
    <SCL script name>     calls arbitrary script from a Software Collection

Use '-' as <command> to read the command from standard input.
```

/etc/scl/prefixes/php56 でディレクトリツリーが /opt/remi を起点にすることを示し、scl コマンドが /opt/remi/php56/enable を呼ぶことにより、php56 が有効になるようです。/opt/remi/php56/enable の中身は以下の通りです。

```
vagrant@cent69base:~$ cat /opt/remi/php56/enable
export PATH=/opt/remi/php56/root/usr/bin:/opt/remi/php56/root/usr/sbin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/opt/remi/php56/root/usr/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export MANPATH=/opt/remi/php56/root/usr/share/man:${MANPATH}
```
scl コマンドは、バージョンによりかなりオプションが変わっているようです。scl-utils-20120927-29.el6_9.x86_64 に含まれる scl コマンドでは、以下のようなオプションが使えます。

- php56 という『コレクション』に含まれるパッケージを表示する

```
vagrant@cent69base:~$ scl -l php56
php56-runtime-2.3-1.el6.remi.x86_64
php56-2.3-1.el6.remi.x86_64

```

- php56 コレクションを有効にして php -v を実行する

```
vagrant@cent69base:~$ scl enable php56 -- php -v
PHP 5.6.35 (cli) (built: Mar 29 2018 07:18:29)
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
vagrant@cent69base:~$ php -v
PHP 5.3.3 (cli) (built: Mar 22 2017 12:27:09)
Copyright (c) 1997-2010 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2010 Zend Technologies
```

php56 が優先されるのは scl コマンドのプロセスの実行中だけのようです。コマンドとして bash を指定すれば、サブシェルとして php56 が有効な環境が作られます。

```
vagrant@cent69base:~$ scl enable php56 -- bash
vagrant@cent69base:~$ php -v
PHP 5.6.35 (cli) (built: Mar 29 2018 07:18:29)
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
vagrant@cent69base:~$ exit
exit
vagrant@cent69base:~$ php -v
PHP 5.3.3 (cli) (built: Mar 22 2017 12:27:09)
Copyright (c) 1997-2010 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2010 Zend Technologies
```

