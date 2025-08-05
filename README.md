### はじめに：必要なもの

1.  **Python 3.x**: 最新版を推奨します。
2.  **SQL Server**: 接続テストに使用するデータベース。
3.  **コマンドプロンプト**または**PowerShell**: コマンド実行に使用します。

-----

### ステップ1: Pythonのインストール

WindowsにPythonをインストールする際は、以下の点に注意してください。

1.  **公式サイトからダウンロード**: [Python公式サイト](https://www.python.org/downloads/windows/)からインストーラー（`Windows installer`）をダウンロードします。
2.  **インストーラーの実行**: ダウンロードしたインストーラーを実行します。
3.  **「Add Python to PATH」にチェック**: **「Install now」をクリックする前に、必ず「Add Python x.x to PATH」というチェックボックスにチェックを入れてください。** これを忘れると、コマンドプロンプトから`python`や`pip`コマンドが使えなくなり、再インストールが必要になる場合があります。

インストール後、コマンドプロンプトを開いて以下のコマンドを実行し、バージョンが表示されることを確認します。

```bash
python --version
pip --version
```

-----

### ステップ2: ODBC Driver for SQL Serverのインストール

`pyodbc`は、SQL Serverに接続するために、事前にODBC Driverがシステムにインストールされている必要があります。

1.  **公式サイトからダウンロード**: [Microsoft ODBC Driver for SQL Serverのダウンロードページ](https://learn.microsoft.com/ja-jp/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver16)にアクセスします。
2.  **OSとバージョンを選択**: ご自身のWindows環境に合わせて、最新の**64-bit版**（x64）のインストーラーをダウンロードします。
3.  **インストーラーの実行**: ダウンロードした`msodbcsql.msi`ファイルを実行し、指示に従ってインストールを進めます。

-----

### ステップ3: 仮想環境の作成とライブラリのインストール

プロジェクトごとに独立した環境を作るために、仮想環境を使用します。

1.  **プロジェクトフォルダの作成**:

    ```bash
    mkdir flask_sqlserver_test
    cd flask_sqlserver_test
    ```

2.  **仮想環境の作成**:

    ```bash
    python -m venv venv
    ```

3.  **仮想環境の有効化**:

    ```bash
    # Windowsの場合
    .\venv\Scripts\activate
    ```

    コマンドプロンプトの左側に `(venv)` と表示されれば成功です。

4.  **ライブラリのインストール**:
    仮想環境が有効な状態で、以下のコマンドを実行して必要なライブラリをインストールします。

    ```bash
    pip install Flask pyodbc
    ```

-----

### ステップ4: サンプルコードの作成

プロジェクトフォルダ内に `app.py` というファイルを作成し、以下のコードを貼り付けます。

#### **サンプルコード (app.py)**

```python
from flask import Flask
import pyodbc

app = Flask(__name__)

# SQL Serverの接続情報
# *** ここをあなたの環境に合わせて変更してください ***
# サーバー名、データベース名、ユーザー名、パスワードを正確に設定します
# SQL Server Expressの場合、サーバー名は「(local)\SQLEXPRESS」のようになります。
# その場合は `SERVER='(local)\\SQLEXPRESS'` と、バックスラッシュをエスケープしてください。
SERVER = 'your_server_name'
DATABASE = 'your_database_name'
USERNAME = 'your_username'
PASSWORD = 'your_password'

# 接続文字列
# インストールしたODBC Driverのバージョンに合わせる必要があります。
# 最新版であれば 'ODBC Driver 17 for SQL Server' または 'ODBC Driver 18 for SQL Server'
# を使用します。
connection_string = f'DRIVER={{ODBC Driver 17 for SQL Server}};SERVER={SERVER};DATABASE={DATABASE};UID={USERNAME};PWD={PASSWORD}'

@app.route('/')
def index():
    return '<h1>Flask SQL Server接続テストアプリです。/test_db_connection にアクセスしてください。</h1>'

@app.route('/test_db_connection')
def test_db_connection():
    try:
        # pyodbc.connect()でデータベースへの接続を試みます
        conn = pyodbc.connect(connection_string)
        
        # 接続が成功したら、簡単なクエリを実行
        cursor = conn.cursor()
        cursor.execute('SELECT @@VERSION') # SQL Serverのバージョン情報を取得するクエリ
        
        # 実行結果を取得
        row = cursor.fetchone()
        
        # 接続を閉じる
        conn.close()

        # 成功メッセージと結果を返す
        return f'<h1>Database connection successful!</h1><p>SQL Server Version: {row[0]}</p>'
    except pyodbc.Error as ex:
        # 接続エラーが発生した場合、エラーメッセージを取得
        sqlstate = ex.args[0]
        return f'<h1>Database connection failed: {sqlstate}</h1><p>詳細: {ex}</p>'

if __name__ == '__main__':
    app.run(debug=True)
```

このサンプルコードでは、単純な`SELECT 1`ではなく、`SELECT @@VERSION`というクエリを実行して、接続だけでなく実際にデータベースから情報を取得できることを確認しています。

-----

### ステップ5: 実行と確認

1.  **Flaskアプリケーションの起動**:
    `app.py`を保存し、仮想環境が有効な状態で以下のコマンドを実行します。

    ```bash
    flask run
    ```

2.  **ブラウザで確認**:
    ブラウザを開き、`http://127.0.0.1:5000/test_db_connection`にアクセスします。

      * **成功の場合**:
        `Database connection successful!` という見出しと、SQL Serverのバージョン情報が表示されます。
      * **失敗の場合**:
        `Database connection failed:` という見出しとともに、エラーの詳細が表示されます。エラーメッセージを参考に、接続情報やODBC Driverのインストール状況を再度確認してください。

この手順に従えば、Windows環境でもPythonとFlaskを使って、SQL Serverへの接続テストを簡単に行うことができます。
