# Nginx in CDN

## Simplest way

```
server {
  listen 80;
  server_name cdn.yourwebsite.com;
  root /var/www;

  location ~* \.(png|jpg|gif|js|css)$ {
    expires 90d;
  }
}
```

如此，使用`cdn.yourwebsite.com`域名访问站点时，如果location匹配的是.png图片等静态文件格式资源，那么便会返回一个`expires`的header：

```
# curl -I -x 127.0.0.1:80 http://cdn.yourwebsite.com/images/arch.png
HTTP/1.1 200 OK
Server: nginx/1.10.3
Date: Tue, 10 Apr 2018 07:24:35 GMT
Content-Type: image/png
Content-Length: 128919
Last-Modified: Wed, 29 Jun 2016 15:44:19 GMT
Connection: keep-alive
ETag: "5773ecd3-1f797"
Expires: Mon, 09 Jul 2018 07:24:35 GMT
Cache-Control: max-age=7776000
Accept-Ranges: bytes
```
