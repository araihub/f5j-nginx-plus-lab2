NGINX Plus 認証・認可の設定
####

1. mTLS / SSL証明書認証
====

クライアントから正しい証明書が提示されたことを検証し、接続を許可する設定です

.. image:: ./media/nginx-mtls-slide.jpg
   :width: 500

1. mTLS / SSL証明書認証の実施
----

利用する証明書の作成
~~~~

まずCAで利用するルート証明書を作成します

.. code-block:: cmdin

  cd ~/f5j-nginx-plus-lab2-conf/ssl
  openssl ecparam -out ./CA.key -name prime256v1 -genkey
  openssl req -new -key ./CA.key -out ./CA-csr.pem -subj '/C=JP/ST=Tokyo/O=EXAMPLE COM/CN=ROOT EXAMPLE COM/emailAddress=admin@example.com'
  openssl req -x509 -nodes -days 3650 -key ./CA.key -in ./CA-csr.pem -out ./CA.pem

SSLを終端する際に利用するサーバ証明書を作成します

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  openssl ecparam -out ./SERVER.key -name prime256v1 -genkey 
  openssl req -new -key ./SERVER.key -out ./SERVER-csr.pem -subj '/C=JP/ST=Tokyo/O=EXAMPLE COM/CN=webapp.example.com/emailAddress=admin@example.com' 
  openssl ca -config ./openssl.cnf -in SERVER-csr.pem -out SERVER.pem

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 30,33

  Using configuration from ./openssl.cnf
  Check that the request matches the signature
  Signature ok
  Certificate Details:
          Serial Number: 1 (0x1)
          Validity
              Not Before: Sep 26 10:35:40 2022 GMT
              Not After : Sep 26 10:35:40 2023 GMT
          Subject:
              countryName               = JP
              stateOrProvinceName       = Tokyo
              organizationName          = EXAMPLE COM
              commonName                = webapp.example.com
              emailAddress              = admin@example.com
          X509v3 extensions:
              X509v3 Basic Constraints:
                  CA:TRUE
              Netscape Cert Type:
                  SSL Client, SSL Server, SSL CA, S/MIME CA
              X509v3 Key Usage:
                  Digital Signature, Non Repudiation, Key Encipherment, Certificate Sign, CRL Sign
              Netscape Comment:
                  OpenSSL Generated Certificate
              X509v3 Subject Key Identifier:
                  19:66:FD:E6:4F:36:A8:87:42:B7:64:27:FB:7E:95:96:4D:94:14:0A
              X509v3 Authority Key Identifier:
                  keyid:D0:01:CB:60:EF:22:4C:DB:E4:0F:F1:83:DC:A9:42:43:B8:4D:45:98
  
  Certificate is to be certified until Sep 26 10:35:40 2023 GMT (365 days)
  Sign the certificate? [y/n]:y  << "y" と入力してください
  
  
  1 out of 1 certificate requests certified, commit? [y/n]y  << "y" と入力してください
  Write out database with 1 new entries
  Data Base Updated


クライアント証明書認証で利用する証明書で必要となるCSRを作成します

まず１つ目のクライアント証明書を作成します

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  openssl ecparam -out ./CLIENT1.key -name prime256v1 -genkey 
  openssl req -new -key ./CLIENT1.key -out ./CLIENT1-csr.pem -subj '/C=JP/ST=Tokyo/O=EXAMPLE COM/CN=client1.example.com/emailAddress=admin@example.com' 
  openssl ca -config ./openssl.cnf -in CLIENT1-csr.pem -out CLIENT1.pem

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 26,29

  Using configuration from ./openssl.cnf
  Check that the request matches the signature
  Signature ok
  Certificate Details:
          Serial Number: 2 (0x2)
          Validity
              Not Before: Sep 26 06:47:31 2022 GMT
              Not After : Sep 26 06:47:31 2023 GMT
          Subject:
              countryName               = JP
              stateOrProvinceName       = Tokyo
              organizationName          = EXAMPLE COM
              commonName                = client1.example.com
              emailAddress              = admin@example.com
          X509v3 extensions:
              X509v3 Basic Constraints:
                  CA:FALSE
              Netscape Comment:
                  OpenSSL Generated Certificate
              X509v3 Subject Key Identifier:
                  1D:43:87:C8:DE:89:E6:10:5F:27:79:F3:CB:50:A6:32:4F:D4:97:3B
              X509v3 Authority Key Identifier:
                  keyid:0E:1E:B3:B3:0F:1C:7D:D6:C1:A6:4F:E7:D4:5F:EE:B7:96:72:F3:64
  
  Certificate is to be certified until Sep 26 06:47:31 2023 GMT (365 days)
  Sign the certificate? [y/n]:y  << "y" と入力してください
  
  
  1 out of 1 certificate requests certified, commit? [y/n]y  << "y" と入力してください
  Write out database with 1 new entries
  Data Base Updated


次に２つ目のクライアント証明書を作成します

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  openssl ecparam -out ./CLIENT2.key -name prime256v1 -genkey 
  openssl req -new -key ./CLIENT2.key -out ./CLIENT2-csr.pem -subj '/C=JP/ST=Tokyo/O=EXAMPLE COM/CN=client2.example.com/emailAddress=admin@example.com' 
  openssl ca -config ./openssl.cnf -in CLIENT2-csr.pem -out CLIENT2.pem

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 30,33

  Using configuration from ./openssl.cnf
  Check that the request matches the signature
  Signature ok
  Certificate Details:
          Serial Number: 3 (0x3)
          Validity
              Not Before: Sep 26 10:37:44 2022 GMT
              Not After : Sep 26 10:37:44 2023 GMT
          Subject:
              countryName               = JP
              stateOrProvinceName       = Tokyo
              organizationName          = EXAMPLE COM
              commonName                = client2.example.com
              emailAddress              = admin@example.com
          X509v3 extensions:
              X509v3 Basic Constraints:
                  CA:TRUE
              Netscape Cert Type:
                  SSL Client, SSL Server, SSL CA, S/MIME CA
              X509v3 Key Usage:
                  Digital Signature, Non Repudiation, Key Encipherment, Certificate Sign, CRL Sign
              Netscape Comment:
                  OpenSSL Generated Certificate
              X509v3 Subject Key Identifier:
                  84:E0:0F:2F:8C:37:62:F8:28:4C:7E:C4:A5:53:FF:19:76:39:B6:6A
              X509v3 Authority Key Identifier:
                  keyid:D0:01:CB:60:EF:22:4C:DB:E4:0F:F1:83:DC:A9:42:43:B8:4D:45:98
  
  Certificate is to be certified until Sep 26 10:37:44 2023 GMT (365 days)
  Sign the certificate? [y/n]:y  << "y" と入力してください
  
  
  1 out of 1 certificate requests certified, commit? [y/n]y  << "y" と入力してください
  Write out database with 1 new entries
  Data Base Updated


``index.txt`` の内容に作成した証明書の情報が記録されていることを確認してください。

.. code-block:: cmdin

  cat index.txt

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  V       230926103540Z           01      unknown /C=JP/ST=Tokyo/O=EXAMPLE COM/CN=webapp.example.com/emailAddress=admin@example.com
  V       230926103629Z           02      unknown /C=JP/ST=Tokyo/O=EXAMPLE COM/CN=client1.example.com/emailAddress=admin@example.com
  V       230926103744Z           03      unknown /C=JP/ST=Tokyo/O=EXAMPLE COM/CN=client2.example.com/emailAddress=admin@example.com

