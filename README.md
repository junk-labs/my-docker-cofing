# my-docker-cofing
自分用のdocker-compose.yml達

Ubuntu Core 22 とSnap版 Docker 向けなので普通のLinuxディスとリビュージョンと違った表記になってます。

特殊環境なので備忘録として公開。。。

各ファイルは/root/docker/各コンテナ/に置く前提で書いてます。

バインドマウントなどで場所を変える場合はdocker-compose@.serviceを適宜編集するとよいでしょう

## ネットワーク設定
docker-compose.yml のnetworkに定義してあるexternal-networkは手動でブリッジを作成して既存ネットワークからコンテナに割り当てたIPアドレスに直接到達可能な構成にしています。

### ブリッジアダプタ作成
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