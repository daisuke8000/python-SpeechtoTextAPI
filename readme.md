

##  1.Python × GCPで音声の文章化を体験してみよう！

#### 目的

1. Google Cloudが提供するAPIを使えるようにする。
2. PyaudioとCloud Speech-to-text APIを使って録音・議事録への書き起こしができるようにする。

#### ゴール

ローカル上でプログラムを起動して、音声をテキストに出力してみよう！



### 構成図

![](https://storage.googleapis.com/zenn-user-upload/ofn5rx0rmtzhps144z4l01efk13k)

### 使用技術

- Python 3.6.3
- GCP : Cloud Speech-to-TextAPI



### 前提条件

- Pythonがインストールされている(ver3.5~)
- GoogleAccountを取得している
- GCPにログイン出来る



## 2. まだPyhton3のインストールが完了していない方のために



あとから[Google Cloud SDK](https://cloud.google.com/sdk/docs/quickstart?hl=ja)をインストールする際に必要になりますので、
以下のリンクを参考に各OSでインストールして下さい。
[Python環境構築ガイド](https://www.python.jp/install/install.html)
※Windows環境の方は[Chocolatey](https://chocolatey.org/install#installing-chocolatey)を推奨します。
なお、ChocolateyでPythonをインストールする際は以下のコマンドを使用します。



```shell
# Pythonのインストール
$ choco install python
# Version指定時
$ choco install -y python --version=3.6.3
# pipのインストール
$ pip install --upgrade pip
```



## 3.GCP上でプロジェクトを作成

1. [GCP](https://console.cloud.google.com/?hl=ja&pli=1)にログインして、「プロジェクトを作成」を選択します。
![](https://storage.googleapis.com/zenn-user-upload/s1i7bfpkltyjhg6ywj36c4yojz6j)

2. プロジェクト名を設定します。（お好きな名前でどうぞ。ここでは「MySpeechApp」 としました。）
    ※場所に関しては「組織なし」のままにしておきます。
    設定したら、作成を押しましょう。
    ![](https://storage.googleapis.com/zenn-user-upload/rk8d331v5ws8mn3s8z35mr3r6yr8)

3.無事作成されたら作成したプロジェクト名のダッシュボードに画面遷移しているので（してると思います）、左上のナビゲーションメニュー（ハンバーガーメニュー？）から「APIとサービス」→「ダッシュボード」へ移動します。
![](https://storage.googleapis.com/zenn-user-upload/flql8kdfnd7gxerba4i8ttxm033y)



4. APIとサービスのダッシュボードへ移動したら「＋APIとサービスの有効化」を選択します。
   ![](https://storage.googleapis.com/zenn-user-upload/zrxhlu1cww3f899dl9ae6n3mcx0i)

5. 「APIライブラリ」という画面になり、APIを選択出来る画面になりますので、
   検索バーに `Cloud Speech-to-Text API`と入力すれば検索にひっかかります。
   ![](https://storage.googleapis.com/zenn-user-upload/oiddnjrn09b1d39b5wxemnjnxl1k)
   ![](https://storage.googleapis.com/zenn-user-upload/294f8m9hdg5jwq73v110noau7m17)

6. APIを選択すると、詳細画面に移ります。ここで「有効にする」を選択します。
   ほどなくして、「概要」の画面に切り替わります。
   ![](https://storage.googleapis.com/zenn-user-upload/oxfty4yybdjsdjc5tdu6yh0v0f9e)
7. 「有効化のステータス」が`有効`　になっていればAPI自体は使える様になっています。
   しかし、画面上に表示されているように認証情報が必要になりますので、「認証情報を作成」
   を選択します。
   ![](https://storage.googleapis.com/zenn-user-upload/pwekr4rpmmkh0sqsnzs0f0l8jype)
8. 「認証情報」画面に切り替わります。ここで使用するAPIを選択します。
   もちろん、`Cloud Speech-to-Text API`を選択します。
   ![](https://storage.googleapis.com/zenn-user-upload/h42z7gcp76819tcvl6sowx8g818p)
9. 次の「App Engine または Compute EngineでこのAPIを使用する予定はありますか？」という選択に対しては、今回の取り組みではインスタンスを使う予定はないので「いいえ、使用していません」を選択し、その下の「必要な認証情報」を選択します。
   ![](https://storage.googleapis.com/zenn-user-upload/89yrvtz8pivihv2lgcx5p4t7s2ek)
10. サービスアカウントを作成します。
    サービスアカウント名を入力（お好きな名前を入力してください）します。自分はここではspeech-apiという名前にしました。
    ※サービスアカウント名を入力すると、「サービスアカウントID」には自動で入力されます。
    その次はロールを選択します。
    ロールに関しては、今回は「オーナー」を選択します。本来は閲覧者や編集者などの権限に沿って設定して下さい。
    キーのタイプは「JSON」を選択し、「次へ」を押すとサービスアカウントが作成され、キーがDL出来ます。
    なお、このJSONファイルは後の作業で使用します。DL後は作業ディレクトリに移動させておきましょう。
    ![](https://storage.googleapis.com/zenn-user-upload/sjm7fccfjrfk9y1w8q77f06bglmo)
    ![](https://storage.googleapis.com/zenn-user-upload/8m98e7y576li0h2gelnp6qvgocl2)
    ![](https://storage.googleapis.com/zenn-user-upload/tcl7l4hvi755crlwluu71frdorfh)
    ![](https://storage.googleapis.com/zenn-user-upload/e6y7jbm87rpg03qp1n1bz1dq1bwp)
11. 認証情報の画面に切り替わると、サービスアカウントに追加されていることが確認出来ます。
    GCP上で設定する内容はここまでです。
    ![](https://storage.googleapis.com/zenn-user-upload/97ha3fg16kxkljxso97t2hg5itqu)



## 4. Google Cloud SDKのインストール

Macの方は[Google Cloud SDK](https://cloud.google.com/sdk/docs/quickstart?hl=ja)をインストールします。

パッケージのtarファイルをDL出来たら解凍します。
```shell
$ tar -xvf {DLしたtar.gzまでのパス}
```
例）
```shell
$ tar -xvf ./Downloads/google-cloud-sdk-324.0.0-darwin-x86_64.tar.gz
```

解凍されたら`google-cloud-sdk`が作成されているため、以下のコマンドでスクリプトを実行します。
```shell
$ ./google-cloud-sdk/install.sh
```

Windowsでは同インストーラーがDL出来るのでそちらの手順に従って設定してください。

※既にインストールされている方はSDKを初期化してください。初期化の方法は以下をリンクを確認して下さい。
[Cloud SDK の初期化](https://cloud.google.com/sdk/docs/initializing?hl=ja)



設定内容としては、
1.) ログイン認証の確認をされるので「Y」と入力して下さい。

```shell
# ここは「Y」と入力してください
You must log in to continue. Would you like to log in (Y/n)?
```

2.) ブラウザが勝手に開くので、プロジェクトを作成したGoogleAccountで認証します。
3.) ターミナル（PowerShell,cmd）に、2で選択したアカウントで使用しているプロジェクトが表示されます。番号のプロジェクトを選択すると設定が完了するので、先ほど作成したプロジェクトに割り当てられている番号を選択します。

```shell
# この場合は1を入力しています
Pick cloud project to use:
 [1] MySpeechApp-xxxx
 [2] hogehoge-xxxxx
 [3] Create a new project
Please enter numeric choice or text value (must exactly match list
item):  1
```



## 5. 仮想環境作成

Pythonを使って開発や実験を行うときは、用途に応じて専用の実行環境を作成し、切り替えて使用するのが一般的です。
主に、システム全体で使うPython環境に影響を与えずにモジュールの追加・入れ替えをしたり、異なるバージョンの Python を使いわけたり、同じモジュールの、複数のバージョンを使い分けたりすることが出来るメリットがあります。

まずは作業するディレクトリを作成します。

作成後にそのディレクトリ内でターミナル（コマンドプロンプト、PowerShell）を開いて、以下のコマンドを実行することで仮想環境を用意します。

1.) 仮想環境を作成
■Mac

```Python
$ python3 -m venv venv
```
■Windows
```Python
$ python -m venv venv
```
上記を実行すると現在の作業ディレクトリにvenvというディレクトリが作成されていると思います。このディレクトリが作成されているのを確認できたら次のコマンドを実行します。

2.) 仮想環境に切り替える
■Mac
```Python
$ . venv/bin/activate
```
■Windows
```Python
$ .\venv\Scripts\activate
```
このコマンドを実行すると、ターミナル（コマンドプロンプト、Powershell）上で以下の画像のようになっていると思います。
![](https://storage.googleapis.com/zenn-user-upload/mipksbaej53jrqfz655e7a3oueq1)



## 6. Credentialsを環境変数に設定する
環境変数　`GOOGLE_APPLICATION_CREDENTIALS` に先程DLしたキー（JSONファイル）迄のファイルパスをセットします。
これは、Cloud SDKでサービスアカウントを使用するには、コードを実行する環境変数を設定する必要があるためです。
環境変数（GOOGLE_APPLICATION_CREDENTIALS）を設定して、アプリケーションコードに認証情報を指定します。
DL後に作業ディレクトリにファイルを移動させていなければこのタイミングで移動させておきましょう。
[こちら](https://cloud.google.com/docs/authentication/getting-started#windows)を参照してください。

作業ディレクトリでターミナル（コマンドプロンプト、Powershell）を開いて以下のコマンドでクリップボードへコピーします。

- Mac(ターミナル)の場合
```shell
$ find $PWD -maxdepth 1 -name "*.json" | pbcopy
```

- Windows（Powershell）の場合①
```shell
$ (Get-ChildItem .\*.json).FullName | Set-Clipboard
```

- Windows（コマンドプロンプト）の場合②
```shell
$ for %a in (%CD%\\*.json) do (echo %a) | clip
```

クリップボードにコピーされているはずなのでそれを以下の手順で環境変数へ設定する。

- Mac
```shell
# 環境変数に設定
# {}は入力不要です。貼り付けの際はダブルクオーテーションで囲って下さい。
$ export GOOGLE_APPLICATION_CREDENTIALS={ここで貼り付け}
#　参考例
$ export GOOGLE_APPLICATION_CREDENTIALS="Users/develop/pyaudio/hogehoge.json"
# 確認(ファイルパスが返ってくればOK)
$ echo $GOOGLE_APPLICATION_CREDENTIALS
```

- Windows
```shell
# PowerShell を使用する場合:
$ $env:GOOGLE_APPLICATION_CREDENTIALS="{ここで貼り付け}"
# 参考例
$ $env:GOOGLE_APPLICATION_CREDENTIALS="C:\Users\develop\pyaudio\hogehoge.json"
# 確認(ファイルパスが返ってくればOK)
$ $env:GOOGLE_APPLICATION_CREDENTIALS
```

```shell
# コマンド プロンプトを使用する場合:
$ set GOOGLE_APPLICATION_CREDENTIALS={DLしたjsonファイルまでのパス}
# 確認(環境変数に設定されているか確認)
$ set
```



## 7. パッケージをインストール

必要なパッケージをインストールします。
音声を文字起こしする際に必要なパッケージと、前述のGoogleのAPIを使うためにこちらのパッケージが必要になります。

```python
# pipのインストール
$ pip install --upgrade pip
# パッケージ類
$ pip install google-cloud-speech pyaudio
```
※WindowsでVer3.7〜で実行する際は前述の[設定方法](https://zenn.dev/daisukesasaki/articles/fd0cafe486c934#pyaudio%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E3%81%A1%E3%82%87%E3%81%A3%E3%81%A8%E8%A3%9C%E8%B6%B3%EF%BC%88windows%E3%81%A7%E3%81%AE%E3%81%BF%E7%A2%BA%E8%AA%8D%EF%BC%89)
を参考にして、以下のコマンドを実行して下さい。

```python
pip install .\PyAudio-0.2.11-cp37-cp37m-win_amd64.whl
```



## 注) pyaudioについて（Windows環境）

資料についてはPythonはver3.6.x~を想定しますが、どうしてもVer3.7~で行う必要がある際はこちらを参考にして下さい。

■なぜこの作業が必要？
pyaudio上で、「portaudio.h」というライブラリが必要になるがこれは完全に独立した、
C言語のライブラリとなっています。[portaudio.hについてはこちら](http://portaudio.com/docs/v19-doxydocs/)
これがWindowsでのPythonのVer3.7~用にビルドされておらず、システムがコンパイルに失敗しているため、インストール時にエラーが発生する。というのが原因になります。

そのため、PythonのVerに応じたビルド済のWindows用バイナリとなるwhlファイルを別途入手する必要があります。
whlファイルを入手し、後述のコマンドを使用してpip install することで要件を満たす事が出来ます。

■PyAudioの依存関係の不具合の解消
Ver3.7以降だとpyaudioのインストール時にverに沿ったwhlファイルをDLする必要があります。そのため、インストール前に[このリンク](https://www.lfd.uci.edu/~gohlke/pythonlibs/#pyaudio)からwhlファイルをDLして下さい。

■whlファイルの選択の仕方

1. リンク先に画面遷移したらPyAudioを探し、所定のバージョンを選択します。
   ![](https://storage.googleapis.com/zenn-user-upload/feoteo0vhicn7qcbo5xigt3gxtdl)

2. バージョンごとのwhlファイルの選択の仕方としては以下のように読み取って選択します。
   ![](https://storage.googleapis.com/zenn-user-upload/cnemklcirv3fsqtc0n1x1nco8ytb)

3. DL後は現在の作業ディレクトリにwhlファイルを置いて以下のコマンドを実行して下さい。
   （基本的には任意の場所で結構です）

```python
pip install .\PyAudio-0.2.11-cp37-cp37m-win_amd64.whl
```



## 8. サンプルコードでの実装

```python
大友さん
＞＞ここにコード貼って下さい。





```

### hostError/OsError

セキュリティソフトやサウンド・マイクの許可が通っていないとうまく動作しないため、思い当たる方はそこも確認してみてください。



吉田さん
＞＞ここに削除手順載せて下さい。