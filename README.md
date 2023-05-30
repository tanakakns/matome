# matome

以下で clone する。

```bash
$ git clone https://github.com/tanakakns/matome.git --recursive
```

`--recursive` つけ忘れたら、 clone 後に以下を実行する。

```bash
$ git submodule update --init
```

その他操作メモ。

```bash
# ローカル実行
$ hugo server -D

# Github Pages 公開
$ rm -fR public # 初回のみ
$ git submodule add -f https://tanakakns@github.com/tanakakns/tanakakns.github.io.git public # 初回のみ
$ hugo
$ cd public
$ git add .
$ git commit -m update
$ git push origin master
```