また参考に以下の情報も確認してください

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  $ cat serial
  04
  $ ls newcerts/
  01.pem  02.pem  03.pem

必要となるファイルをコピーします

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  sudo mkdir /etc/nginx/conf.d/ssl
  sudo cp SERVER.key /etc/nginx/conf.d/ssl
  sudo cp SERVER.pem /etc/nginx/conf.d/ssl
  sudo cp CA.pem /etc/nginx/conf.d/ssl


設定
~~~~

設定内容を確認します

.. code-block:: cmdin

  cat ~/f5j-nginx-plus-lab2-conf/lab/mtls1.conf

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 8-10,12-13

  upstream server_group {
      zone backend 64k;
  
      server backend1:81;
  }
  
  server {
     listen 443 ssl;
     ssl_certificate_key conf.d/ssl/SERVER.key;
     ssl_certificate conf.d/ssl/SERVER.pem;
  
     ssl_client_certificate conf.d/ssl/CA.pem;
     ssl_verify_client on;
  
     location / {
         proxy_pass http://server_group;
     }
  }

- 8-10行目で、SSLを終端する設定とします
- 12-13行目で、SSL証明書認証を行う設定となります


設定を反映します

.. code-block:: cmdin

  sudo cp -r ~/f5j-nginx-plus-lab2-conf/ssl /etc/nginx/conf.d/
  sudo cp ~/f5j-nginx-plus-lab2-conf/lab/mtls1.conf /etc/nginx/conf.d/default.conf
  sudo nginx -s reload

動作確認
~~~~

