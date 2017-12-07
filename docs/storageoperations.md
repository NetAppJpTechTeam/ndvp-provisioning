# ストレージの事前準備コマンド

## 全体の環境

![nDVP PoC environment](./images/env.png)

IPアドレスなどは仮のIPを入れてあります。
後続の手順内で任意のIPを設定してください。

\#{XXX} となっている箇所は任意の値を設定する箇所です。

## 前提

NetApp ONTAP/SolidFire の基本的な用語については理解している前提です。

しかし、この手順どおり実施すれば試しに動かすぐらいの環境であればすぐに作成することができます。

ストレージの操作は管理者権限(admin)で実施する想定です。

また、この手順は __「試しに動かす」__ ために必要な設定のため、本番運用時には詳細な設計をすることを推奨します。

プロダクション環境で使用する際には CISCO Validated Design のような Docker Enterprise のリファレンスアーキテクチャを確認ください。

- [FlexPod Datacenter with Docker Enterprise
 Edition for Container Management
 Design and Deployment Guide for FlexPod Datacenter with Docker
 Enterprise Edition for Container Management](http://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/UCS_CVDs/flexpod_docker_deploy_design.pdf)



## ONTAP編

SVMの作成 → ネットワーク設定 → nfs プロトコルの有効化 → サービス起動 → アクセスできるネットワークの設定を実施します。

### デモ用のSVM作成、NFS v3, v4 を有効化

設定パラメータ

|設定パラメータ	|説明	|
|:---	|:---	|
| #{SVMNAME}   	| SVM 名、任意の名前に設定  	|
| #{IPSPACE}	| 任意のipspace 名を設定  	|
| #{AGGR}  	|svmに関連付ける aggregate を設定   	|



```
vserver create -vserver #{SVMNAME} -ipspace #{IPSPACE} -aggregate #{AGGR} -language C.UTF-8 -rootvolume root -rootvolume-security-style unix
nfs create -vserver #{SVMNAME} -access true -v3 enabled -v4.0 enabled -tcp enabled
```

### すべてのネットワークからアクセスを許可する


|設定パラメータ	|説明	|
|:---	|:---	|
| #{SVMNAME}   	| SVM 名、任意の名前に設定  	|

```
export-policy rule create -vserver #{SVMNAME} -policyname default -clientmatch 0.0.0.0/0 -rorule any -rwrule any  -superuser any
```

### データアクセス用のLIF・管理用のLIFの作成

NFS接続用のデータアクセス通信用のインターフェースを作成します。

- \#{SVMNAME}: SVM 名、任意の名前に設定
- \#{DATA-LIF}: データ通信用のLIF 名、任意の名前に設定
- \#{HOME-NODE}: LIFを作成するホームノード
- \#{HOME-PORT}: LIFを作成するホームポート


|設定パラメータ	|説明	|
|:---	|:---	|
| #{SVMNAME}   	| SVM 名、任意の名前に設定  	|
| #{DATA-LIF}   | NFS通信用のネットワークインターフェース |
| #{HOME-NODE} | LIFを作成するホームノード    |
| #{HOME-PORT} | LIFを作成するホームポート |
| #{IP}        | NFS通信をするLIFのIP |
| #{NETMASK}    | ネットマスク    |

```
network interface create -vserver #{SVMNAME} -lif #{DATA-LIF} -role data -data-protocol nfs -home-node #{HOME-NODE} -home-port #{HOME-PORT} -address #{IP} -netmask #{NETMASK} -status-admin up -failover-policy system-defined -firewall-policy data -auto-revert true
```

SVM管理用のインターフェースを作成します。

|設定パラメータ	|説明	|
|:---	|:---	|
| #{SVMNAME}   	| SVM 名、任意の名前に設定  	|
| #{MGMT-LIF}   | SVM管理用のネットワークインターフェース |
| #{HOME-NODE} | LIFを作成するホームノード    |
| #{HOME-PORT} | LIFを作成するホームポート |
| #{SVM-MGMT-IP} | 管理通信をするLIFのIP  |
| #{NETMASK}    | ネットマスク    |

```
network interface create –vserver #{SVMNAME} –lif svm-mgmt –role data –data-protocol none –home-node #{HOME-NODE} -home-port  #{HOME-PORT} –address #{SVM-MGMT-IP} -netmask #{NETMASK} -status-admin up –failover-policy system-defined –firewall-policy mgmt –auto-revert true
```

### 作成した SVM の管理者ユーザの有効化、パスワード設定

SVM管理者を使用する場合、vsadmin のパスワード設定、ロック解除


|設定パラメータ	|説明	|
|:---	|:---	|
| #{USERNAME}   	| SVM上に存在するユーザ名を指定、SVM管理者の場合`vsadmin`を指定  	|
| #{SVMNAME}   	| ユーザ名が存在するSVMを指定  	|

```
security login password -username #{USERNAME} -vserver #{SVMNAME}
security login unlock -vserver #{SVMNAME} -username #{USERNAME}

```

### SVM から操作できるアグリゲートを設定

vsadmin(SVM管理者)で nDVP を使用する場合は SVM から操作できるアグリゲートの設定をする必要があります。

|設定パラメータ	|説明	|
|:---	|:---	|
| #{SVMNAME}   	| 設定対象のSVMを指定  	|
| #{AGGR}   	| svmに関連付ける aggregate を指定   	|


```
vserver modify -vserver #{SVMNAME} -aggr-list #{AGGR}
```

## SolidFire 編

SolidFire では nDVP のユーザアカウントを作成するのみです。

### ユーザアカウントの作成
SolidFireで ndvp で使用するユーザアカウントの作成、create-account.json というファイル名で保存する。

```json
{
   "method": "AddAccount",
   "params": {
     "username": "docker",
     "attributes": ""
   },
   "id": 1
}
```

作成した json を curl コマンドで REST API 経由で実行。

```!commandline
$ curl -k -X POST -H "Content-type: application/json-rpc" --data @create-account.json "https://admin:solidfire@192.168.199.224/json-rpc/7.0"
```

```json
{
  "id": 1,
  "result": {
    "accountID": 1
  }
}
```

### アカウント作成確認

作成したアカウントを確認、 get-account.json として以下の内容で作成する。

```json
{
  "method": "GetAccountByName",
  "params": {
    "username": "docker"
  },
  "id": 1
}
```

上記で作成した

```!commandline
$ curl -k -X POST -H "Content-type: application/json-rpc" --data @get-account.json "https://admin:solidfire@192.168.199.224/json-rpc/7.0"
```

```json
{
  "id": 1,
  "result": {
    "account": {
      "accountID": 1,
      "attributes": {},
      "initiatorSecret": "e!%V(17DkZH5[cen",
      "status": "active",
      "targetSecret": "7!##H8KzA6wQ3lmY",
      "username": "docker",
      "volumes": []
    }
  }
}
```
