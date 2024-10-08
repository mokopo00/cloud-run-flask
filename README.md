# Cloud RunでFlaskを使ったWebアプリ構築

## 概要
このプロジェクトでは、Flask を使用した Web アプリケーションを Cloud Run で実行する方法を説明します。<br> 
今回は、「Hello World!」を表示するシンプルなアプリについて解説します。

## 準備
### 1. ライブラリのインストール

必要なライブラリをインストールするために、requirements.txtに記述します。
```bash
Flask
gunicorn
```
- Flaskは、Pythonの軽量なWebアプリケーションフレームワークです。シンプルなAPI設計と拡張性の高さから、多くのWebアプリケーション開発に利用されています。
- gunicornは、Python用のWSGI HTTPサーバーです。Flaskアプリケーションを本番環境で効率的に動作させるために使用します.
- WSGI(Web Server Gateway Interface 読み方：ウィズギー): PythonでWebアプリを開発するための標準インターフェースです。

### 2. Flask アプリの作成

まずは、WebアプリケーションフレームワークFlaskを使ったapp.pyを作成します。<br>
これにより、基本的なルーティングとテンプレートレンダリングが可能になります。
```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def hello_world():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```
  
次に、templates/index.htmlを作成します。<br>
このHTMLファイルは、FlaskアプリケーションがルートURLにアクセスされた際に表示される内容を定義します。
```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello World</title>
</head>
<body>
    <h1>Hello, World!</h1>
    <p>This is a simple Flask app serving an HTML page.</p>
</body>
</html>
```
- @app.route('/')でルートURLを指定し、hello_world()関数が呼ばれるとHTMLページが返されます。
- render_templateは、テンプレートファイル（index.html）をレンダリングして返すFlaskの関数です。

### 3. Dockerfile の作成

Dockerとは、アプリケーションをコンテナ化するためのプラットフォームです。コンテナを使用することで、環境に依存せず一貫した動作を保証できます。<br>
Dockerfileには、アプリケーションの実行環境を定義するための命令を記述します。
```bash
# Use the official Python image from the Docker Hub
FROM python:3.12-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 8080 available to the world outside this container
EXPOSE 8080

# Run main.py when the container launches
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "app:app"]
```
- FROM python:3.12-slimは軽量なPythonベースのDockerイメージを指定しています。
- COPYコマンドでローカルのファイルをコンテナ内にコピーし、pip installで必要な依存ライブラリをインストールします。
- EXPOSE 8080で外部アクセス用のポート8080を開放し、CMDでコンテナ起動時にgunicornを使ってアプリを実行します。

※ 注意: CMDで指定するモジュール名はapp.pyに対応するため、app:appとしています。main:appの場合は、main.pyにアプリケーションが存在する場合となります。

## ローカル環境でのアプリの動作確認
アプリを実行するために、イメージ名```xxxxx```を設定します。
### 1. Dockerイメージのビルド

Dockerイメージとは、アプリケーションとその依存関係を含む実行可能なパッケージです。<br>
イメージをビルドすることで、コンテナを生成してアプリケーションを実行できるようになります。
```bash
docker build -t xxxxx .
```

### 2. Dockerイメージの実行

Webアプリケーションをローカル環境で実行します。
以下のコマンドを実行し、ウェブでプレビューのボタンを押すとWebブラウザで動作確認ができます。
```bash
docker run -p 8080:8080 xxxxx
```
- -p 8080:8080は、ローカルのポート8080とコンテナ内のポート8080をバインドして、Webブラウザからアクセスできるようにします。

## Webアプリのデプロイ
    実際にWebアプリケーションを公開します。
### 1. Dockerイメージのビルド

Cloud Runにデプロイするために、Google Container Registry (GCR) 用のタグを付けてイメージをビルドします。
```bash
docker build -t gcr.io/プロジェクト名/xxxxx:1.0 .
```
- 今回は、イメージをバージョン 1.0 としてビルドし、その後のデプロイやテストの際に特定のバージョンのイメージを利用できるようにする。

### 2. Dockerイメージのプッシュ

ビルドしたイメージをGoogle Container Registryにプッシュします。<br>これにより、Cloud Runからイメージを参照できるようになります。
```bash
docker push gcr.io/プロジェクト名/xxxxx:1.0
```
### 3. Webアプリのデプロイ

プッシュしたDockerイメージをCloud Runにデプロイします。<br>
以下のコマンドは、リージョンをus-central1に指定してデプロイする例です。<br>
Webアプリのサービス名をyyyyyに設定する場合は以下のように実行します。
```bash
gcloud run deploy yyyyy --image gcr.io/プロジェクト名/xxxxx:1.0 --platform managed --region us-central1
```
- --platform managedオプションはCloud Runのマネージド環境でデプロイするために指定します。

デプロイが成功すると、アプリケーションのURLが表示されます。

# （補足資料）
## 特定のイメージをpullして、別のイメージとしてpushする方法
### 1. イメージのpull
まず、Container RegistryにあるDockerイメージをローカルにpullします。<br>
```bash
docker pull gcr.io/[PROJECT_ID]/[IMAGE_NAME]:[TAG]
```

### 2. ソースコードを取り出す
Dockerイメージを直接エディタで編集することはできませんが、Dockerイメージからコードを抽出することで編集できます。<br>
まずは。ローカルでイメージを実行します。
```bash
docker run -d --name temp-container gcr.io/[PROJECT_ID]/[IMAGE_NAME]:[TAG]
docker cp temp-container:/path/to/code ./local-directory
```
- temp-containerは、Dockerコンテナを一時的に作成・実行する際に、任意で付けたコンテナの名前
- temp-container:/path/to/codeは、コンテナ内の目的のディレクトリのパス
- temp-container:/path/to/codeは、コンテナ内の目的のディレクトリのパス

※ コンテナ内のどのディレクトリにコードがあるか確認したい場合
→ 以下の手順でコンテナ内でシェルを起動して、ディレクトリ構造を確認する。

#### 2-1. コンテナにアクセス
```bash
docker exec -it temp-container /bin/bash
```
#### 2-2. ルートディレクトリから探索
```bash
ls /
```

例えば、コンテナ内でアプリケーションコードが /app/src に保存されており、
my-codeというディレクトリにコピーしたい場合は、以下のように指定します：
```bash
docker cp temp-container:/app/src /home/user name/my-code
```
これで、別のイメージとしてソースファイルを取り出すことができました。<br>
エディタ画面で確認すると、指定したディレクトリにソースファイルがコピーされています。

### 3. 編集後のイメージをエディタで確認後、イメージをビルドしてプッシュ
```bash
cd my-code
docker build -t gcr.io/[PROJECT_ID]/[NEW_IMAGE_NAME]:[NEW_TAG] .
docker push gcr.io/[PROJECT_ID]/[NEW_IMAGE_NAME]:[NEW_TAG]
```

## まとめ
この手順に従って、Flaskを使用したシンプルなWebアプリケーションをCloud Run上で動作させることができました。Cloud Runを利用することで、スケーラブルで管理の容易なコンテナベースのサービスを迅速にデプロイできます。今後は、アプリケーションに機能を追加したり、設定をカスタマイズすることで、より高度なWebサービスを構築してみてください。

## 参考資料
- [Flask公式ドキュメント](https://msiz07-flask-docs-ja.readthedocs.io/ja/latest/)
- [Gunicorn公式ドキュメント](https://docs.gunicorn.org/en/stable/)
- [Docker公式ドキュメント](https://docs.docker.jp/)
- [Google Cloud Run公式ドキュメント](https://cloud.google.com/run/docs?hl=ja)
