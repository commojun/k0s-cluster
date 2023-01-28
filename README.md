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

### 4, ノード側の準備

- cgroupを有効にする ([参考](https://zenn.dev/link/comments/18ff5c881781be))
- microsdで動いているノードはswapを無効にしておく ([参考](https://letraspberry.hatenablog.com/entry/2021/02/12/233725#2-swap%E3%81%AE%E7%84%A1%E5%8A%B9%E5%8C%96))

### 4, 適用

```
$ k0sctl apply --config ./k0sctl.yaml
```
