#CSCA in nginx

## 1) Configuration

```
# HTTPS server
#
server {
        listen 443;
        
	[...]

        ssl on;
        ssl_certificate /etc/ssl/ca/certs/users/server.crt;
        ssl_certificate_key /etc/ssl/ca/private/server.key;
        ssl_client_certificate /etc/ssl/ca/certs/ca.crt;
        #ssl_crl /etc/ssl/ca/private/ca.crl;
        ssl_verify_client on;

        [...]

	location ~ \.php$ {

		[...]

		fastcgi_param VERIFIED $ssl_client_verify;
		fastcgi_param DN $ssl_client_s_dn;

		[...]
	}
}
```

Explanations:

`ssl on`: This enable the ssl module on nginx

`ssl_certificate [...]`: This is the path to the server's certificate

`ssl_certificate_key [...]`: This is the path to the server's private key

`ssl_client_certificate [...]`: This is the path to the Certificate Authority to verify clients against

`ssl_verify_client on`: Use "optional" if you want non-verified clients to be able to access the server, "on" otherwise

`fastcgi_param VERIFIED $ssl_client_verify`: Add a $_SERVER['VERIFIED'] php variable which is either "SUCCESS", "FAILED:reason" or "NONE"

`fastcgi_param DN $ssl_client_s_dn`: Add a $_SERVER['DN'] php variable which represents the client's DN

> Note: You can find more variable here: http://nginx.org/en/docs/http/ngx_http_ssl_module.html

## 2) Testing your configuration

```
curl https://localhost -k --cert /etc/ssl/ca/certs/users/user001.crt --key /etc/ssl/ca/certs/users/user001.key
```

