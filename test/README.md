# ストレージの実験

pi41のみ、SSDで起動しているので、データベースなどのPodをpi41のみで起動するようにし、保存ファイルをすべてSSDに集めるみたいなことをしたい。

## localとnodeAffinityを使う

PersistentVolumeのlocalという要素で、ホストマシンのディレクトリをk8sで使えるストレージ化できるようだ。localを使う条件として、nodeAffinityという要素で、どのノードでストレージを使えるのかという制約を決める必要がある。今回でいえばここにpi41と指定すれば、このPersistentVolumeを使うPodはすべてpi41にデプロイされるという感じになるっぽい。

参考とした記事: https://qiita.com/sotoiwa/items/09d2f43a35025e7be782#local

## nginxをデプロイしてみた

ストレージ内に適当なindex.htmlを作成し、マウントしてnginxをデプロイしてみたところうまいことマウントできていた。さらにコンテナ内に入り込んでファイルを書き換えてみたところ、書き込みは成功し、ホスト側のファイルシステムにも無事反映されていた。ということでdocker-composeのvolumeのように使えることがわかった。

## nfsを利用したストレージ構築

こちらの記事を参考にする: https://blog.denet.co.jp/building-an-nfs-server-and-using-it-as-storage-from-kubernetes/

nfsサーバーをpi41にインストール
```
$ sudo apt install nfs-kernel-server
```

今回は `/home/commojun/nfs` を公開ディレクトリとする。ディレクトリの公開設定

```
$ sudo sh -c 'echo "/home/commojun/nfs 192.168.10.0/24(rw,sync,no_root_squash)" >> /etc/exports'
```

すべてのノードでnfsを利用できるようにツールをインストールする
```
$ sudo apt install nfs-common
```
-> と思ったが、これはもう入っていた。k0sctlがインストールしていたのかもしれない。

pi31でnfsがマウントできるか試してみたところ、うまくいった。ファイルの読み書き、pi41上での反映確認までできた。
```
pi31 $ sudo mount pi41:/home/commojun/nfs ./test
```

次、PV,PVC,PODを用意してコンテナにマウントできるかを確認する。

`nfs-pv.yml` `nfs-pvc.yml` `nfs-pod.yml` をそれぞれ作成した。nginxコンテナに `/home/commojun/nfs/test` をマウントする実験をする。また、`nfs-pod.yml` にて、nfsの実体を持ったpi41以外のノードにnginxがデプロイされるよう、NodeAffinityを設定した。

事前に、マウントするnfsディレクトリ内に適当なindex.htmlを作成した。
```
pi41:/home/commojun/nfs/test $ echo "hoge" > index.html
```

続いて各種k8s設定ファイルをデプロイしたところ問題なくPODが起動した。どのノードでPODが起動したかを確認
```
$ kubectl get pods -o wide
NAME          READY   STATUS    RESTARTS      AGE     IP            NODE   NOMINATED NODE   READINESS GATES
mypod-nfs     1/1     Running   0             7m17s   10.244.1.12   pi32   <none>           <none>
```

ブラウザでアクセスしたところ、あらかじめnfs内に作成したindex.htmlが無事表示された。

コンテナに入り込んでindex.htmlを編集したところ、編集も反映されることを確認した。
```
$ kubectl exec -it mypod-nfs -- bash
# cd /usr/share/nginx/html
# echo "fuga" >> index.html
```

pi41側で実体のファイルも反映を確認
```
pi41:/home/commojun/nfs/test $ cat index.html
hoge
fuga
```
