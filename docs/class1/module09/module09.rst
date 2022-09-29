補足情報
####

Tips1. サンプルアプリケーションとして動作するNGINXの設定
====

以下がサンプルアプリケーションの参考設定となります。
Luaスクリプトモジュールを導入したNGINXで実行してください。

- ``/etc/nginx/conf.d/default.conf`` 

.. code-block:: bash
  :caption: Version情報
  :linenos:

  # /etc/nginx/conf.d/default.conf
  server {
      listen       80 default_server;
      server_name  localhost;
  
      #charset koi8-r;
      #access_log  /var/log/nginx/host.access.log  main;
  
      location / {
          root   /usr/share/nginx/html;
          index  index.html index.htm;
      }
  
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
  
  }
  ### this is for back to basic part2
  server {
      listen 81;
      listen 82;
      return 200 "{ \"request_uri\": \"$request_uri\",\"server_addr\":\"$server_addr\",\"server_port\":\"$server_port\"}";
  }
  server {
      listen 83;
      location / {
          content_by_lua_block {
              ngx.sleep(1)
              ngx.print("{ \"request_uri\": \""..ngx.var.request_uri.."\",\"server_addr\":\""..ngx.var.server_addr.."\",\"server_port\":\""..ngx.var.server_port.."\"}")
          }
      }
  }
  server {
      listen 84;
      location / {
          return 500 "Server Error";
      }
  }
  
  server {
      listen 443 ssl;
      ssl_certificate_key conf.d/ssl/nginx-ecc-p256.key;
      ssl_certificate conf.d/ssl/nginx-ecc-p256.pem;
      return 200 "{ \"request_uri\": \"$request_uri\",\"server_addr\":\"$server_addr\",\"server_port\":\"$server_port\"}";
  }


- ``/etc/nginx/conf.d/ssl`` に ``証明書(nginx-ecc-p256.pem)`` 、 ``鍵(nginx-ecc-p256.key)`` を配置します


Tips2. OpenSSL CA(認証局) の設定
====

ホストのVersion情報は以下の通り

.. code-block:: bash
  :caption: Version情報
  :linenos:

  $ cat /etc/*release
  DISTRIB_ID=Ubuntu
  DISTRIB_RELEASE=20.04
  DISTRIB_CODENAME=focal
  DISTRIB_DESCRIPTION="Ubuntu 20.04.3 LTS"
  NAME="Ubuntu"
  VERSION="20.04.3 LTS (Focal Fossa)"
  ID=ubuntu
  ID_LIKE=debian
  PRETTY_NAME="Ubuntu 20.04.3 LTS"
  VERSION_ID="20.04"
  HOME_URL="https://www.ubuntu.com/"
  SUPPORT_URL="https://help.ubuntu.com/"
  BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
  PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
  VERSION_CODENAME=focal
  UBUNTU_CODENAME=focal
  
  $ openssl version
  OpenSSL 1.1.1f  31 Mar 2020


``openssl.cnf`` をコピー

.. code-block:: cmdin

  # mkdir ssl
  # cd ssl
  cp /etc/ssl/openssl.cnf .

以下の内容を参考に、openssl.cnfの内容を変更

.. code-block:: cmdin

  vi openssl.cnf

.. code-block:: bash
  :caption: oepnssl.cnf 記述差分
  :linenos:

   ####################################################################
   [ CA_default ]
  
  -dir            = ./demoCA              # Where everything is kept
  +dir            = ./            # Where everything is kept
   certs          = $dir/certs            # Where the issued certs are kept
   crl_dir                = $dir/crl              # Where the issued crl are kept
   database       = $dir/index.txt        # database index file.
  @@ -50,12 +50,12 @@
                                          # several certs with same subject.
   new_certs_dir  = $dir/newcerts         # default place for new certs.
  
  -certificate    = $dir/cacert.pem       # The CA certificate
  +certificate    = $dir/CA.pem   # The CA certificate
   serial         = $dir/serial           # The current serial number
   crlnumber      = $dir/crlnumber        # the current crl number
                                          # must be commented out to leave a V1 CRL
   crl            = $dir/crl.pem          # The current CRL
  -private_key    = $dir/private/cakey.pem# The private key
  +private_key    = $dir/CA.key# The private key
  
   x509_extensions        = usr_cert              # The extensions to add to the cert
  
  @@ -169,7 +169,8 @@
   # This goes against PKIX guidelines but some CAs do it and some software
   # requires this to avoid interpreting an end user certificate as a CA.
  
  -basicConstraints=CA:FALSE
  +basicConstraints=CA:TRUE
  +#basicConstraints=CA:FALSE
  
   # Here are some examples of the usage of nsCertType. If it is omitted
   # the certificate can be used for anything *except* object signing.
  @@ -186,9 +187,13 @@
   # and for everything including object signing:
   # nsCertType = client, email, objsign
  
  +nsCertType = sslCA, emailCA, server, client
  +
   # This is typical in keyUsage for a client certificate.
   # keyUsage = nonRepudiation, digitalSignature, keyEncipherment
  
  +keyUsage = cRLSign, keyCertSign, nonRepudiation, digitalSignature, keyEncipherment


必要となるフォルダ、ファイルの作成

.. code-block:: cmdin

  mkdir newcerts
  touch index.txt
  echo 01 > serial
  echo 01 > crlnumber

Tips3. DNSコンテンツサーバのデプロイ
====

- Docker Compose の実行

各ファイルを同ディレクトリに配置し、以下コマンドを実行します

.. code-block:: cmdin

  docker-compose -f docker-compose2.yaml up -d

- ``docker-compose.yaml`` 

.. code-block:: bash
  :caption: docker-compose.yaml
  :linenos:

  version: '2'
  services:
    dns:
      restart: always
      image: strm/dnsmasq
      volumes:
        - ./dnsmasq.conf:/etc/dnsmasq.conf
        - ./hosts-dnsmasq:/etc/hosts-dnsmasq
      ports:
        - "53:53/udp"
      cap_add:
        - NET_ADMIN

- ``/etc/dnsmasq.conf``

.. code-block:: bash
  :caption: dnsmasq.conf
  :linenos:

  port=53
  no-hosts
  addn-hosts=/etc/hosts-dnsmasq
  expand-hosts
  domain=example.com
  domain-needed
  bogus-priv

- ``hosts-dnsmasq``

.. code-block:: bash
  :caption: hosts-dnsmasq
  :linenos:

  10.1.1.8 backend1 backend2 backend3 backend4
  10.1.1.5 elasticsearch security-backend1 security-backend2 security-backend3 app-backend1 app-backend2 app-backend3
  10.1.1.81 api1
  10.1.1.82 api1
  10.1.1.83 api1
  10.1.1.84 api1

Tips3. KeyCloakのデプロイ
====

- Docker Compose の実行

各ファイルを同ディレクトリに配置し、以下コマンドを実行します

.. code-block:: cmdin

  docker-compose -f docker-compose2.yaml up -d

- ``docker-compose.yaml``

.. code-block:: bash
  :caption: docker-compose.yaml
  :linenos:

  version: '3'
  services:
    keycloak:
      image: quay.io/keycloak/keycloak:15.0.2
      ports:
        - 8443:8443
        - 8081:8080
      environment:
        - KEYCLOAK_USER=admin
        - KEYCLOAK_PASSWORD=admin

