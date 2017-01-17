# Nginx Role for Ansible

[![Build Status](https://travis-ci.org/petemcw/ansible-role-nginx.svg?branch=master)](https://travis-ci.org/petemcw/ansible-role-nginx)

This role installs [NGINX](http://nginx.org/) which is a free, open-source, high-performance HTTP server and reverse proxy, as well as an IMAP/POP3 proxy server. NGINX is known for its high performance, stability, rich feature set, simple configuration, and low resource consumption.

## Requirements

This role requires [Ansible](http://www.ansibleworks.com/) version 2.0 or higher and the Debian/Ubuntu/CentOS/RedHat platforms.

## Role Variables

The variables that can be passed to this role and a brief description about
them are as follows (additional variables are available in the source):

```yaml

# For Ubuntu only, specifies installing from PPA and version
nginx_ppa_use: false
nginx_ppa_version: "stable"

# Remove unnecessary files
nginx_remove_apache: true
nginx_remove_default_files: true

nginx_error_log: "/var/log/nginx/error.log warn"
nginx_access_log: "/var/log/nginx/access.log main buffer=16k"

nginx_worker_processes: "{{ ansible_processor_vcpus | default(ansible_processor_count) }}"
# Sets the maximum number of simultaneous connections that can be opened by a
# worker process.
nginx_worker_connections: "1024"
nginx_multi_accept: "off"

# The server dictates the choice of cipher suites.
nginx_ssl_prefer_server_ciphers: "on"

# Use only Perfect Forward Secrecy Ciphers. Fallback on non ECDH
# for crufty clients.
nginx_ssl_ciphers: "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH"

# No SSL2 support. Legacy support of SSLv3.
nginx_ssl_protocols: "TLSv1 TLSv1.1 TLSv1.2"

# Pregenerated Diffie-Hellman parameters.
nginx_ssl_dhparam: "dh2048.pem"

# Curve to use for ECDH.
nginx_ssl_ecdh_curve: "secp384r1"

# Enable OCSP stapling. A better way to revocate server certificates.
nginx_ssl_stapling: "on"

# Enable SSL cache
nginx_ssl_cache_use: true
nginx_ssl_session_cache: "shared:SSL:20m"
nginx_ssl_session_timeout: "1d"

# Basic Auth users & passwords
nginx_htpasswd_users: []

# Bad extensions
nginx_drop_extensions:
  - 'aspx'
  - 'asp'
  - 'jsp'
  - 'cgi'

# Adds the specified charset to the “Content-Type” response header field.
nginx_utf8_use: true

# Enables GZIP compression
nginx_gzip_use: true
nginx_gzip_http_version: '1.1'
# Enables gzipping of responses for the specified MIME types.
nginx_gzip_types:
  - 'text/plain'
  - 'text/css'
  - 'application/javascript'
  - 'application/x-javascript'
  - 'text/javascript'
  - 'text/js'
  - 'application/json'
  - 'text/comma-separated-values'
  - 'text/xml'
  - 'application/xml'
  - 'application/xml+rss'
  - 'application/atom+xml'
  - 'image/svg+xml'
nginx_gzip_disable: 'msie6'

# Example extra gzip options, printed inside the main server http config:
nginx_extra_gzip_options: |
  gzip_buffers     16 8k;
  gzip_comp_level  6;
  gzip_min_length  1024;
  gzip_proxied     any;
  gzip_static      on;
  gzip_vary        on;

# Example extra main options, used within the main Nginx's context:
nginx_extra_conf_options: |
  env VARIABLE;
  include /etc/nginx/main.d/*.conf;

# Example extra http options, printed inside the main server http config:
nginx_extra_http_options: |
  client_body_buffer_size '128k';
  client_max_body_size '75m';
  keepalive_requests 100;
  keepalive_timeout 65;
  merge_slashes 'on';
  sendfile 'on';
  server_name_in_redirect 'off';
  server_names_hash_bucket_size 128;
  server_tokens 'off';
  tcp_nodelay 'on';
  tcp_nopush 'on';
  types_hash_max_size 2048;
  underscores_in_headers 'on';

# Virtual Hosts configuration
nginx_vhosts: []

# Example upstream below
nginx_upstreams:
  - name: myapp1
    strategy: "ip_hash" # "least_conn", etc.
    keepalive: 16 # optional
    servers: {
      "srv1.example.com",
      "srv2.example.com weight=3",
      "srv3.example.com"
    }
```

### Basic Auth Configuration

```yaml
# The usernames for protecting a virtual host
nginx_htpasswd_users:
  - user:
      name: 'admin_user1'
      pass: 'random-password'
  - user:
      name: 'admin_user2'
      pass: 'random-password'
```

### Virtual Host Structure

```yaml
nginx_vhosts:
  - vhost:
    server_name: 'www.example.com'  # Used to determine which server block is used for a given request
    root: '/var/www/example.com/production/current/'
    index: 'index.php'
    listen: 80
    listen_secure: 443
    is_staging: false               # Enables error reporting in some cases
    is_protected: false             # Enables Basic Auth protection
    is_secure: true                 # Enables SSL
    redirect_uris:                  # Used to generate redirection blocks for alternate domains
      - 'example.net'
      - 'www.example.net'
    rewrites:                       # Used to create URL redirects
      - origin: 'www.example.com/old-page.html'
        destination: 'www.example.com/contact-us'
  - vhost:
    server_name: 'example.io'
    is_staging: true
    is_protected: true
    is_secure: false
    redirect_uris:
      - 'www.example.io'
    # By specifying locationX entries, you can create arbitrary blocks
    location1: {name: /foo/, try_files: "$uri $uri/ /index.html"}
    location2: {name: /bar/, try_files: "$uri $uri/ /index.html"}
```

## Examples

1) Install Nginx with HTTP directives of choice, but with no virtual host configured:

```yaml
    ---
    # This playbook installs Nginx

    - name: Install Nginx to all nodes
      hosts: all
      roles:
        - { role: nginx,
            nginx_access_log: "/var/log/nginx/access.log",
            nginx_vhosts: []
          }
```

*Note: Please make sure the HTTP directives passed are valid, as this role won't check for the validity of the directives. See the [nginx documentation](http://nginx.org/en/docs/) for details.*

2) Install Nginx and configure a virtual host:

```yaml
    ---
    # This playbook installs Nginx

    - name: Install Nginx to all nodes
      hosts: all
      roles:
        - { role: nginx,
            nginx_gzip_use: false,
            nginx_extra_http_options: "keepalive_timeout 15",
            nginx_upstreams:
              - name: phpfpm
                servers: { 127.0.0.1:9000 }
            nginx_vhosts:
              - vhost:
                hostname: 'augustash.com'
                server_name: 'www.augustash.com'
                root: '/var/www/augustash.com/production/current/'
                index: 'index.php'
                is_staging: false
                is_protected: false
                is_secure: true
                listen: 80
                listen_secure: 443
                redirect_uris:
                  - 'augustash.com'
                rewrites:
                  - origin: 'www.augustash.org'
                    destination: 'www.augustash.com/non-profits'
                location1: 
                  - name: /foo/
                    try_files: "$uri $uri/ /index.html"
                location2: 
                  - name: /bar/
                    try_files: "$uri $uri/ /index.html"
          }
```

## Dependencies

None.

## License

MIT.
