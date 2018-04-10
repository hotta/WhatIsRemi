# How to build php56

```bash
$ git clone git@bitbucket.org:michi_hotta/ansible-centos6.git
$ cd ansible-centos6
$ ansible-playbook -i hosts site.yml
$ cd
$ rpmbuild --rebuild ~/rpmbuild/SRPMS/php56-php-5.6.35-1.remi.src.rpm
```

# remi repository に含まれるもの

## php-\* パッケージ

- 標準パッケージ
- デフォルトシステムを上書きする
- 同時には１つしかインストールできない
- バージョンごとにリポジトリが別 (remi-php56, remi-php70 and remi-php71)
- NTS と ZTS の両方のパッケージを提供する

## php56-php-\*, php70-php-\*, php71-php-\* パッケージ

- Softwares Collections パッケージ
- （/opt/remi 配下に入れることで）デフォルトシステムと共存可能
- 同時に複数バージョンをインストール可能
- すべてのバージョンが remo-safe リポジトリに入っている
- 提供するのは NTS のみ
