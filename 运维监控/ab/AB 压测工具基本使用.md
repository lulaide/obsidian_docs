---
source: "https://juejin.cn/post/7464246752457113627#heading-0"
tags:
  - "ab"
---
AB（ApacheBench）是一款轻量级、易用且功能强大的 HTTP 服务器性能测试工具。它能够模拟多个并发用户对服务器发起请求，并统计服务器的响应时间、吞吐量等性能指标，帮助开发者评估服务器性能、发现性能瓶颈。本文将深入探讨 AB 压测工具，涵盖使用场景、语法、案例、最佳实践以及高级技巧，助您从入门到精通。

### 一、AB 压测工具使用场景

AB 压测工具适用于各种需要进行 HTTP 服务器性能测试的场景，例如：

- Web 服务器性能评估: 测试 Web 服务器在不同并发用户数下的响应时间、吞吐量等性能指标，评估服务器性能表现。
- API 接口性能测试: 测试 API 接口在不同并发请求数下的响应时间、错误率等性能指标，评估接口性能表现。
- 负载测试: 模拟高并发场景，测试服务器在高负载情况下的稳定性和性能表现，发现潜在的性能瓶颈。
- 性能优化验证: 在进行服务器性能优化后，使用 AB 压测工具验证优化效果，确保优化措施有效。

### 二、AB 压测工具语法

AB 压测工具的基本语法如下：

ab \[options\] \[http\[s\]://\]hostname\[:port\]/path

其中， `options` 是可选参数，用于指定测试的具体细节，例如并发用户数、请求总数等。 `hostname` 是服务器的域名或 IP 地址， `port` 是服务器的端口号（默认为 80）， `path` 是请求的路径。

常用选项解析：

### 三、AB 压测工具案例

案例 1：测试 Web 服务器性能

```bash
ab -n 1000 -c 100 http://www.example.com/
```

该命令表示模拟 100 个并发用户，总共发送 1000 个请求到 `http://www.example.com/` ，并统计服务器的响应时间、吞吐量等性能指标。

案例 2：测试 API 接口性能

```bash
ab -n 1000 -c 100 -T "application/json" -p data.json http://api.example.com/v1/users
```

该命令表示模拟 100 个并发用户，总共发送 1000 个 POST 请求到 `http://api.example.com/v1/users` ，请求体为 `data.json` 文件中的 JSON 数据，并统计 API 接口的响应时间、错误率等性能指标。

### 四、一个案例

ab -n 1000 -c 100 http://www.example.com/

```text
% ab -n 1000 -c 100 http://www.example.com/
% This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
% Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
% Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking www.example.com (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests
```
- `ab -n 1000 -c 100 http://www.example.com/` ：这是 AB 压测命令，表示模拟 100 个并发用户（ `-c 100` ），总共发送 1000 个请求（ `-n 1000` ）到 `http://www.example.com/` 。
- `Benchmarking www.example.com (be patient)` ：开始对目标服务器进行性能测试。
- `Completed X requests` ：实时显示已完成的请求数。

```text
Server Software:        
Server Hostname:        www.example.com
Server Port:            80
```
- `Server Software` ：服务器的软件信息（例如 Apache、Nginx）。如果未识别，则显示为空。
- `Server Hostname` ：测试的目标服务器主机名。
- `Server Port` ：测试的目标服务器端口号（默认为 80）\\

```text
Document Path:          /
Document Length:        1256 bytes
```
- `Document Path` ：测试的请求路径（例如 `/` 表示根路径）。
- `Document Length` ：服务器返回的响应内容长度（单位为字节）。

```text
Concurrency Level:      100
Time taken for tests:   40.719 seconds
Complete requests:      1000
Failed requests:        3
  (Connect: 0, Receive: 0, Length: 3, Exceptions: 0)
Non-2xx responses:      3
Total transferred:      1519488 bytes
HTML transferred:       1254430 bytes
```
- `Concurrency Level` ：并发用户数（ `-c 100` 表示 100 个并发用户）。
- `Time taken for tests` ：完成所有请求的总时间（40.719 秒）。
- `Complete requests` ：成功完成的请求数（1000 个）。
- `Failed requests` ：失败的请求数（3 个）。
- `Connect` ：连接失败数（0 个）。
- `Receive` ：接收数据失败数（0 个）。
- `Length` ：响应内容长度不一致的失败数（3 个）。
- `Exceptions` ：其他异常失败数（0 个）。
- `Non-2xx responses` ：非 2xx 状态码的响应数（3 个）。
- `Total transferred` ：所有请求的总数据传输量（1519488 字节）。
- `HTML transferred` ：所有请求的 HTML 内容总传输量（1254430 字节）。

```text
Requests per second:    24.56 [#/sec] (mean)
Time per request:       4071.900 [ms] (mean)
Time per request:       40.719 [ms] (mean, across all concurrent requests)
Transfer rate:          36.44 [Kbytes/sec] received
```

说明：

- `Requests per second` ：每秒处理的请求数（24.56 个/秒），即服务器的吞吐量。
- `Time per request` ：
- `4071.900 [ms] (mean)` ：每个请求的平均响应时间（从客户端角度看）。
- `40.719 [ms] (mean, across all concurrent requests)` ：每个请求的平均响应时间（从服务器角度看）。
- `Transfer rate` ：数据传输速率（36.44 KB/秒），表示每秒从服务器接收的数据量。

```text
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0  208 180.8    182    3180
Processing:   166  400 2087.4    184   34717
Waiting:      166  345 2025.1    183   35062
Total:        334  607 2093.2    367   34717
```
- `Connection Times` ：连接时间的统计信息（单位为毫秒）。
- `Connect` ：建立连接的时间。
- `min` ：最小连接时间（0 毫秒）。
	- `mean` ：平均连接时间（208 毫秒）。
	- `[+/-sd]` ：连接时间的标准差（180.8 毫秒）。
	- `median` ：连接时间的中位数（182 毫秒）。
	- `max` ：最大连接时间（3180 毫秒）。
- `Processing` ：服务器处理请求的时间。
- `Waiting` ：客户端等待服务器响应的时间。
- `Total` ：从建立连接到接收完数据的总时间。
```text
Percentage of the requests served within a certain time (ms)
  50%    367
  66%    372
  75%    377
  80%    380
  90%    550
  95%   1128
  98%   2087
  99%   3590
```

说明：

- `Percentage of the requests served within a certain time (ms)` ：请求响应时间的分布情况。
- `50%` ：50% 的请求在 367 毫秒内完成。
- `66%` ：66% 的请求在 372 毫秒内完成。
- `75%` ：75% 的请求在 377 毫秒内完成。
- `80%` ：80% 的请求在 380 毫秒内完成。
- `90%` ：90% 的请求在 550 毫秒内完成。
- `95%` ：95% 的请求在 1128 毫秒内完成。
- `98%` ：98% 的请求在 2087 毫秒内完成。
- `99%` ：99% 的请求在 3590 毫秒内完成。

### 五、小结

通过 AB 压测工具的输出，您可以全面了解服务器的性能表现，包括：

- 吞吐量：每秒处理的请求数（Requests per second）。
- 响应时间：每个请求的平均响应时间（Time per request）。
- 失败率：失败请求的比例（Failed requests）。
- 数据传输：总数据传输量和速率（Total transferred 和 Transfer rate）。
- 响应时间分布：不同百分比的请求响应时间（Percentage of the requests served）。

这些数据可以帮助您评估服务器的性能瓶颈，优化系统配置，提升用户体验

 