クライアント証明書を提示せず、通信を行います

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  curl -v --cacert ./CA.pem https://webapp.example.com --resolve webapp.example.com:443:127.0.0.1

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 37,45,48

  * Added webapp.example.com:443:127.0.0.1 to DNS cache
  * Hostname webapp.example.com was found in DNS cache
  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to webapp.example.com (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: ./CA.pem
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.2 (IN), TLS handshake, Certificate (11):
  * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
  * TLSv1.2 (IN), TLS handshake, Request CERT (13):
  * TLSv1.2 (IN), TLS handshake, Server finished (14):
  * TLSv1.2 (OUT), TLS handshake, Certificate (11):
  * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
  * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.2 (OUT), TLS handshake, Finished (20):
  * TLSv1.2 (IN), TLS handshake, Finished (20):
  * SSL connection using TLSv1.2 / ECDHE-ECDSA-AES256-GCM-SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: C=JP; ST=Tokyo; O=EXAMPLE COM; CN=webapp.example.com; emailAddress=admin@example.com
  *  start date: Sep 26 10:58:45 2022 GMT
  *  expire date: Sep 26 10:58:45 2023 GMT
  *  common name: webapp.example.com (matched)
  *  issuer: C=JP; ST=Tokyo; O=EXAMPLE COM; CN=ROOT EXAMPLE COM; emailAddress=admin@example.com
  *  SSL certificate verify ok.
  > GET / HTTP/1.1
  > Host: webapp.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 400 Bad Request
  < Server: nginx/1.21.6
  < Date: Mon, 26 Sep 2022 11:01:38 GMT
  < Content-Type: text/html
  < Content-Length: 237
  < Connection: close
  <
  <html>
  <head><title>400 No required SSL certificate was sent</title></head>
  <body>
  <center><h1>400 Bad Request</h1></center>
  <center>No required SSL certificate was sent</center>
  <hr><center>nginx/1.21.6</center>
  </body>
  </html>
  * Closing connection 0
  * TLSv1.2 (OUT), TLS alert, close notify (256):

37、45、48行目で示す通り、SSL証明書が正しく提示されないためエラーとなっています

次に、１つ目のクライアントを示す証明書を使い通信を行います。

.. code-block:: cmdin

  curl -v --cacert ./CA.pem --key ./CLIENT1.key --cert ./CLIENT1.pem https://webapp.example.com --resolve webapp.example.com:443:127.0.0.1

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 38

  * Added webapp.example.com:443:127.0.0.1 to DNS cache
  * Hostname webapp.example.com was found in DNS cache
  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to webapp.example.com (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: ./CA.pem
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.2 (IN), TLS handshake, Certificate (11):
  * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
  * TLSv1.2 (IN), TLS handshake, Request CERT (13):
  * TLSv1.2 (IN), TLS handshake, Server finished (14):
  * TLSv1.2 (OUT), TLS handshake, Certificate (11):
  * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
  * TLSv1.2 (OUT), TLS handshake, CERT verify (15):
  * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.2 (OUT), TLS handshake, Finished (20):
  * TLSv1.2 (IN), TLS handshake, Finished (20):
  * SSL connection using TLSv1.2 / ECDHE-ECDSA-AES256-GCM-SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: C=JP; ST=Tokyo; O=EXAMPLE COM; CN=webapp.example.com; emailAddress=admin@example.com
  *  start date: Sep 26 10:58:45 2022 GMT
  *  expire date: Sep 26 10:58:45 2023 GMT
  *  common name: webapp.example.com (matched)
  *  issuer: C=JP; ST=Tokyo; O=EXAMPLE COM; CN=ROOT EXAMPLE COM; emailAddress=admin@example.com
  *  SSL certificate verify ok.
  > GET / HTTP/1.1
  > Host: webapp.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.21.6
  < Date: Mon, 26 Sep 2022 11:35:15 GMT
  < Content-Type: application/octet-stream
  < Content-Length: 65
  < Connection: keep-alive
  <
  * Connection #0 to host webapp.example.com left intact
  { "request_uri": "/","server_addr":"10.1.1.8","server_port":"81"}

38行目で ``200 OK`` が応答されておりエラーなく正しい応答が帰ってきていることが確認できます。

２つ目のクライアントのファイルを利用して動作確認をいただいた場合にも同様の内容が応答されることを確認いただけます。


2. 証明書のRevoke時の動作
----

証明書のRevoke
~~~~

2つ目のクライアントの証明書をRevoke(利用を停止)し、その際の動作を確認します

以下コマンドを入力します

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  openssl ca -config ./openssl.cnf -gencrl -revoke CLIENT2.pem

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  Using configuration from ./openssl.cnf
  -----BEGIN X509 CRL-----
  MIIBCDCBrwIBATAKBggqhkjOPQQDAjBwMQswCQYDVQQGEwJKUDEOMAwGA1UECAwF
  VG9reW8xFDASBgNVBAoMC0VYQU1QTEUgQ09NMRkwFwYDVQQDDBBST09UIEVYQU1Q
  TEUgQ09NMSAwHgYJKoZIhvcNAQkBFhFhZG1pbkBleGFtcGxlLmNvbRcNMjIwOTI2
  MTEwNDA3WhcNMjIxMDI2MTEwNDA3WqAOMAwwCgYDVR0UBAMCAQEwCgYIKoZIzj0E
  AwIDSAAwRQIgbZViSMalmcHC+W4JP5+78PGTEPTS/DuiXFeMXx4t85wCIQC7c/av
  7L1t/g0B+m1Ls2XwilqS/zJsuMq1NnWJ7SRn9Q==
  -----END X509 CRL-----
  Revoking Certificate 03.
  Data Base Updated


``index.txt`` の結果を確認してください。Revokeを行った ``CLIENT2.pem`` の先頭が ``R`` と表示されています

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  cat index.txt

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 3

  V       230926105845Z           01      unknown /C=JP/ST=Tokyo/O=EXAMPLE COM/CN=webapp.example.com/emailAddress=admin@example.com
  V       230926105859Z           02      unknown /C=JP/ST=Tokyo/O=EXAMPLE COM/CN=client1.example.com/emailAddress=admin@example.com
  R       230926105912Z   220926110407Z   03      unknown /C=JP/ST=Tokyo/O=EXAMPLE COM/CN=client2.example.com/emailAddress=admin@example.com

以下コマンドを実行し、 CRLファイルとして ``crl.pem`` を作成します

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  openssl ca -config ./openssl.cnf -gencrl -out CRL.pem

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  Using configuration from ./openssl.cnf

作成したCRLの情報を表示し、確認します

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  openssl crl -inform pem -in CRL.pem -text

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 10-13

  Certificate Revocation List (CRL):
          Version 2 (0x1)
          Signature Algorithm: ecdsa-with-SHA256
          Issuer: C = JP, ST = Tokyo, O = EXAMPLE COM, CN = ROOT EXAMPLE COM, emailAddress = admin@example.com
          Last Update: Sep 26 11:04:16 2022 GMT
          Next Update: Oct 26 11:04:16 2022 GMT
          CRL extensions:
              X509v3 CRL Number:
                  2
  Revoked Certificates:
      Serial Number: 03
          Revocation Date: Sep 26 11:04:07 2022 GMT
      Signature Algorithm: ecdsa-with-SHA256
           ** 省略 **
  -----BEGIN X509 CRL-----
  ** 省略 **
  -----END X509 CRL-----

10-13行目にRevokeされた証明書の情報が記述されており、 ``Serial Number 03`` が対象となっていることが確認できます

設定
~~~~

設定内容を確認します

.. code-block:: cmdin

  cat ~/f5j-nginx-plus-lab2-conf/lab/mtls2-revoke.conf

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 13

  upstream server_group {
      zone backend 64k;
  
      server backend1:81;
  }
  
  server {
     listen 443 ssl;
     ssl_certificate_key conf.d/ssl/SERVER.key;
     ssl_certificate conf.d/ssl/SERVER.pem;
  
     ssl_client_certificate conf.d/ssl/CA.pem;
     ssl_crl conf.d/ssl/CRL.pem;
     ssl_verify_client on;
  
     location / {
         proxy_pass http://server_group;
     }
  }

13行目でCRLファイルを参照しています

設定を反映します

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  sudo cp CRL.pem /etc/nginx/conf.d/ssl
  sudo cp ~/f5j-nginx-plus-lab2-conf/lab/mtls2-revoke.conf /etc/nginx/conf.d/default.conf
  sudo nginx -s reload

動作確認
~~~~

証明書をRevokeしたクライアントの動作を確認します

.. code-block:: cmdin

  # cd ~/f5j-nginx-plus-lab2-conf/ssl
  curl -v --cacert ./CA.pem --key ./CLIENT2.key --cert ./CLIENT2.pem https://webapp.example.com --resolve webapp.example.com:443:127.0.0.1

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 38,46,48-49

  * Added webapp.example.com:443:127.0.0.1 to DNS cache
  * Hostname webapp.example.com was found in DNS cache
  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to webapp.example.com (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: ./CA.pem
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.2 (IN), TLS handshake, Certificate (11):
  * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
  * TLSv1.2 (IN), TLS handshake, Request CERT (13):
  * TLSv1.2 (IN), TLS handshake, Server finished (14):
  * TLSv1.2 (OUT), TLS handshake, Certificate (11):
  * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
  * TLSv1.2 (OUT), TLS handshake, CERT verify (15):
  * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.2 (OUT), TLS handshake, Finished (20):
  * TLSv1.2 (IN), TLS handshake, Finished (20):
  * SSL connection using TLSv1.2 / ECDHE-ECDSA-AES256-GCM-SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: C=JP; ST=Tokyo; O=EXAMPLE COM; CN=webapp.example.com; emailAddress=admin@example.com
  *  start date: Sep 26 10:58:45 2022 GMT
  *  expire date: Sep 26 10:58:45 2023 GMT
  *  common name: webapp.example.com (matched)
  *  issuer: C=JP; ST=Tokyo; O=EXAMPLE COM; CN=ROOT EXAMPLE COM; emailAddress=admin@example.com
  *  SSL certificate verify ok.
  > GET / HTTP/1.1
  > Host: webapp.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 400 Bad Request
  < Server: nginx/1.21.6
  < Date: Mon, 26 Sep 2022 11:07:13 GMT
  < Content-Type: text/html
  < Content-Length: 215
  < Connection: close
  <
  <html>
  <head><title>400 The SSL certificate error</title></head>
  <body>
  <center><h1>400 Bad Request</h1></center>
  <center>The SSL certificate error</center>
  <hr><center>nginx/1.21.6</center>
  </body>
  </html>
  * Closing connection 0
  * TLSv1.2 (OUT), TLS alert, close notify (256):

- 38,46,48-49行目で ``400 Bad Request`` のエラーが表示されていることが確認できます
- 49行目の内容を確認すると、 ``The SSL certificate error`` と表示されていることが確認できます

参考に、Revokeを行っていないクライアントで再度アクセスし、エラーなく結果が表示できることを確認します

.. code-block:: cmdin

  curl -v --cacert ./CA.pem --key ./CLIENT1.key --cert ./CLIENT1.pem https://webapp.example.com --resolve webapp.example.com:443:127.0.0.1

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 38

  * Added webapp.example.com:443:127.0.0.1 to DNS cache
  * Hostname webapp.example.com was found in DNS cache
  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to webapp.example.com (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: ./CA.pem
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.2 (IN), TLS handshake, Certificate (11):
  * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
  * TLSv1.2 (IN), TLS handshake, Request CERT (13):
  * TLSv1.2 (IN), TLS handshake, Server finished (14):
  * TLSv1.2 (OUT), TLS handshake, Certificate (11):
  * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
  * TLSv1.2 (OUT), TLS handshake, CERT verify (15):
  * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.2 (OUT), TLS handshake, Finished (20):
  * TLSv1.2 (IN), TLS handshake, Finished (20):
  * SSL connection using TLSv1.2 / ECDHE-ECDSA-AES256-GCM-SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: C=JP; ST=Tokyo; O=EXAMPLE COM; CN=webapp.example.com; emailAddress=admin@example.com
  *  start date: Sep 26 10:58:45 2022 GMT
  *  expire date: Sep 26 10:58:45 2023 GMT
  *  common name: webapp.example.com (matched)
  *  issuer: C=JP; ST=Tokyo; O=EXAMPLE COM; CN=ROOT EXAMPLE COM; emailAddress=admin@example.com
  *  SSL certificate verify ok.
  > GET / HTTP/1.1
  > Host: webapp.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.21.6
  < Date: Mon, 26 Sep 2022 11:06:50 GMT
  < Content-Type: application/octet-stream
  < Content-Length: 65
  < Connection: keep-alive
  <
  * Connection #0 to host webapp.example.com left intact
  { "request_uri": "/","server_addr":"10.1.1.8","server_port":"81"}
  

2. Basic認証
====

Webサーバなどで用いられるシンプルなユーザ認証であるBasic認証の動作を確認します

.. image:: ./media/nginx-basic-slide.jpg
   :width: 500

設定
----

設定内容を確認します

.. code-block:: cmdin

  cat ~/f5j-nginx-plus-lab2-conf/lab/basicauth.conf

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 12-14

  upstream server_group {
      zone backend 64k;
  
      server backend1:81;
  }
  
  server {
     listen 80;
     location / {
         proxy_pass http://server_group;
     }
     location /auth {
         auth_basic           "Administrator’s Area";
         auth_basic_user_file conf.d/password/htpasswd;
         proxy_pass http://server_group;
     }
  }

- 12行目で、 ``/auth`` という認証を実施するPATHを作成します
- 13行目で、ベーシック認証を有効にします
- 14行目で、ユーザの認証情報に利用する htpasswd のファイルを指定します

設定を反映します

.. code-block:: cmdin

  sudo cp -r ~/f5j-nginx-plus-lab2-conf/password /etc/nginx/conf.d/
  sudo cp ~/f5j-nginx-plus-lab2-conf/lab/basicauth.conf /etc/nginx/conf.d/default.conf
  sudo nginx -s reload

動作確認
----

Curlコマンドで対象のPATHにアクセスします

.. code-block:: cmdin

  curl -v -s localhost/auth

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 10,16,19,21

  *   Trying 127.0.0.1:80...
  * TCP_NODELAY set
  * Connected to localhost (127.0.0.1) port 80 (#0)
  > GET /auth HTTP/1.1
  > Host: localhost
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 401 Unauthorized
  < Server: nginx/1.21.6
  < Date: Mon, 26 Sep 2022 13:27:31 GMT
  < Content-Type: text/html
  < Content-Length: 179
  < Connection: keep-alive
  < WWW-Authenticate: Basic realm="Administrator’s Area"
  <
  <html>
  <head><title>401 Authorization Required</title></head>
  <body>
  <center><h1>401 Authorization Required</h1></center>
  <hr><center>nginx/1.21.6</center>
  </body>
  </html>
  * Connection #0 to host localhost left intact

- ``401 Unauthorized`` のエラーが表示されます
- 16行目で、Basic認証が設定されていることが確認できます

httpasswd の内容は以下のユーザ情報を記述しています

+-----+--------+
|User |Password|
+=====+========+
|user1|user1   |
+-----+--------+
|user2|user2   |
+-----+--------+
|user3|user3   |
+-----+--------+

対象のPATHに対して ユーザ名 ``user1`` パスワード ``user1`` を指定し、動作を確認します

.. code-block:: cmdin

  curl -v -s -u user1:user1 localhost/auth

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 12,20
  
  *   Trying 127.0.0.1:80...
  * TCP_NODELAY set
  * Connected to localhost (127.0.0.1) port 80 (#0)
  * Server auth using Basic with user 'user1'
  > GET /auth HTTP/1.1
  > Host: localhost
  > Authorization: Basic dXNlcjE6dXNlcjE=
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.21.6
  < Date: Mon, 26 Sep 2022 13:33:09 GMT
  < Content-Type: application/octet-stream
  < Content-Length: 69
  < Connection: keep-alive
  <
  * Connection #0 to host localhost left intact
  { "request_uri": "/auth","server_addr":"10.1.1.8","server_port":"81"}


エラーなく正しく表示されていることが確認できます

3. JWTによる通信制御
====

NGINX Plus は JWT の検証が可能です。またJWT Claimにアクセスし、様々な通信制御を実施することが可能です

.. image:: ./media/nginx-jwt-slide.jpg
   :width: 500

1. JWT Validation
----


利用するJWTを確認
~~~~

利用するファイルの内容を確認します

JWK(Json Web Key)の内容を確認します


出力結果が以下となります。

.. code-block:: cmdin

  cat ~/f5j-nginx-plus-lab2-conf/jwt/api_secret.jwk

.. code-block:: json
  :linenos:
  :caption: jwk を base64 デコードした結果
  :emphasize-lines: 3

  {"keys":
      [{
          "k":"ZmFudGFzdGljand0",
          "kty":"oct",
          "kid":"0001"
      }]
  }


各パラメータ内容は以下の通り

.. list-table::
    :widths: 2 6 2 
    :header-rows: 1
    :stub-columns: 1

    * - **Parameter**
      - **意味**
      - **Link**
    * - k
      - k (key value) パラメータは, kty octで利用する base64url encodeされたKey文字列をもつ
      - `JSON Web Algorithms (JWA) 6.4.1 "k" <https://www.rfc-editor.org/rfc/rfc7518.txt>`__
    * - kty
      - kty (key type) パラメータは, RSA や EC といった暗号アルゴリズムファミリーを示す
      - `JSON Web Key (JWK) 4.1 "kty" <https://openid-foundation-japan.github.io/rfc7517.ja.html#ktyDef>`__
    * - kid
      - kid (key ID) パラメータは特定の鍵を識別するために用いられる
      - `JSON Web Key (JWK) 4.5 "kid" <https://openid-foundation-japan.github.io/rfc7517.ja.html#kidDef>`__


kty "oct" で利用する Keyの内容をBase64デコードした結果は以下の通り

.. code-block:: cmdin

  echo -n "ZmFudGFzdGljand0" | base64 -d

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  fantasticjwt

この結果により、このサンプルでは ``fantasticjwt`` という文字列がKeyとして使用されていることが確認できます。

今回サンプルリクエストに利用するJWTがこの文字列で署名されたものであるか確認します。 
``~/f5j-nginx-plus-lab2-conf/jwt/nginx1.jwt`` の内容を表示します。

| `JWT.io <https://jwt.io/>`__ を開き、 **Algorithm** が ``HS256`` であることを確認します。
| **VERIFY SIGNATURE** 欄の ``your-256-bit-secret`` に先程 ``jwk`` の内容をデコードして確認した文字列 ``fantasticjwt`` を入力してください。
| 画面左側 **Eocoded** 欄に、 ``nginx1.jwt`` の内容を貼り付け、左下の表示が ``Signature Verified`` となることを確認してください。
| この結果より、クライアントリクエストで利用するJWTは、検証可能なものであることが確認できます。またこのJWTに含まれる情報が右側に表示されますので合わせて確認ください。

   .. image:: ./media/jwtio_verify.jpg
      :width: 500

その他、NGINX Plus / JWT に関する詳細は 
`Blog:Authenticating API Clients with JWT and NGINX Plus <https://www.nginx.com/blog/authenticating-api-clients-jwt-nginx-plus/>`__ 
を参照してください


設定
~~~~

設定内容を確認します

.. code-block:: cmdin

  cat ~/f5j-nginx-plus-lab2-conf/lab/jwt1.conf

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 12-14

  upstream server_group {
      zone backend 64k;
  
      server app-backend1:8080;
  }
  
  server {
     listen 80;
     location / {
         proxy_pass http://server_group;
     }
     location /auth {
         auth_jwt "Products API";
         auth_jwt_key_file conf.d/jwt/api_secret.jwk;
         proxy_pass http://server_group;
     }
  }

JWTの動作確認では、HTTPリクエストの情報を応答するサーバを転送先として指定しています。

設定を反映します

.. code-block:: cmdin

  sudo cp -r ~/f5j-nginx-plus-lab2-conf/jwt/ /etc/nginx/conf.d/
  sudo cp ~/f5j-nginx-plus-lab2-conf/lab/jwt1.conf /etc/nginx/conf.d/default.conf
  sudo nginx -s reload

動作確認
~~~~

PATH ``/auth`` に対し、JWTを含まないリクエストを送信し、動作を確認します

.. code-block:: cmdin

  curl -v localhost/auth

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 10,16,19,21

  *   Trying 127.0.0.1:80...
  * TCP_NODELAY set
  * Connected to localhost (127.0.0.1) port 80 (#0)
  > GET /auth HTTP/1.1
  > Host: localhost
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 401 Unauthorized
  < Server: nginx/1.21.6
  < Date: Mon, 26 Sep 2022 14:41:47 GMT
  < Content-Type: text/html
  < Content-Length: 179
  < Connection: keep-alive
  < WWW-Authenticate: Bearer realm="Products API"
  <
  <html>
  <head><title>401 Authorization Required</title></head>
  <body>
  <center><h1>401 Authorization Required</h1></center>
  <hr><center>nginx/1.21.6</center>
  </body>
  </html>
  * Connection #0 to host localhost left intact

- ``401 Authorization Required`` とエラーが表示され、通信が拒否されていることが確認できます

次に正しいJWTを含むリクエストを送り、結果を確認します

.. code-block:: cmdin

  curl -v localhost/auth -H "Authorization: Bearer `cat ~/f5j-nginx-plus-lab2-conf/jwt/nginx1.jwt`"

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 8,11
  
  *   Trying 127.0.0.1:80...
  * TCP_NODELAY set
  * Connected to localhost (127.0.0.1) port 80 (#0)
  > GET /auth HTTP/1.1
  > Host: localhost
  > User-Agent: curl/7.68.0
  > Accept: */*
  > Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6IjAwMDEifQ.eyJpc3MiOiJNeSBJRFAiLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoibmdpbngtcGx1cyIsInNjb3BlIjoicHJvZmlsZSBlbWFpbCIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6Im5naW54MSB1c2VyIiwicHJlZmVycmVkX3VzZXJuYW1lIjoibmdpbngxLXVzZXIiLCJnaXZlbl9uYW1lIjoibmdpbngxIiwiZmFtaWx5X25hbWUiOiJ1c2VyIiwiZW1haWwiOiJuZ2lueDFAZXhhbXBsZS5jb20ifQ.sOKct6cXUVpHdbF7s7U46LRXOWxzPZVZPL4hSaLiFoE
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.21.6
  < Date: Mon, 26 Sep 2022 14:47:08 GMT
  < Content-Type: application/octet-stream
  < Content-Length: 940
  < Connection: keep-alive
  <
  * Connection #0 to host localhost left intact
  {"request":{"headers":[["Host","server_group"],["Connection","close"],["User-Agent","curl/7.68.0"],["Accept","*/*"],["Authorization","Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6IjAwMDEifQ.eyJpc3MiOiJNeSBJRFAiLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoibmdpbngtcGx1cyIsInNjb3BlIjoicHJvZmlsZSBlbWFpbCIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6Im5naW54MSB1c2VyIiwicHJlZmVycmVkX3VzZXJuYW1lIjoibmdpbngxLXVzZXIiLCJnaXZlbl9uYW1lIjoibmdpbngxIiwiZmFtaWx5X25hbWUiOiJ1c2VyIiwiZW1haWwiOiJuZ2lueDFAZXhhbXBsZS5jb20ifQ.sOKct6cXUVpHdbF7s7U46LRXOWxzPZVZPL4hSaLiFoE"]],"status":0,"httpversion":"1.0","method":"GET","scheme":"http","uri":"/auth","fullPath":"/auth"},"network":{"clientPort":"51744","clientAddress":"10.1.1.7","serverAddress":"172.19.0.2","serverPort":"8080"},"ssl":{"isHttps":false},"session":{"httpConnection":"close","requestId":"8f8e5fbc233a0d05683f0718f789e23b","connection":"2","connectionNumber":"1"},"environment":{"hostname":"echoapp.net"}}

- 8行目で、リクエストのAuthorization Headerに、指定した値が含まれていることがわかります
- 11行目で、 ``200 OK`` が応答されており、正しく通信ができたことが確認できます

2. JWTの詳細ログ
----

JWTは様々な情報をClaimとして保持します。それらの情報を確認します

設定
~~~~

設定内容を確認します

.. code-block:: cmdin

  cat ~/f5j-nginx-plus-lab2-conf/lab/jwt2-detailinfo.conf

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 1-3,26,20-24

  log_format jwt '$remote_addr - $remote_user [$time_local] "$request" '
                 '$status $body_bytes_sent "$http_referer" "$http_user_agent" '
                 '$jwt_header_alg $jwt_claim_sub $jwt_claim_scope $jwt_claim_name $jwt_claim_email';
  
  upstream server_group {
      zone backend 64k;
  
      server app-backend1:8080;
  }
  
  server {
     listen 80;
     location / {
         proxy_pass http://server_group;
     }
     location /auth {
         auth_jwt "Products API";
         auth_jwt_key_file conf.d/jwt/api_secret.jwk;
  
         proxy_set_header API-Client $jwt_claim_sub;
         proxy_set_header JWT-alg $jwt_header_alg;
         proxy_set_header JWT-sub $jwt_claim_sub;
         proxy_set_header JWT-scope $jwt_claim_scope;
         proxy_set_header JWT-email $jwt_claim_email;
  
         access_log /var/log/nginx/access_jwt.log jwt;
         proxy_pass http://server_group;
     }
  }


- 1-3行目で、 ``log_format`` Directive を利用して、LogFormatを記述しています。3行目に ``$jwt_`` から始まる変数を指定します
- 26行目で、 ``access_log`` Directive を利用して ``log_format`` で定義した ``jwt`` のフォーマットを指定することで、指定のファイルに指定のフォーマットでログを出力します
- 20-24行目で、 ``proxy_set_header`` Directive を利用して バックエンドサーバへ転送するリクエストにJWTの情報をHTTPヘッダーとして付与します


設定を反映します

.. code-block:: cmdin

  #sudo cp -r ~/f5j-nginx-plus-lab2-conf/jwt/ /etc/nginx/conf.d/
  sudo cp ~/f5j-nginx-plus-lab2-conf/lab/jwt2-detailinfo.conf /etc/nginx/conf.d/default.conf
  sudo nginx -s reload

動作確認
~~~~

2つのJWTを使ってリクエストを送ります。

1回目 ``nginx1`` の応答を確認します

.. code-block:: cmdin

  curl -s localhost/auth -H "Authorization: Bearer `cat ~/f5j-nginx-plus-lab2-conf/jwt/nginx1.jwt`" | jq .request.headers

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 3-4,7-8,11-12,15-16,19-20

  [
    [
      "API-Client",
      "nginx-plus"
    ],
    [
      "JWT-alg",
      "HS256"
    ],
    [
      "JWT-sub",
      "nginx-plus"
    ],
    [
      "JWT-scope",
      "profile email"
    ],
    [
      "JWT-email",
      "nginx1@example.com"
    ],
    [
      "Host",
      "server_group"
    ],
    [
      "Connection",
      "close"
    ],
    [
      "User-Agent",
      "curl/7.68.0"
    ],
    [
      "Accept",
      "*/*"
    ],
    [
      "Authorization",
      "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6IjAwMDEifQ.eyJpc3MiOiJNeSBJRFAiLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoibmdpbngtcGx1cyIsInNjb3BlIjoicHJvZmlsZSBlbWFpbCIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6Im5naW54MSB1c2VyIiwicHJlZmVycmVkX3VzZXJuYW1lIjoibmdpbngxLXVzZXIiLCJnaXZlbl9uYW1lIjoibmdpbngxIiwiZmFtaWx5X25hbWUiOiJ1c2VyIiwiZW1haWwiOiJuZ2lueDFAZXhhbXBsZS5jb20ifQ.sOKct6cXUVpHdbF7s7U46LRXOWxzPZVZPL4hSaLiFoE"
    ]
  ]

設定で付与したHTTPヘッダーの情報が、バックエンドのサーバから応答されていることが確認できます

2回目 ``nginx2`` の応答を確認します

.. code-block:: cmdin

  curl -s localhost/auth -H "Authorization: Bearer `cat ~/f5j-nginx-plus-lab2-conf/jwt/nginx2.jwt`" | jq .request.headers

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 3-4,7-8,11-12,15-16,19-20

  [
    [
      "API-Client",
      "nginx-plus"
    ],
    [
      "JWT-alg",
      "HS256"
    ],
    [
      "JWT-sub",
      "nginx-plus"
    ],
    [
      "JWT-scope",
      "profile email"
    ],
    [
      "JWT-email",
      "nginx2@example.com"
    ],
    [
      "Host",
      "server_group"
    ],
    [
      "Connection",
      "close"
    ],
    [
      "User-Agent",
      "curl/7.68.0"
    ],
    [
      "Accept",
      "*/*"
    ],
    [
      "Authorization",
      "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6IjAwMDEifQ.eyJpc3MiOiJNeSBJRFAiLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoibmdpbngtcGx1cyIsInNjb3BlIjoicHJvZmlsZSBlbWFpbCIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6Im5naW54MiB1c2VyIiwicHJlZmVycmVkX3VzZXJuYW1lIjoibmdpbngyLXVzZXIiLCJnaXZlbl9uYW1lIjoibmdpbngyIiwiZmFtaWx5X25hbWUiOiJ1c2VyIiwiZW1haWwiOiJuZ2lueDJAZXhhbXBsZS5jb20ifQ.CVUH7upnT5n2yzLnlGhNTMKL_Ev6yJfj8FHSFC5v5ME"
    ]
  ]

先程と同様に、正しくHTTPヘッダーが付与されていることが確認できます。

ログを確認します

.. code-block:: cmdin

  tail -2 /var/log/nginx/access_jwt.log

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  127.0.0.1 - - [27/Sep/2022:00:04:41 +0900] "GET /auth HTTP/1.1" 200 1079 "-" "curl/7.68.0" HS256 nginx-plus profile email nginx1 user nginx1@example.com
  127.0.0.1 - - [27/Sep/2022:00:04:48 +0900] "GET /auth HTTP/1.1" 200 1079 "-" "curl/7.68.0" HS256 nginx-plus profile email nginx2 user nginx2@example.com

nginx1, nginx2 のそれぞれの接続で正しく ``200`` が応答されており、 JWTの情報がログに記録されていることが確認できます

3. 複雑な制御
----

JWTに含まれる情報を用いて、通信を制御する動作を確認します

設定
~~~~

設定内容を確認します

.. code-block:: cmdin

  cat ~/f5j-nginx-plus-lab2-conf/lab/jwt3-complicate.conf

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 1,3-6,8-12,14,20,29,37,36,34

  limit_req_zone $jwt_claim_sub zone=1rpm_per_client:1m rate=1r/m;
  
  log_format jwt '$remote_addr - $remote_user [$time_local] "$request" '
                 '$status $body_bytes_sent "$http_referer" "$http_user_agent" '
                 '$jwt_claim_scope $jwt_claim_name $jwt_claim_email '
                 '$jwt_claim_sub $jwt_upstream';
  
  map $jwt_claim_scope $jwt_upstream {
      ~group1 "slow_group";
      ~group2 "slow_group";
      default default_group;
  }
  
  upstream slow_group {
      zone backend 64k;
  
      server app-backend1:8080;
  }
  
  upstream default_group {
      zone backend 64k;
  
      server app-backend2:8080;
  }
  
  server {
     listen 80;
     location / {
         proxy_pass http://$jwt_upstream;
     }
     location /auth {
         auth_jwt "Products API";
         auth_jwt_key_file conf.d/jwt/api_secret.jwk;
         access_log /var/log/nginx/access_jwt.log jwt;
  
         limit_req zone=1rpm_per_client;
         proxy_pass http://$jwt_upstream;
     }
  }

- 1行目で ``$jwt_claim_sub`` をKeyとした、Request Limitを設定し、36行目で、 ``/auth`` のPATHに適用します
- 8-12行目で、 ``map`` Directiveを使用し、 ``$jwt_claim_scope`` に含まれる値に応じて ``$jwt_upstream`` という変数の値を選択するよう記述します。group1 , group2 の場合には、 応答の遅いUpstreamを想定し ``slow_group`` 、 通常は ``default_group`` に転送する動作となります
- 29行目、37行目で転送先を指定しますが、宛先Upstreamの名称に ``$jwt_upstream`` を指定します

設定を反映します

.. code-block:: cmdin

  #sudo cp -r ~/f5j-nginx-plus-lab2-conf/jwt/ /etc/nginx/conf.d/
  sudo cp ~/f5j-nginx-plus-lab2-conf/lab/jwt3-complicate.conf /etc/nginx/conf.d/default.conf
  sudo nginx -s reload

動作確認
~~~~

1回目 ``nginx3.jwt`` 、 2回目 ``nginx1.jwt`` で続けて接続します

.. code-block:: cmdin

  curl -s localhost/auth -H "Authorization: Bearer `cat ~/f5j-nginx-plus-lab2-conf/jwt/nginx3.jwt`" | jq .request.headers

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  [
    [
      "Host",
      "slow_group"
    ],
    [
      "Connection",
      "close"
    ],
    [
      "User-Agent",
      "curl/7.68.0"
    ],
    [
      "Accept",
      "*/*"
    ],
    [
      "Authorization",
      "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6IjAwMDEifQ.eyJpc3MiOiJNeSBJRFAiLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoibmdpbngtcGx1cyIsInNjb3BlIjoicHJvZmlsZSBlbWFpbCBncm91cDIiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsIm5hbWUiOiJuZ2lueDMgdXNlciIsInByZWZlcnJlZF91c2VybmFtZSI6Im5naW54My11c2VyIiwiZ2l2ZW5fbmFtZSI6Im5naW54MyIsImZhbWlseV9uYW1lIjoidXNlciIsImVtYWlsIjoibmdpbngzQGV4YW1wbGUuY29tIn0.CGa2fDJFiTJwlNgqW6IdCENu_Re0gkPTaww-glCHckM"
    ]
  ]


.. code-block:: cmdin

  curl -s localhost/auth -H "Authorization: Bearer `cat ~/f5j-nginx-plus-lab2-conf/jwt/nginx1.jwt`"

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 2,4

  <html>
  <head><title>503 Service Temporarily Unavailable</title></head>
  <body>
  <center><h1>503 Service Temporarily Unavailable</h1></center>
  <hr><center>nginx/1.21.6</center>
  </body>
  </html>

| 1回目 ``nginx3.jwt`` でアクセスした場合、正しく応答が確認できました
| 2回目 ``nginx1.jwt`` でアクセスした場合、 ``503 Service Temporarily Unavailable`` が応答されています

ログを確認します

.. code-block:: cmdin

  tail -2 /var/log/nginx/access_jwt.log

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  127.0.0.1 - - [27/Sep/2022:09:38:00 +0900] "GET /auth HTTP/1.1" 200 948 "-" "curl/7.68.0" profile email group2 nginx3 user nginx3@example.com nginx-plus slow_group
  127.0.0.1 - - [27/Sep/2022:09:38:07 +0900] "GET /auth HTTP/1.1" 503 197 "-" "curl/7.68.0" profile email nginx1 user nginx1@example.com nginx-plus default_group

- JWTを利用した接続ログを確認すると、Curlコマンドの接続結果と同様となっています。
- 1回目 ``nginx3`` を含むリクエストのHTTPレスポンスコードが ``200`` 、2回目 ``nginx1`` を含むリクエストのHTTPレスポンスコードが ``503`` であることがわかります。
- 1回目の接続は、$jwt_claim_scope ``profile email group2`` 、 $jwt_claim_sub が ``nginx-plus`` 、 $jwt_upstream が ``slow_group`` となっています
- 2回目の接続は、$jwt_claim_scope ``profile email`` 、$jwt_claim_sub が ``nginx-plus`` 、 $jwt_upstream が ``default_group`` となっています
- 1回目の接続では、$jwt_upstream に ``group2`` が含まれているため、適切に $jwt_upstream が ``slow_group`` となっています
- 2回目の接続では、$jwt_upstream に ``group2`` が含まれていないため、適切に $jwt_upstream が ``default_group`` となっています
- 1回目と2回目の接続の、 $jwt_claim_sub は双方 ``nginx-plus`` となっています。この値がRateLimitのKeyとなっているため、連続2回の通信で2回めの ``nginx1`` がRateLimitで拒否されています

Errorログを確認します

.. code-block:: cmdin

  grep 1rpm_per_client /var/log/nginx/error.log

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
 
  2022/09/27 09:38:07 [error] 1845#1845: *17 limiting requests, excess: 0.886 by zone "1rpm_per_client", client: 127.0.0.1, server: , request: "GET /auth HTTP/1.1", host: "localhost"

2回目の接続が ``1rpm_per_client`` で Request Limit のルールに該当したことがわかります


4. OIDCによる通信制御
====

OIDCのRPとしてNGINXを動作させる方法を説明します

.. image:: ./media/nginx-oidc-slide.jpg
   :width: 500

- Keycloakの起動

Keycloakを起動するため、 ``docker_host`` へログインします

Windows Jump Hostへログインいただくと、SSHClientのショートカットがありますので、
そちらの ``docker_host`` をクリックし、接続してください。

   - .. image:: ../module01/media/putty_icon.jpg
      :width: 50

   - .. image:: ../module01/media/putty_menu.jpg
      :width: 200


以下コマンドを実行し、Keycloakを読み込みます。

.. code-block:: cmdin

  # cd ~/
  # git clone https://github.com/BeF5/f5j-nginx-plus-lab2-conf
  cd ~/f5j-nginx-plus-lab2-conf/docker-compose/keycloak
  docker-compose -f keycloak-docker-compose.yaml up -d

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  Starting keycloak_keycloak_1 ... done

Keycloakの設定
----

Identity Provider として利用する Keycloakの設定を行います


| ブラウザからKeycloakにアクセスし、設定を行います。
| Chromeを開き、 `http://10.1.1.5:8081/ <http://10.1.1.5:8081/>`__ へアクセスしてください。

   .. image:: ./media/keycloak_top.jpg
      :width: 500

**Administration Console** を開きます。ログイン画面が表示されますので以下の情報でログインしてください。

* ログイン情報
=========== ============
**usename** **password**  
=========== ============
admin       admin
=========== ============

  .. image:: ./media/keycloak_login.jpg
     :width: 500

左メニューより **Clients** を開き、 **Create** から新規作成を行います。

  .. image:: ./media/keycloak_clients.jpg
     :width: 500

Client ID: ``nginx-plus`` を指定し、 **Save** します。

  .. image:: ./media/keycloak_clients_new.jpg
     :width: 500

SettingsタブのAccess Type: ``confidential`` を選択し、Valid Redirect URIs: `http://10.1.1.7:80/_codexch <http://10.1.1.7:80/_codexch>`__ を入力し、 **Save** します。

  .. image:: ./media/keycloak_clients_setting.jpg
     :width: 500

Credentialsタブを開きます。後ほどSecretの値を利用しますので表示されている文字列を記録しておきます。

  .. image:: ./media/keycloak_clients_secret.jpg
     :width: 500

Rolesタブを開き、 **Add Role** から追加を行います。

  .. image:: ./media/keycloak_clients_role.jpg
     :width: 500

Role Name: ``nginx-keycloak-role`` を指定し、 **Save** します。

  .. image:: ./media/keycloak_clients_role2.jpg
     :width: 500

左メニュー **Users** を開き、 **Add user** からユーザの新規作成を行います。

  .. image:: ./media/keycloak_clients_users.jpg
     :width: 500

Username: ``nginx-user`` を指定し、 **Save** します。

  .. image:: ./media/keycloak_clients_users_new.jpg
     :width: 500

Credentialsタブを開き、Password: ``test`` を入力、Temporary: ``Off`` を選択し、nginx-userのパスワードを設定します。

  .. image:: ./media/keycloak_clients_users_pass.jpg
     :width: 500

Role Mappingsタブを開き、Client Roles: ``nginx-plus`` を選択し、Available Rolesに表示される ``nginx-keycloak-role`` を選択し、 **Add selected** でRoleをAssignします。

  .. image:: ./media/keycloak_clients_users_role_mapping.jpg
     :width: 500

これでKeycloakの準備は完了しました。


設定
----

NGINXの設定を行います

NGINX で OIDCの制御を行うため、NJSモジュールを利用します。
以下コマンドでNJSモジュールをインストールします。

.. code-block:: cmdin

  sudo apt install nginx-plus-module-njs

NJSモジュールがインストールされたことを確認します

.. code-block:: cmdin

  dpkg -l  | grep nginx-plus-module-njs

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  ii  nginx-plus-module-njs              27+0.7.7-1~focal                      amd64        NGINX Plus njs dynamic modules

必要となるファイルを取得します。

.. code-block:: cmdin

  cd ~/
  git clone https://github.com/nginxinc/nginx-openid-connect.git

取得したGitリポジトリに、OIDCに必要となる情報を取得し、ファイルを生成するスクリプトが保存されています。以下コマンドで、Keycloakから必要となる情報を取得します

.. code-block:: cmdin

  cd ~/nginx-openid-connect
  ./configure.sh http://10.1.1.5:8081/auth/realms/master/.well-known/openid-configuration

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  configure.sh: NOTICE: Downloading ./idp_jwk.json
  configure.sh: NOTICE: Configuring ./frontend.conf ... no change
  configure.sh: NOTICE: Configuring ./openid_connect_configuration.conf
  configure.sh: NOTICE:  - $oidc_authz_endpoint ... ok
  configure.sh: NOTICE:  - $oidc_token_endpoint ... ok
  configure.sh: NOTICE:  - $oidc_jwt_keyfile ... ok
  configure.sh: NOTICE:  - $oidc_hmac_key ... ok
  configure.sh: NOTICE:  - $oidc_pkce_enable ... ok
  configure.sh: NOTICE: Success - test configuration with 'nginx -t'

生成されるコンフィグの配置イメージ、及び参照は以下のようになります。
このラボでは ``frontend.conf`` は別途予め修正した別コンフィグをコピーし ``default.conf`` として配置します

.. image:: ./media/nginx-oidc-files-slide.jpg
   :width: 500

NJSモジュールを有効にするため ``load_modules`` ディレクティブを ``nginx.conf`` に追加します

.. code-block:: cmdin

  sudo mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-bak
  cat ~/f5j-nginx-plus-lab2-conf/lab/oidc-loadmodule.conf /etc/nginx/nginx.conf-bak > ~/nginx.conf
  sudo cp ~/nginx.conf /etc/nginx/nginx.conf

追加した結果を確認します。正しくNJSモジュールを読み込む設定が記述されていることを確認します

.. code-block:: cmdin

  head -3 /etc/nginx/nginx.conf

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  # for NJS
  load_module modules/ngx_http_js_module.so;
  load_module modules/ngx_stream_js_module.so;

設定ファイルを一部変更します。 ``<Client Secret>`` の情報は先程Keycloakの設定画面で確認した文字列を入力してコマンドを実行してください

.. code-block:: cmdin

  sed -i -e 's/my-client-id/nginx-plus/g' ~/nginx-openid-connect/openid_connect_configuration.conf
  sed -i -e 's/my-client-secret/<Client Secret>/g' ~/nginx-openid-connect/openid_connect_configuration.conf


設定内容を確認します

.. code-block:: cmdin

  cat ~/f5j-nginx-plus-lab2-conf/lab/oidc-front.conf

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 4,14,20-21,24,27

  # This is the backend application we are protecting with OpenID Connect
  upstream my_backend {
      zone my_backend 64k;
      server backend1:81;
  }
  
  # Custom log format to include the 'sub' claim in the REMOTE_USER field
  log_format main_jwt '$remote_addr - $jwt_claim_sub [$time_local] "$request" $status '
                      '$body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"';
  
  # The frontend server - reverse proxy with OpenID Connect authentication
  #
  server {
      include conf.d/openid_connect.server_conf; # Authorization code flow and Relying Party processing
      error_log /var/log/nginx/error.log debug;  # Reduce severity level as required
  
      listen 80; # Use SSL/TLS in production
  
      location / {
          proxy_hide_header "Content-Type";
          add_header 'Content-Type' 'text/html';
  
          # This site is protected with OpenID Connect
          auth_jwt "" token=$session_jwt;
          error_page 401 = @do_oidc_flow;
  
          auth_jwt_key_file $oidc_jwt_keyfile; # Enable when using filename
          #auth_jwt_key_request /_jwks_uri; # Enable when using URL
  
          # Successfully authenticated users are proxied to the backend,
          # with 'sub' claim passed as HTTP header
          proxy_set_header username $jwt_claim_sub;
          proxy_pass http://my_backend; # The backend site/app
  
          access_log /var/log/nginx/access.log main_jwt;
      }
  }

- 4行目で、転送先サーバを指定します
- ラボ環境で、応答データの ``Content-Type`` を ``text/html`` とするため、20-21行目のように設定します
- 14行目で、OIDCに必要となる各種Pathを指定した別の設定ファイルを読み込みます
- 24,27行目で、OIDCのフローに従って正しくJWT Tokenを取得したクライアントのアクセスを評価します

必要となるファイルをコピーします。 OIDCのGitHub上にサンプルとなるfrontend.confがありますが、代わりに予め作成した別のコンフィグをコピーします。
設定を反映します

.. code-block:: cmdin

  sudo cp ~/nginx-openid-connect/idp_jwk.json /etc/nginx/conf.d/
  sudo cp ~/nginx-openid-connect/openid_connect_configuration.conf /etc/nginx/conf.d/
  sudo cp ~/nginx-openid-connect/openid_connect.server_conf /etc/nginx/conf.d/
  sudo cp ~/nginx-openid-connect/openid_connect.js /etc/nginx/conf.d/
  sudo cp ~/f5j-nginx-plus-lab2-conf/lab/oidc-front.conf /etc/nginx/conf.d/default.conf
  sudo nginx -s reload

動作確認
----

Chromeブラウザを開き、 ``Secret Tab (New Incognito Window)`` を開いてください。

  .. image:: ./media/chrome_secret_tab.jpg
     :width: 500

`http://10.1.1.7 <http://10.1.1.7>`__ へアクセスしてください。

  .. image:: ./media/chrome_tcp80.jpg
     :width: 500

Keycloakのログイン画面が表示されます。先程設定を行った ``nginx-user`` でログインしてください。

* ログイン情報
=========== ============
**usename** **password**  
=========== ============
nginx-user  test
=========== ============

  .. image:: ./media/chrome_tcp80_keycloak_login.jpg
     :width: 500

ログインが正常に行われた場合、Webアプリケーションの結果をブラウザで確認いただけます。

  .. image:: ./media/chrome_tcp80_logined.jpg
     :width: 500

不要ファイル削除
----

その他項目で不要となるファイルを削除します

.. code-block:: cmdin

  sudo mv /etc/nginx/nginx.conf-bak /etc/nginx/nginx.conf 
  sudo rm /etc/nginx/conf.d/openid_connect_configuration.conf

その他設定に関する補足
----

IDaaSサービスと連携するなどインターネットを経由してOIDCを利用する場合、SSL/TLSの利用が必要となります。
以下の設定を参考に、適切な認証局に署名された証明書を利用してください

- IDaaSサービスで指定するリダイレクトURLとして ``https://<FQDN>/_codexch`` を指定してください

- 以下設定を参考に、 ``443 ssl`` 、 適切な証明書を ``ssl_certificate / key`` で指定してください

.. code-block:: bash
  :caption: 設定サンプル
  :linenos:
  :emphasize-lines: 4,14,20-21,24,27

  server {

      listen 443 ssl;

      server_name <FQDN>;
      ssl_certificate /etc/letsencrypt/live/<FQDN>/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/<FQDN>/privkey.pem;

  **省略**

