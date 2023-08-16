---
title: Web - 訪問一個網站時發生什麼事?
tags:
  - Web
categories:
  - Web
---

<!-- more -->

# URL 的組成

# 訪問網站流程

1. DNS 查找 : 向 DNS(Domain Name Server) 發請求，取的網頁伺服器真實 IP 地址。
2. TCP/IP socket 將 Client 與 Web server 連接 :
   - TCP(傳輸控制協議，Transmission Control Protocol)
   - IP(網路協議，Internet Protocol)
3. 發送 HTTP 請求
4. 取得 HTTP 響應
5. 依序載入 HTML、CSS、JavaScript 與圖片，完成後即呈現網頁。

## TCP/IP

TCP :
中斷請求並在響應之前將響應數據分解成數個小塊，當 Client 收到響應時，TCP 將收到的數據重新組裝。這可以使數據傳輸更加快速。

IP :
在每個數據塊上使用 IP 地址，並透過網路傳輸數據，確保每個小塊都傳送到正確的地方。

## 請求與響應報文 request/response message

request :

```
GET /rest/v2/alpha/PT HTTP/1.1    // Start line: HTTP method + request target + HTTP version

Host: www.google.com              // HTTP request headers
User-Agent: Mozilla/5.0
Accept-Language: en-US

<BODY>                            // Request body
```

response :

```
HTTP/1.1 200 OK                   // Start line: HTTP version + status code + status message

Date: Mon, 14 Aug 2023            // HTTP response headers
Content-Type: text/html
Transfer-Encoding: chunked

<BODY>                            // Response body
```

# 參考資料
