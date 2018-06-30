# 开启NGINX缓存图片功能

```conf
		#缓存图片文件
		location ~ \.(jpeg|jpg|png)$ {
            #缓存时间为1天1d
			#同样可以缓存css
			expires 1d
			#expires 1h
        }
```



#nginx 的gzip压缩功能

压缩资源，通过网络发送时业所资源，就更节省宽带了

web服务器进行压缩，浏览器需要进行解压

```conf
#gzip  on;
```

