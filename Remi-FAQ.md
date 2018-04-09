Remi''s RPM repository - English FAQ
  
# 僕の仕事に関係してる？

このリポジトリやブログは僕の現在の雇用主とは全くの無関係。ただし僕の現在の仕事は PHP のパッケージングが主だから、このプロジェクトに対してより多くの時間をつぎ込んでいるのは間違いない。

ここに書かれている意見等は僕個人のものであって、いかなるコミュニティー、プロジェクト、団体（Fedora, PHP, Red Hat, CentOS, ...)とも無関係です。

# このリポジトリはどこを目指しているの？

Fedora と Enterprise Linux (RHEL, CentOS, Oracle, Scientific Linux, ...) を使っているユーザーに対して、最新バージョンの PHP スタックと、その周辺機能を提供すること。主に以下のものを提供している：

- 僕が Fedora でも保守しているもの
- Fedora の開発版で利用可能なパッケージのバックポート
- Fedora のポリシーに一部合致しないパッケージ群
- Fedora リポジトリで採択される前の作業中のパッケージ群
- 原型のまま（ほとんど手を入れていない）版

これは Enteprise Linux の [Backporting Security Fixes](https://access.redhat.com/security/updates/backporting/) のポリシーとはかけ離れている。

# どこに聞けばいい？ Remi にコンタクトを取る方法は？

ブログのコメント欄は議論には向いてない。質問、サポート、議論には [フォーラム](https://forum.remirepo.net/) を使ってほしい。

IRC の #remirepo チャンネル (freenode) に来てもらってもいい。[WebChat](http://webchat.freenode.net/?channels=remirepo) でのアクセスもできるよ。

# Remi を応援したいんだけど？

やり方はいろいろあるよ。たとえば、

- [Merci / Thanks](http://webchat.freenode.net/?channels=remirepo) でちょっとしたコメントを書く
- リポジトリを使ってみて、そのユースケースについてフォーラムで報告する
- 自分のブログでこのリポジトリを紹介する
- https://blog.remirepo.net/pages/English-FAQ の右上にある "Donate" ボタンから寄付する。そのお金は、次回のビルドや僕の Web ホスティング / Web アクセスなどに使わせてもらいます。

# リポジトリをどうやって使うの？

[Repository Configuration](https://blog.remirepo.net/pages/Config-en) のページに詳しく書いてあるよ。[Configuration Wizard](https://rpms.remirepo.net/wizard/) も試してみるといいね。

# なぜこのリポジトリをデフォルトで有効(enabled)にしないの？

"remi-sage" リポジトリは実際に安全だから、デフォルトで有効になっているよ。



# Why the repository isn't enabled on default ?
The "remi-safe" repository is enabled by default, as it is really safe.

Available packages in "remi" and others "remi-*" repositories override those in official repository. So their installation must be a pondered administrator choice. But it's really easy to permanently enable it.

For PHP, you have to choose the repository which provides the version you wamt  (remi-php56, remi-php70,  remi-php71...).

# The "remi-safe" repository ?
On Enterprise Linux 6 and 7, the new "remi-safe" repository is now enabled by default.

This repository doesn't override/replace any package of the distribution. It provides a  set of packages, mostly PHP extensions, not yet in EPEL. It also provides recent version of some libraries which can be installed beside system version (e.g. gd-last).

New feature: it provides all depedencies for packages in the other repositories (so "remi" is no more required)

# How to keep informed of repository news ?
First, reading this blog, each new major version is announced by an entry, minor one by a comment Afterward, each repository have a RSS feed, ex for e6.x86_64.

You can follow RemiRepository on twitter.

# Can I synchronize a local mirror of remi repository ?
Yes : you can now use rsync, for this, check the mirrors list, and search one close to your location with the rsync flag.

Access to the main server is restricted to official mirrors.

Recursive wget/ftp/curl from the primary mirror is a very bad discouraged practice.

# Can I update Fedora with remi repository enabled ?
Yes, using the dnf system-upgrade tool, with some minor precautions:

Upgrade to Fedora 25
disable the remi-php70 repository, as PHP 7.0 is the default version in remi for F25.
Upgrade to Fedora 26 or 27
import the latest key before the upgrade (i.e. /etc/pki/rpm-gpg/RPM-GPG-KEY-remi2017)
disable the remi-php71 repository, as PHP 7.1 is the default version in remi since F26.
remove the php54-* and php55-* packages as these collections are no more available
from Fedora 25, if installed the php56-runtime package must be downgraded to 2.1
Upgrade to Fedora 28
import the latest key before the upgrade (i.e. /etc/pki/rpm-gpg/RPM-GPG-KEY-remi2018)
disable the remi-php72 repository, as PHP 7.2 is the default version in remi since F28.
# Can MySQL be updated without PHP ?
Yes : compat-mysql51 (fedora 11-14) mysqlclient15 (fedora 8-10 and EL-5) and mysqlclient14 (EL-4) packages provide the connection library for the other packages in the official repository. Yum will handle of this.

# Can PHP be updated without MySQL ?
Yes : compat-mysql55 package provides the connection library for php-mysql. It must be installed "before" the PHP update.

The use of php-mysqlnd allow to use mysql extension without dependency on MySQL library.

Starting with PHP 5.5, there is no more dependency on MySQL.

# Replacement of the base packages ?
Some third party repositories have chosen to not replace the default base package, but to provide packages with different names (see The SafeRepo Initiative).

when I open my repository, in 2005, this idea didn't exist yet (recent versions were provided by Red Hat in a dedicated channel: RHWAS)
I think different names only make sens if both can be installed simultaneously ; which is the case for the libraries (*last packages) and SCL. Conflicts, are evil.
replacement stay an admin choice as the repository is not enabled by default
includepkgs and exclude yum directives can be used to finely configure what will be pulled from my repository.
having various packages providing the same dependencies (ex php-zip provided by php-common, php53-common, php-pecl-zip...) is just a nightmare for yum, raising various dependency issues (an example)
remi repository doesn't only provide PHP and a few set of extensions but a (close to) full stack (more than 500 packages), most of them working with all available versions
some packages are created/developed first in remi, before being imported in Fedora / EPEL.
# Difference between php-* and php##-php-* packages ?
php-* packages are standard ones:

they override the default system version (still an admin choice, as you have to enable the repository to get them)
only one version can be installed
one repository per version (remi-php56, remi-php70 and remi-php71)
packages provide both the NTS and ZTS (Thread Safe) versions
php56-php-*, php70-php-* and php71-php-* packages are Softwares Collections:

they can be installed beside the default system version (in /opt/remi)
various versions can be installed simultaneously
all the versions are in the remi-safe repository
packages only provide the NTS version
Notice: some other repositories have choose to use different names (php56u, php70w) but this create various conflicts. I think a different name only make sense if it allow parallel installation with standard version.

# Can the repository be used only for the Softwares Collections ?
Yes : simply add this line in the remi.repo configuration file

includepkgs=php70*
The php56,  php70 and php71 SCL are now  available int the remi-safe repository, so no configuration change required.

# Missing dependencies
All dependencies are available in standard repository, and for  Fedora in RPMFusion and for Enterprise Linux in EPEL.

remi-test, remi-php55 and remi-php56 repositories also require the reml repository.

Common dependencies for packages in remi-php** repositories are in remi-safe, enabled by default (as this is new, please report any missing package).

RPM for Enterprise Linux are build using official RHEL 5.11, RHEL 6.9 and RHEL-7.4 repositories. Some packages can be missing in older versions (ex openssl 1.0.2).

Note : for RHEL-6 and RHEL-7 the Optional channel is mandatory  (rhel-x86_64-server-optional-6, rhel-7-server-optional-rpms).

Some "noarch" packages in "remi" repository (e.g. phpMyAdmin) now require php(language) >= 5.5 or 5.6. As PHP 5.4 and 5.5 have reached their "End of life", if you want to use these applications, you have to update, using the "remi-php55" repository (or greater).

# No packages found ?
Check if the repository is correctly enabled.

Else, packages from base repository can be protected by the yum priorities plugin. In this case, you may have to disable the plugin (--noplugins or --disableplugin=priorities) or change the repository configuration to set a higher priority.

# Conflict between i386 and x86_64 packages ?
This is not directly an issue with remi repository, but rather a old known bug of the old fedora versions and EL-5 installer. Bug which is fixed in more recent versions. This bug cause both the 32 bits and 64 bits libraries being installed. For me, the simplest is the full clean of all 32 bits libraries after a 64 bits system installation:

yum remove glibc.i686
# Can I  get an old version of a package ?
Notice : the main goal of remi repository is to provide the latest versions to have benefit of improvements and fixes, especially security ones.

Yes, this is possible, just open a request on the forum, with all the needed information about the distribution, version and architecture. As this require a search in my off-line backup, you must be patient.

# debuginfo availability ?
They are available in the repository for packages built after January 2013.

# Firefox, Thunderbird, etc for EL ?
Graphical applications, are only available for fedora, which I consider as the best choice for a workstation. For me, Enterprise Linux is mainly for server system.

Despite this, this applications are now available for EL-6 (le latest version).

Because lack of time, I have suspended their maintenance. I prefer to concentrate my work on the PHP stack.

Publié le jeudi 25 août 2011 par Remi

PayPal - The safer, easier way to pay online!
Informations
English : Site introduction
English : User testimonials
English : Repository Configuration
English : FAQ
English : pending reviews
Présentation du site
Configuration du dépôt
FAQ en Français
Merci / Thanks
PHP extensions RPM status (from PECL and other sources)
Statistiques de téléchargement Download counter
RPM : 265978557
Date : vendredi 6 avril 2018 - 06:04
Recherche


 

Tags
Beta dépôt dotclear firefox FusionInventory GLPI GUI memcache MySQL OCSNG PHP PHPUnit planet-php planetlibre qelectrotech RHEL RPM SCL thunderbird WebApp
Tous les tags

Licence information
This site content is licensed under a Creative Commons Attribution-ShareAlike 4.0 International License.
Le contenu de ce site est mis à disposition selon les termes de la Licence Creative Commons Attribution - Partage dans les Mêmes Conditions 4.0 International.
 Licence Creative Commons
