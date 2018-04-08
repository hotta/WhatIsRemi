# Remi のサイトにある [Wizard](https://rpms.remirepo.net/wizard/) で相談した時の内容

###  【質問セクション】Operating system and version selection
- Operating system:  [CentOS 6 (保守期限 2020/03)
- Wanted PHP version:  [5.6.35 (セキュリティ対応のみ 2020/12 まで)
- Type of installation:  [複数バージョンを同時にインストール]

### Wizard answer (回答セクション）
- CentOS 6 では公式リポジトリで PHP 5.3 を提供中
- EPEL をインストールするコマンド：
    yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
    → これは単に yum install epel-release でもいけます。
- Remi リポジトリをインストールするコマンド：
    yum install http://rpms.remirepo.net/enterprise/remi-release-6.rpm
    → これも yum install remi-release でもいけます。
- (yum-config-manager コマンドを使うために）yum-utils をインストール：
    yum install yum-utils
- 複数バージョンの同時インストールには Software Collection を利用
  - php56 コレクションは remi-safe リポジトリで利用できる
- インストールするコマンド
    yum  install php56
- 追加パッケージをインストールするコマンド
    yum  install php56-php-xxx
- テストパッケージをインストールするコマンド
    yum --enablerepo=remi-test install php56-php-xxx
- インストールされたバージョンと利用可能な拡張を表示：
    php56 --version
    php56 --modules
