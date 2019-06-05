[こちら](https://discord.gg/Rngz5k)からDiscordグループに参加してください。

# セットアップ
1. parityのインストール
2. アカウントの作成
3. dapps toolsのインストール
4. mcdパッケージのインストール
5. .sethrcファイルの設定
6. Kovan ETHを入手
7. COL1 tokensを入手

1) parityのインストール
```
// Rustのインストール
$ curl https://sh.rustup.rs -sSf | sh
// parityクライアントのインストール
$ bash <(curl https://get.parity.io -L) -r stable
```
_やり方のわかる方はgethでも大丈夫ですが、テストはしていないので、parityの方が望ましいです。_

2) アカウントの作成
```
$ parity --chain kovan account new
Please note that password is NOT RECOVERABLE.
Type password: passpasspass
Repeat password: passpasspass
// パスワードを入力してください。
// このパスワードは後で使うので、忘れないようにしてください。
0x5e232581fc1cb8cafacfcd951de05cf906e73f93
```

以下のコマンドでアカウントをリストできます。
```
$ parity --chain kovan account list
0x5e232581fc1cb8cafacfcd951de05cf906e73f93
```

3) dapps toolsのインストール

以下のコマンドですぐにインストールできます(GNU/LinuxもしくはmacOS)。
```
$ curl https://dapp.tools/install | sh
```

4) mcdパッケージのインストール
```
$ dapp pkg install mcd
```

v0.2.6 を使っています。
```
$ mcd --version
mcd 0.2.6-rc.1
```

5) .sethrcファイルの設定

ルートディレクトリにある.sethrcファイルの編集
```
export ETH_FROM=0x5e232581fc1cb8cafacfcd951de05cf906e73f93
# 作成したアドレスをここに指定
export MCD_CHAIN=kovan
export SETH_CHAIN=kovan
```

6) Kovan ETHを入手

事前にETHを入手されていない方は、こちらの[faucet](https://faucet.kovan.network/)、もしくは[Gitterチャンネル](https://gitter.im/kovan-testnet/faucet)から入手できます。なかなか入手できないかもしれないので、その場合はスタッフから配布します。

7) COL1 tokensを入手

COL1のアドレスを環境変数に設定
```
$ export COL1A=0xc644e83399f3c0b4011d3dd3c61bc8b1617253e5
```

Faucetのアドレスを環境変数に設定
```
$ export FAUCET=0xa402e771a4662dcbe661e839a6e8c294d2ce44cf
```

gulp(address)関数を呼んで、COL1トークンを入手
```
$ seth send $FAUCET 'gulp(address)' $COL1A
seth-send: warning: `ETH_GAS' not set; using default gas amount
Ethereum account passphrase (not echoed): seth-send: Published transaction with 36 bytes of calldata.
seth-send: 0xf0ac981342ac8ac9c59cd25107667ecc7fd2dadc4089e59aa0b09055d5d08d09
seth-send: Waiting for transaction receipt...
seth-send: Transaction included in block 11268445.
```

gemのbalanceサブコマンドで残高を確認できます。[KovanのEtherscan](https://kovan.etherscan.io)でも上のアドレスから確認できます。
```
$ mcd --ilk=COL1-A gem balance ext
50.000000000000000000
```

# CDPのライフサイクルを１周する
では、これからCOL1トークンを担保として、CDPのライフサイクルを１周してみます。大きく分けると、以下のようなステップで進めます。
1. CDPを開く
2. COL1トークンをデポジットする
3. DAIを引き出す
4. DAIを支払い返す
5. 担保を除く
6. CDPを閉じる

コマンドの性質上、一つ一つのステップに分けづらいので、まずは最初の３つのステップをカバーします。

1. CDPを開く
2. COL1トークンをデポジットする
3. DAIを引き出す

まず、以下のコマンドを走らせてください
```
$ mcd --ilk=COL1-A gem join 30
```

以下のようなアウトプットが表示されます
```
vat 30.000000000000000000 Unlocked collateral (COL1)
ink 0.000000000000000000 Locked collateral (COL1)
ext 20.000000000000000000 External account balance (COL1)
```

COL1トークンが担保としてvatという内部レジストリに追加されました。上のアウトプットでvat内の担保の残高を見ることができます。

次にCOL1トークンをCDPにロックして、DAIを引き出します。

以下のコマンドを走らせてください
```
$ mcd --ilk=COL1-A frob 30 1
```

以下のようなアウトプットが表示されます。
```
ilk  COL1-A                                     Collateral type
urn  0x5e232581Fc1cB8caFAcfCd951De05cF906e73F93 Urn handler
ink  30.000000000000000000                      Locked collateral (COL1)
art  1.000000000000000000                       Issued debt (Dai)
```

inkにCOL1トークンの残高が移り、CDPにCOL1トークンがロックされました。また、DAIが発行されたこともわかります(art)。

DAIはまだvatの中にあるので、それを引き出してextに入れる必要があります。

以下のコマンドを走らせてください
```
$ mcd dai exit 1
```

これでDAIがvatからextに移ります。
```
vat 0.000479620379870463446010918000000000000000000 Vat Dai balance
ext 1.000000000000000000 ERC20 Dai balance
```

ここまでで、CDPを開き、COL1トークンを担保としてCDPにロックし、発行したDAIを引き出しました。

では今度は逆にDAIを払い戻し、担保をCDPから取り出して、CDPを閉じてみましょう。

4. DAIを支払い返す
5. 担保を除く
6. CDPを閉じる

まず以下のコマンドを走らせてください
```
$ mcd dai join 1
```

これでDAIがextから内部レジストリであるvatに移動しました。
```
$ mcd dai balance
vat 1.000479620379870463446010918000000000000000000 Vat Dai balance
ext 0.000000000000000000 ERC20 Dai balance
```

次にDAIをバーンし、COL1トークンをinkからアンロックします
```
$ mcd --ilk=COL1-A frob -- -30 -1
```

アウトプットを見てみましょう
```
ilk  COL1-A                                     Collateral type
urn  0x5e232581Fc1cB8caFAcfCd951De05cF906e73F93 Urn handler
ink  0.000000000000000000                       Locked collateral (COL1)
art  0.000000000000000000                       Issued debt (Dai)
```
inkにあったロックされていた担保とartにあったDAIがなくなっているのがわかります。

最後にvatからCOL1トークンを取り除き、extに戻します
```
$ mcd --ilk=COL1-A gem exit 30
```

以下のようなアウトプットが出て、extにCOL1が戻っていることがわかります。
```
vat 0.000000000000000000 Unlocked collateral (COL1)
ink 0.000000000000000000 Locked collateral (COL1)
ext 50.000000000000000000 External account balance (COL1)
```
