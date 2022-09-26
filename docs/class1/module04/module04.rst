
設定内容を確認します

.. code-block:: cmdin

  cat ~/f5j-nginx-plus-lab2-conf/lab/connlimit1.conf

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 1-8

- 1-8行目が、ロードバランシングに該当する設定となります
- 3行目に ``ip_hash`` と記述しており、送信元IPアドレスに応じて転送先を決定する動作となります


設定を反映します

.. code-block:: cmdin

  sudo cp ~/f5j-nginx-plus-lab2-conf/lab/connlimit1.conf /etc/nginx/conf.d/default.conf
  sudo nginx -s reload

エラーが出力されないことを確認してください

動作確認
----


動作を確認します。

| ステータスを確認するためNGINX Plusのダッシュボードを開いてください。
| 画面上部 ``HTTP Upstreams`` のタブを選択してください。

以下のコマンドを実行し、動作を確認します。

.. code-block:: cmdin

  curl -I -s localhost &

コマンドの入力結果は以下となります。

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 1,3,6,15

コマンドの入力結果は以下となります。

Error Log の内容を確認します

.. code-block:: cmdin

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

Access Log の内容を確認します

.. code-block:: cmdin

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

NGINX Plus Dashboardの内容は以下のように表示されます

.. image:: ./media/nginx-connlimit1.jpg
   :width: 200

- ``Server`` の列に、ポート番号 ``81`` 、 ``82`` 、 ``83`` の3つが宛先として表示されています
- ``Requests`` の列を見ると、各 ``2`` となっており、均一に分散されていることが確認できます
- 右端 ``Response time`` の列を見ると、 ``83`` のホストは応答が遅いことが確認できますが、その応答状況に関わらず均一の分散となっています


########################
####################

NGINX Plus 冗長構成
####

1. 冗長構成
====

設定
----

動作確認
----

2. 設定同期
====

設定
----

動作確認
----

2. ステータス同期
====

1. limit Request
----

設定
~~~~

動作確認
~~~~

2. Key Value Store
----

設定
~~~~

動作確認
~~~~

