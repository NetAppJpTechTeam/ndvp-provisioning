# Docker ホストの事前準備

詳細は 

## 必要パッケージ一覧

- apt-transport-https
- ca-certificates
- curl
- software-properties-common
- open-iscsi
- lsscsi
- sg3-utils
- multipath-tools
- scsitools
- nfs-common

## 必要なサービス（有効化、起動済みとするもの）
  - open-iscsi
  - multipath-tools
  - rpcbind


## あると便利なパッケージ

- jq

## OS設定

以下の内容を `/etc/multipath.conf` へ保存

```
defaults {
  user_friendly_names yes
  find_multipaths yes
}
```

