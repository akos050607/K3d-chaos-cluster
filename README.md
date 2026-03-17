# K3d-chaos-cluster

tailscale to not make k3d public
kube-proxy cause kubernetes has no public ip
accidentally this happened, while I closed my computer:
<img width="932" height="501" alt="image" src="https://github.com/user-attachments/assets/3a5bffd8-0c9d-4533-b74b-3b4c110cf5af" />

chaos: ab -n 100000 -c 100 http://100.120.50.29:30662/
tailscale ip:100.120.50.29

[Screencast from 2026-03-17 21-06-41.webm](https://github.com/user-attachments/assets/b6ef64e3-ffc7-4452-bb03-615322705d39)

log:

----------

This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 100.120.50.29 (be patient)
Completed 10000 requests
Completed 20000 requests
Completed 30000 requests
Completed 40000 requests
Completed 50000 requests
Completed 60000 requests
Completed 70000 requests
Completed 80000 requests
Completed 90000 requests
^C

Server Software:        nginx/1.29.6
Server Hostname:        100.120.50.29
Server Port:            30662

Document Path:          /
Document Length:        896 bytes

Concurrency Level:      100
Time taken for tests:   57.965 seconds
Complete requests:      99603
Failed requests:        0
Total transferred:      112451787 bytes
HTML transferred:       89244288 bytes
**Requests per second:    1718.34 [#/sec] (mean)**
Time per request:       58.196 [ms] (mean)
Time per request:       0.582 [ms] (mean, across all concurrent requests)
Transfer rate:          1894.54 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       14   29  11.7     30     101
Processing:    14   29  11.4     30     102
Waiting:       13   29  11.5     30     102
Total:         29   58  22.2     62     166

Percentage of the requests served within a certain time (ms)
  50%     62
  66%     69
  75%     73
  80%     76
  90%     87
  95%     96
  98%    109
  99%    118
 100%    166 (longest request)

----------

this was a test where it has exactly 4 replicas

one nginx can have 50m in the next setting:

sudo k3s kubectl set resources deployment mini-netflix --requests=cpu=50m --limits=cpu=100m

sudo k3s kubectl autoscale deployment mini-netflix --cpu-percent=50 --min=4 --max=20
[Screencast from 2026-03-17 21-21-18.webm](https://github.com/user-attachments/assets/ca27102f-b2cb-43f9-85e2-a28be8518dae)
log:

----------

This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 100.120.50.29 (be patient)
Completed 10000 requests
Completed 20000 requests
Completed 30000 requests
Completed 40000 requests
Completed 50000 requests
Completed 60000 requests
Completed 70000 requests
Completed 80000 requests
Completed 90000 requests
Completed 100000 requests
Finished 100000 requests


Server Software:        nginx/1.29.6
Server Hostname:        100.120.50.29
Server Port:            30662

Document Path:          /
Document Length:        896 bytes

Concurrency Level:      100
Time taken for tests:   59.113 seconds
Complete requests:      100000
Failed requests:        0
Total transferred:      112900000 bytes
HTML transferred:       89600000 bytes
Requests per second:    1691.67 [#/sec] (mean)
Time per request:       59.113 [ms] (mean)
Time per request:       0.591 [ms] (mean, across all concurrent requests)
Transfer rate:          1865.13 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       14   29  12.6     30     125
Processing:    14   30  12.2     31     125
Waiting:       14   30  12.2     30     125
Total:         29   59  23.7     62     221

Percentage of the requests served within a certain time (ms)
  50%     62
  66%     69
  75%     72
  80%     76
  90%     88
  95%    101
  98%    116
  99%    130

----------

the other pods closed after a while

in the future: more masters in the cluster
