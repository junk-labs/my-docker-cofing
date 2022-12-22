# my-docker-cofing
自分用のdocker-compose.yml達

Ubuntu Core 22 とSnap版 Docker 向けなので普通のLinuxディスとリビュージョンと違った表記になってます。

特殊環境なので備忘録として公開。。。

各ファイルは/root/docker/各コンテナ/に置く前提で書いてます。

~~ バインドマウントなどで場所を変える場合はdocker-compose@.serviceを適宜編集するとよいでしょう ~~

docker-compose@.serviceの使用辞めました。

## ネットワーク設定
docker-compose.yml のnetworkに定義してあるexternal-networkは手動でブリッジを作成して既存ネットワークからコンテナに割り当てたIPアドレスに直接到達可能な構成にしています。

### ブリッジアダプタ作成
Ubuntu Core 22 なのでnetplan用の設定ファイルを作成する

/etc/netplan/99-bridges.yaml

```yaml
# This is the network config written by 'console-conf'
network:
  ethernets:
    ens33:
      dhcp4: no
  bridges:
    br0:
      interfaces:
        - ens33
      dhcp4: no
      addresses:
        - 192.168.100.223/24
      gateway4: 192.168.100.1
      nameservers:
        addresses:
        - 192.168.100.200
        - 192.168.100.201
        search:
        - junk-labs.net
      parameters:
        forward-delay: 0
        stp: no
      optional: true
  version: 2
```

### Dockerをブリッジアダプタに接続
引数の–gateway は作成したブリッジのアドレスを指定する。ルーターのアドレスなどを指定するとブリッジが壊れて締め出される羽目になる。

```
# docker network create \
  --driver=bridge \
  --subnet 192.168.100.0/24 \
  --gateway 192.168.100.223 \
  --opt "com.docker.network.bridge.name"="br0" \
  external-network
```

## 使用イメージ
### pihole
広告排除DNSサーバー Pi-Hole
仮想マシン並みに多数のポートが開くので外部ネットワーク直結がよく似合う。

- 公式 https://pi-hole.net/
- イメージ https://hub.docker.com/r/pihole/pihole

### feed
自分用全文RSS生成サービス fivefilters Full-Text RSS
イメージがPHP5+Apacheベースなのでその点要注意

- 公式 https://www.fivefilters.org/full-text-rss/
- ソース https://bitbucket.org/fivefilters/full-text-rss/src
- ソースその2 https://github.com/fivefilters/ftr-site-config
- 使用イメージ（非公式） https://hub.docker.com/r/heussd/fivefilters-full-text-rss
- 上記のDockerfile https://github.com/heussd/fivefilters-full-text-rss-docker

### php7、php8
Nginxとphp-fpmがセットになったイメージ
Adminer.php等を収容している。上記のfull-text-rssも収容予定
tag指定でPHPのバージョンを変えることができるので7と8をそれぞれ立てている。

- 使用イメージ https://hub.docker.com/r/webdevops/php-nginx

### php7dev、php8dev
上記にxdebug等開発向けの設定が追加されたバージョン

- 使用イメージ https://hub.docker.com/r/webdevops/php-nginx-dev

### dash.old
自宅サーバー向けダッシュボード Dashy
Azureのアクセスパネル的なものを作成することができるOSS
一通り設定した後にさらに好みのアプリが見つかったため15分で運用停止

- 公式 https://demo.dashy.to/
- ソース https://github.com/Lissy93/dashy

```
find . -type f -print | xargs chmod 666
find . -type d -print | xargs chmod 777

🎰
```

### dash
自宅サーバー向けダッシュボード Homarr 🦀
こっちのほうがデザインが好みで設定がシンプルなので採用決定。

- 公式 https://homarr.ajnart.fr/ja
- ソース https://github.com/ajnart/homarr
