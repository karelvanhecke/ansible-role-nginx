# ansible-role-nginx
NGINX role for Centos 8

Example of how to configure the role. Complete list of configurable variables will be published later. (For now you can look at the Jinja2 templates)
```
common_configs:
  - name: ssl
    ssl_certificate: '<path to cert>'
    ssl_certificate_key: '<path to cert key>'
    ssl_trusted_certificate: '<path to cert chain>'
    ssl_protocols: TLSv1.2 TLSv1.3
    ssl_ciphers: ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-CCM:ECDHE-ECDSA-AES256-CCM
    ssl_session_timeout: 1d
    ssl_session_cache: 'shared:SSL:10m'
    ssl_session_tickets: no
    headers:
      - name: Strict-Transport-Security
        value: max-age=63072000; includeSubDomains; preload
        always: yes
    ssl_stapling: yes
    ssl_stapling_verify: yes

serverblocks:
  - name: http_to_https_redirect
    server_name: '_'
    locations:
      - path: /
        return: '301 https://$host$request_uri'

  - name: 00-www.example.org
    server_name: www.example.org
    ssl: yes
    includes:
      - ssl
    headers:
      - name: Content-Type
        value: text/plain
    locations:
      - path: /
        return: '200 "hello"'

  - name: 01-example.org
    server_name: example.org
    ssl: yes
    includes:
      - ssl
    locations:
      - path: /
        return: '301 https://www.example.org$request_uri'

  - name: application.example.org
    server_name: application.example.org
    ssl: yes
    includes:
      - ssl
    locations:
      - path: /
        backend: 'http://backend.example.org'
        proxy_buffering: no
        proxy_http_version: 1.1
        proxy_headers:
          - name: X-Forwarded-For
            value: $proxy_add_x_forwarded_for
          - name: Host
            value: $host
          - name: X-Forwarded-Proto
            value: $scheme
          - name: X-Real-IP
            value: $remote_addr
          - name: Upgrade
            value: $http_upgrade
          - name: Connection
            value: Upgrade
```