# [IUS Community Project](https://ius.io/)

# プロジェクトの目的

- RHEL/CentOS 向けに、高品質の RPM パッケージを作成する
- Upstream がリリースされたら速やかにパッケージを追随する
- 元の RPM パッケージを自動的に置き換えない

# 根本原理

- 概要

Enterprise Linux に含まれるパッケージは、それぞれのアップストリームと比較して著しく古くなっている場合があります。また、Red Hat の[バックポート](https://access.redhat.com/security/updates/backporting)セキュリティフィックスに新しい機能が追加されることは滅多にありません。このアプローチにより OS 全体が非常に安定した状態を保てますが、アップストリームにある新しい機能を使いたいユーザーにとってはフラストレーションが溜まることになります。
  
IUS は *I*nline with *U*pstream *S*table の略です。私達のパッケージ群はオリジナルのパッケージ群とは根本的に異なります。私達のパッケージはそれぞれのソフトウェアの最新版を追跡します。このアプローチの場合、本質的なリスクが発生します。



```
yum install -y https://centos7.iuscommunity.org/ius-release.rpm
```
