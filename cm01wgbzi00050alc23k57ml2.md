---
title: "Building a Lightweight HTTP Client in Bash: `exec 3<>/dev/tcp/"$host"/80`"
datePublished: Tue Aug 20 2024 04:03:06 GMT+0000 (Coordinated Universal Time)
cuid: cm01wgbzi00050alc23k57ml2
slug: building-a-lightweight-http-client-in-bash-exec-3devtcphost80
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1724126535066/038fea24-3962-4c2d-bb5e-d4abfa6698d0.png
tags: http, backend, bash, tcp

---


In the world of web development and system administration, HTTP clients are essential tools for interacting with web services. While many developers rely on popular utilities like `curl` or `wget`, it's both educational and sometimes necessary to create a basic HTTP client using nothing but Bash. This approach not only deepens our understanding of the HTTP protocol but also proves useful in environments where standard tools may not be available.
Our Bash-based HTTP client supports the fundamental HTTP methods: `GET`, `POST`, `PUT`, `DELETE`, and `OPTIONS`. Here's a breakdown of its key components:
1. TCP Connection Handling
The script uses Bash's built-in `/dev/tcp/` pseudo-device to establish TCP connections. This feature, often overlooked, allows for raw socket programming directly in Bash.
```bash
exec 3<>/dev/tcp/"$host"/80
```
This line opens a bidirectional connection to the specified host on port `80`, using file descriptor 3.
2. Request Formatting
The script constructs HTTP requests using here-documents, ensuring proper formatting of headers and body content.
```bash
echo -e "$method $path HTTP/1.1\r
Host: $host\r
Connection: close\r
Content-Length: ${#data}\r
\r
$data" >&3
```
3. Response Handling
After sending the request, the script reads the response using the cat command:
```bash
cat <&3
```
This simple approach captures both headers and body content.
4. Method-Specific Functions
The script provides separate functions for each HTTP method, simplifying usage and improving readability.
The complete Script is on [my GitHub](https://github.com/lyluongthien/rb-http-request/blob/main/rb-req-client.sh)

```bash
#!/bin/bash

perform_request() {
    local method="$1"
    local url="$2"
    local data="$3"
    local host=$(echo "$url" | awk -F/ '{print $3}')
    local path=$(echo "$url" | awk -F/ '{print "/" $4}')

    exec 3<>/dev/tcp/"$host"/80

    echo -e "$method $path HTTP/1.1\r
Host: $host\r
Connection: close\r
Content-Length: ${#data}\r
\r
$data" >&3

    cat <&3
    exec 3>&-
}

get_request() {
    perform_request "GET" "$1" ""
}

post_request() {
    perform_request "POST" "$1" "$2"
}

put_request() {
    perform_request "PUT" "$1" "$2"
}

delete_request() {
    perform_request "DELETE" "$1" ""
}

options_request() {
    perform_request "OPTIONS" "$1" ""
}

# Usage examples
# get_request "http://example.com"
# post_request "http://example.com/api" "key1=value1&key2=value2"
# put_request "http://example.com/api/resource" "updated_data"
# delete_request "http://example.com/api/resource"
# options_request "http://example.com"
```
While this implementation is basic, it demonstrates the power and flexibility of Bash for networking tasks. It's particularly useful for quick tests, debugging, or in environments where installing additional tools is not feasible.
In our next article, we'll explore the server-side of HTTP by building a simple CRUD server in Bash, rounding out our exploration of raw HTTP handling with Bash.