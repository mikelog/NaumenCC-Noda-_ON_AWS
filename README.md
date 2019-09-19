# Naumen Contact Center, он же Noda, установка на AWS
1. Подводный камень:
  - После запуска *service ncc-installer start*, вместо ожидаемого приложения визарда на порту 8001, я получил в логи такие ошибки:
      ```
      ImportError: pycurl: libcurl link-time ssl backend (openssl) is different from compile-time ssl backend (nss)
      ```
    Если брать стандарный CentOS 7_minimal.iso, то на  нем curl идет с NSS и чуть древнее, чем на AWS, но это не принципипально. Вот выхлоп со свежепоставленного Centos7:
      ```
      curl --version
      curl 7.29.0 (x86_64-redhat-linux-gnu) libcurl/7.29.0 NSS/3.36 zlib/1.2.7 libidn/1.28 libssh2/1.4.3
      ```
    И pycurl.version:
      ```
      >>> pycurl.version
      'libcurl/7.29.0 NSS/3.36 zlib/1.2.7 libidn/1.28 libssh2/1.4.3'
      ```
    А вот, что меня ждало на AWS:
      ```
      curl --version
      curl 7.61.1 (x86_64-koji-linux-gnu) libcurl/7.61.1 OpenSSL/1.0.2k zlib/1.2.7 libidn2/2.0.4 libssh2/1.4.3 nghttp2/1.31.1
      ```
    И pycurl.version:
      ```
      >>> pycurl.version
      'PycURL/7.43.0.3 libcurl/7.61.1 OpenSSL/1.0.2k zlib/1.2.7 libidn2/2.0.4 libssh2/1.4.3 nghttp2/1.31.1'
      ```
    Но не страшны нам ни дождь, ни град, ни ветер, а тем более ./configure, make, make install. 
    Тянем исходники CURL под версиюю на инстансе AWS
      ```
      wget http://curl.haxx.se/download/curl-7.61.1.tar.gz
      ```
    Распаковываем, конфигурим(ключевой момент *'--without-ssl' '--with-nss'*), собираем, инсталим и пересобираем pycurl
      ```
      tar -xf curl-7.61.1.tar.gz
      cd curl-7.61.1/
      ./configure '--build=x86_64-koji-linux-gnu' '--host=x86_64-koji-linux-gnu' '--program-prefix=' '--disable-dependency-tracking' '--prefix=/usr' '--exec-prefix=/usr' '--bindir=/usr/bin' '--sbindir=/usr/sbin' '--sysconfdir=/etc' '--datadir=/usr/share' '--includedir=/usr/include' '--libdir=/usr/lib64' '--libexecdir=/usr/libexec' '--localstatedir=/var' '--sharedstatedir=/var/lib' '--mandir=/usr/share/man' '--infodir=/usr/share/info' '--cache-file=../config.cache' '--disable-static' '--enable-symbol-hiding' '--enable-ipv6' '--enable-ldaps' '--enable-manual' '--enable-threaded-resolver' '--with-gssapi' '--with-libidn2' '--with-libmetalink' '--with-libssh2' '--with-nghttp2' '--without-ssl' '--with-nss' '--with-ca-bundle=/etc/pki/tls/certs/ca-bundle.crt' 'build_alias=x86_64-koji-linux-gnu' 'host_alias=x86_64-koji-linux-gnu' 'CFLAGS=-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches   -m64 -mtune=generic' 'LDFLAGS=-Wl,-z,relro '
      make
      make install
      PYCURL_SSL_LIBRARY=nss
      pip install --no-cache-dir --compile --ignore-installed --install-option="--with-nss" pycurl
      ```
   И, вуаля:
      ```
      >>> pycurl.version
      'PycURL/7.43.0.3 libcurl/7.61.1 NSS/3.36 zlib/1.2.7 libssh2/1.4.3'
      ```
      
      
      
      
    
  
    
