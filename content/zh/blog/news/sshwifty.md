---
title: 我的 Sshwifty 终端页面
date: 2025-05-31
---

## 嵌入式 SSH 终端

这是一个使用 Hugo 生成的页面，您可以在下面直接访问 SSH 终端：

{{< iframe_embed url="https://raop.top/sshwifty/" width="90%" height="700px" max-width="1000px" title="Sshwifty Web SSH Terminal" >}}

---

如果您想嵌入一个 YouTube 视频，您可以这样做：

{{< iframe_embed url="https://www.youtube.com/embed/your-video-id" width="80%" height="450px" title="YouTube Video Example" >}}

---

根据 sshwifty 的文档 [Can I serve Sshwifty under a subpath such as https://my.domain/ssh?](https://github.com/nirui/sshwifty?tab=readme-ov-file#can-i-serve-sshwifty-under-a-subpath-such-as-httpsmydomainssh)，sshwifty 不能部署在一个子目录里。为了绕过
这个限制，使用了 [https://github.com/nirui/sshwifty/issues/10#issuecomment-562925134](https://github.com/nirui/sshwifty/issues/10#issuecomment-562925134) 里的黑魔法。

```
events{
    worker_connections 1024;
}

http{
    index index.html index.html;

    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }

    upstream sshwifty_backend {
        # Up stream Sshwifty backend server, change address accordingly
        server 10.220.179.110:8182;
    }

    server{
        listen 80;
        server_name localhost;

        location /sshwifty/socket {
            # Proxy to the websocket interface, change address accordingly
	    proxy_pass http://sshwifty_backend/sshwifty/socket;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
        }

        location /sshwifty/ {
            rewrite ^/sshwifty/assets/(.*) /sshwifty/assets/$1 break;
            rewrite ^/sshwifty/(.*) /$1 break;

            # Proxy to the landing page, change address accordingly
	    proxy_pass http://sshwifty_backend;
        }
    }
}
```

The configuration above will map Sshwifty to /sshwifty. If you instead want to use /sshclient however, then you need to redirect /sshwifty to /sshclient/sshwifty:

```
events{
    worker_connections 1024;
}

http{
    index index.html index.html;

    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }

    upstream sshwifty_backend {
        # Up stream Sshwifty backend server, change address accordingly
        server 10.220.179.110:8182;
    }

    server{
        listen 80;
        server_name localhost;

        location /sshwifty/socket {
            # Proxy to the websocket interface, change address accordingly
	    proxy_pass http://sshwifty_backend/sshwifty/socket;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
        }

        # Notice that you have to redirect the request from /sshwifty/* to /sshclient/sshwifty/*
        location ~ ^/sshwifty/assets/(.*) {
            return 301 /sshclient/sshwifty/assets/$1;
        }

        location /sshclient/ {
            rewrite ^/sshclient/(.*) /$1 break;

            # Proxy to the landing page, change address accordingly
	    proxy_pass http://sshwifty_backend;
        }
    }
}```