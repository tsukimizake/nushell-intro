# nushellとは

https://www.nushell.sh/ja/

mac, win, linuxで動作する正気なシェル

![Screenshot 2023-11-23 at 14 20 44](https://github.com/tsukimizake/nushell-intro/assets/2472792/e4b41a94-0dc3-479a-b2f1-8e79eb144dbf)

インストールしてlsコマンドなどを叩いてみると出力がリッチでなるほどそういう感じねとなるが、

実はそれどころではなく実用レベルの型付きシェルスクリプトという全人類の夢


# nushellのいいところ
## なんと型がある！！！！！！！（ただし動的型）
bashやzshなどの通常のシェルは基本的に全てのデータを文字列として流し、sedやawkなどで整形して扱う (TODO 例)

nushellのパイプを流れるデータには型がつく！
https://www.nushell.sh/book/types_of_data.html

string, boolean, intなどの基本型の他にlist, record, tableなどのデータもpipeで流せる
また、dateやfile sizeなどを不等号で比較してフィルタなどという芸当もできる

スクリプトを書いている途中に型を見たければdescribeコマンドにpipeすればよい

関数にシグネチャも書けるが静的チェックは現状されず、どうもドキュメントとして書けるだけらしい

## manと違ってhelpが普通に読める
zshのman ls

![Screenshot 2023-11-23 at 14 39 04](https://github.com/tsukimizake/nushell-intro/assets/2472792/6fc730eb-992a-4a87-af3d-20ac61caad25)

謎の一文字オプションの解説が延々と続いていて目が滑る

nushellの help ls

![Screenshot 2023-11-23 at 14 38 30](https://github.com/tsukimizake/nushell-intro/assets/2472792/82b268ef-8c91-46f3-bc4d-28163df7f00f)

読める！！！！

また、helpで解決しなかった場合でもdiscordが活発なので質問を投げると10分くらいで大抵解決する

## スクリプト自体もbashと違ってなんとなくで読める

web上のnushell一枚スクリプトをインストールする君（自作）

```nu
module plug {
  def plugins_file [] {
    $nu.default-config-dir | path join 'plugins.nu'
  }

  def gc [] {
    open (plugins_file)
      | lines
      | uniq
      | filter {|l| $l != ""}
      | to text
      | save (plugins_file) -f
  }

  export def install [file_url : string] {
    let plugin_file = (curl $file_url)

    let plugin_name = $file_url 
            | path split 
            | last

    let plugin_path = $nu.default-config-dir 
            | path join 'plugins' 
            | path join $plugin_name

    $plugin_file | save $plugin_path -f

    $'source "($plugin_path)"' 
      | save (plugins_file) --append
    
    gc
  }
}
```

読める！！！！！！

## 全体的に罠が少ない
例えば、上の `def gc` をbashで同様に書くと、入力ファイルと出力ファイルが同じなのでファイルの内容が消えたりする（はず）
他に set -euo pipefailというおまじないをファイル先頭で唱えておかないとエラー時に止まらず続きの行も実行ししたりする
bashには無数の罠があるが、nushellでこういった罠はほぼなく普通に書ける

# 慣れるまでちょっと気になるところ
## lexerが気難しい
例えば `def plugins_file [] {...}` と書くべきところを `def plugins_file[] {...}` と書いてしまうとパースエラーになる

## 正規表現がrustのregexモジュールそのままで若干書きづらい
`help parse` を参照

## デフォルトで入っている補完などが弱い
https://github.com/nushell/nu_scripts に大体揃っているので↑のインストーラー君などで入れると普通に使えるようになる
