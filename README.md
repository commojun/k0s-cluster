# k0s-cluster
k0sのクラスターを作る

詳しい使い方については[githubリポジトリ](https://github.com/k0sproject/k0sctl)を参照

## おおまかな流れ

### 1, k0sctlをインストールする（最新を確認）
```
$ go install github.com/k0sproject/k0sctl@latest
```

もしくは[releases](https://github.com/k0sproject/k0sctl/releases)のページから最新のバイナリをDLしてPATHの通った場所に配置する

### 2, k0sのバージョンを記載

spec.k0s.version に記載するバージョンを[k0sのリリースページ](https://github.com/k0sproject/k0s/releases)で確認して記載する

### 3, ノード構成を記載

k0sctl.ymlにノード構成を記載する

### 4, 適用

```
$ k0sctl apply --config ./k0sctl.yaml
```
