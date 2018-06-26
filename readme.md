# A Simple Reverse Proxy tool  
[![Go Report Card](https://goreportcard.com/badge/github.com/Arryboom/reverse_proxy_go)](https://goreportcard.com/report/github.com/Arryboom/reverse_proxy_go)  
**Devloped with GoLang**  

Recently I got traffic block on music.163.com,so I need a reverse proxy to bypass it.First I tried nginx,it works good but I got a much more simple solution for non IT guys,here it is.  

1.Just download captable execution file for your OS  
[Win64](https://github.com/Arryboom/reverse_proxy_go/releases/download/1.0/reverse_proxy64.exe)  
[Win32](https://github.com/Arryboom/reverse_proxy_go/releases/download/1.0/reverse_proxy32.exe)  
[Linux64](https://github.com/Arryboom/reverse_proxy_go/releases/download/1.0/reverse_proxy64)  
[Linux32](https://github.com/Arryboom/reverse_proxy_go/releases/download/1.0/reverse_proxy32)  
2.Run it on a machine doesn't affected by filter rule(eg one of your VPS not in your company's local network),then you should got some echo like below:  
> 2018/06/26 17:/57:28 Listening on 0.0.0.0:80, forwarding to http://music.163.com:80  

3.Now open your broswer and input the ipaddress of the machine running this tool,should able to access the website you want.  

**Other params**  
eg you want to redirect google,on windows simple run 
```
reverse_proxy.exe -r "http://google.com:80"
```

>  -l string
        listen on ip:port (default "0.0.0.0:8888")

>  -r string
        reverse proxy addr (default "http://music.163.com:80")

**Advanced**  


or you can replace 
```
remote := flag.String("r", "http://music.163.com:80", "reverse proxy addr")
```
to 
```
remote := flag.String("r", "http://google.com:80", "reverse proxy addr")
```
in source code file reverseproxy.go to change default proxy addr.  
You need install golang first to compile it in this situation.  
And for cross platform compile:
**For windows**  
>In CMD console:  
 ```set GOARCH=386```  
 for 32bit  
 ```set GOARCH=amd64```  
 for 64bit  
```set GOOS=windows```    
 then   
```go build -o xxx.exe```  

**For linux**  
Replace GOOS value with keyword linux.


Source from https://www.imsxm.com/2017/12/go-active-proxy-tool.html

```
package main

import (
	"flag"
	"log"
	"net/http"
	"net/http/httputil"
	"net/url"
)

type handle struct {
	reverseProxy string
}

func (this *handle) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	remote, err := url.Parse(this.reverseProxy)
	if err != nil {
		log.Fatalln(err)
	}
	proxy := httputil.NewSingleHostReverseProxy(remote)
	r.Host = remote.Host
	proxy.ServeHTTP(w, r)
	log.Println(r.RemoteAddr + " " + r.Method + " " + r.URL.String() + " " + r.Proto + " " + r.UserAgent())
}

func main() {
	bind := flag.String("l", "0.0.0.0:80", "listen on ip:port")
	remote := flag.String("r", "http://music.163.com:80", "reverse proxy addr")
	flag.Parse()
	log.Printf("Listening on %s, forwarding to %s", *bind, *remote)
	h := &handle{reverseProxy: *remote}
	err := http.ListenAndServe(*bind, h)
	if err != nil {
		log.Fatalln("ListenAndServe: ", err)
	}
}
```