補足情報
####

Tips1. 
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

  vi ~/openssl.cnf

.. code-block:: bash
  :caption: 実行結果サンプル
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
