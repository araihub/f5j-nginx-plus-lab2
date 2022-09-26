NGINX Plus 認証・認可の設定
####

1. mTLS / SSL証明書認証
====

クライアントから正しい証明書が提示されたことを検証し、接続を許可する設定です

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
  :emphasize-lines: 1-8

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
  :emphasize-lines: 1-8

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
  :emphasize-lines: 1-8

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
  :emphasize-lines: 1-8

  V       230926103540Z           01      unknown /C=JP/ST=Tokyo/O=EXAMPLE COM/CN=webapp.example.com/emailAddress=admin@example.com
  V       230926103629Z           02      unknown /C=JP/ST=Tokyo/O=EXAMPLE COM/CN=client1.example.com/emailAddress=admin@example.com
  V       230926103744Z           03      unknown /C=JP/ST=Tokyo/O=EXAMPLE COM/CN=client2.example.com/emailAddress=admin@example.com

また参考に以下の情報も確認してください

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:
  :emphasize-lines: 1-8

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

  sudo cp ~/f5j-nginx-plus-lab2-conf/lab/mtls1.conf /etc/nginx/conf.d/default.conf
  sudo nginx -s reload

動作確認
~~~~

クライアント証明書を提示せず、通信を行います

.. code-block:: cmdin

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

Webサーバナドでユーザ認証をシンプルに実施する方法として、Basic認証の動作を確認します

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

設定
----

動作確認
----

4. OIDCによる通信制御
====

設定
----

動作確認
----