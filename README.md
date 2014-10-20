# Nginx

This role installs (Nginx)[http://nginx.org/] which is an HTTP and reverse proxy
server, as well as a mail proxy server.

## Requirements

This role requires [Ansible](http://www.ansibleworks.com/) version 1.4 or higher
and the Debian/Ubuntu platform.

## Role Variables

The variables that can be passed to this role and a brief description about
them are as follows (additional variables are available in the source):

```yaml
# Adds the specified charset to the “Content-Type” response header field.
nginx_utf8: true

# Defines a file that will store the process ID of the main process.
nginx_pid: '/var/run/nginx.pid'

# Sets the maximum number of simultaneous connections that can be opened by a
# worker process.
nginx_worker_connections: 1024

# Sets the address and port for IP, or the path for a UNIX-domain socket on
# which the server will accept requests.
nginx_listen_port: 80

# Sets the address and port for IP, or the path for a UNIX-domain socket on
# which the server will accept requests.
nginx_listen_port_secure: 443

# The directives ssl_protocols and ssl_ciphers can be used to limit connections
# to include only the strong versions and ciphers of SSL/TLS.
nginx_ssl_protocols: 'SSLv3 TLSv1'
nginx_ssl_ciphers: 'RC4:HIGH:!aNULL:!MD5'

# Enables or disables gzipping of responses.
nginx_gzip: true

# Sets the minimum HTTP version of a request required to compress a response.
nginx_gzip_http_version: '1.0'

# Enables gzipping of responses for the specified MIME types.
nginx_gzip_types:
    - 'text/plain'
    - 'text/css'
    - 'application/x-javascript'

# Disables gzipping of responses for requests with “User-Agent” header fields
# matching any of the specified regular expressions.
nginx_gzip_disable: 'MSIE [1-6].(?!.*SV1)'

# Any valid configuration parameter for ngx_http_core_module
# http://nginx.org/en/docs/http/ngx_http_core_module.html
nginx_http_params:
  access_log: '/var/log/nginx/access.log main'
  error_log:  '/var/log/nginx/error.log warn'
  keepalive_timeout: 25

# Enables caching
nginx_cache_enabled: false

# Defines the address and other parameters of an upstream server.
nginx_fastcgi_use_socket: true
nginx_fastcgi_socket: '/var/run/php5-fpm.sock'
nginx_fastcgi_listen_tcp: '127.0.0.1:9000'

# Extensions that should be served as static assets
nginx_asset_extensions:
  - 'jpe?g'
  - 'gif'
  - 'png'
  - 'ico'

# Extensions that should be dropped
nginx_drop_extensions:
  - 'aspx'
  - 'asp'
  - 'jsp'
  - 'cgi'
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
# The list of user accounts to be added to the system
nginx_vhosts:
  - server:
    platform:
      - name: 'magento'             # Currently supported: magento, drupal, wordpress, cmsms
      - name: 'wordpress'
        subdir: 'blog/'             # If defined, attempts to write correct location blocks
    magento_run_type: 'store'       # Currently supported: website, store
    magento_run_code: 'base'        # User generated website/store code
    magento_developer_mode: true    # Enable Magento development features
    server_name: 'www.example.com'  # Used to determine which server block is used for a given request
    application_name: 'example.com' # Used to create correct file system path and virtual host file
    listen_port: 80
    listen_port_secure: 443
    is_staging: false               # Enables error reporting in some cases
    is_protected: false             # Enables Basic Auth protection
    is_ssl: true                    # Enables SSL server block
    redirect_urls:                  # Used to generate redirection blocks for alternate domains
      - 'example.net'
      - 'www.example.net'
    rewrites:                       # Used to create URL redirects
      - origin: 'www.example.com/old-page.html'
        destination: 'www.example.com/contact-us'
  - server:
    platform:
      - name: 'drupal'
    server_name: 'example.io'
    application_name: 'example.io'
    is_staging: true
    is_protected: true
    is_ssl: false
    redirect_urls:
      - 'www.example.io'
    # By specifying locationX entries, you can create arbitrary blocks
    location1: {name: /foo/, try_files: "$uri $uri/ /index.html"}
    location2: {name: /bar/, try_files: "$uri $uri/ /index.html"}
```

## Examples

1) Install Nginx with HTTP directives of choice, but with no vhosts configured:

```yaml
    ---
    # This playbook installs Nginx

    - name: Apply Nginx to all web nodes
      hosts: web_nodes
      roles:
        - { role: nginx,
            access_log: "/var/log/nginx/access.log",
            nginx_vhosts: none
          }
```

*Note: Please make sure the HTTP directives passed are valid, as this role won't check for the validity of the directives. See the [nginx documentation](http://nginx.org/en/docs/) for details.*

2) Install Nginx and configure a vhost:

```yaml
    ---
    # This playbook installs Nginx

    - name: Apply Nginx to all web nodes
      hosts: web_nodes
      roles:
        - { role: nginx,
            nginx_http_params: { keepalive_timeout: 15 },
            nginx_vhosts:
                - server:
                  platform:
                    - name: 'magento'
                  magento_run_type: 'store'
                  magento_run_code: 'base'
                  server_name: 'www.augustash.com'
                  application_name: 'augustash.com'
                  listen_port: 80
                  is_staging: false
                  is_protected: false
                  is_ssl: false
                  redirect_urls:
                    - 'augustash.com'
                    - 'augustash.net'
                    - 'www.augustash.net'
                  rewrites:
                    - origin: 'www.augustash.com/old-page.html'
                      destination: 'www.augustash.com/contact-us'
                  location1: {name: /foo/, try_files: "$uri $uri/ /index.html"}
                  location2: {name: /bar/, try_files: "$uri $uri/ /index.html"}
          }
```

## Dependencies

The following packages may be required for Debian derivatives:

- `aptitude`
- `python-apt`
- `python-passlib`

## License

MIT.
