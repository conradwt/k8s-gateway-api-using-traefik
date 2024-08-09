# K8s Gateway API Using Kong Benchmark

```zsh
oha -c 32 -n 300000 --disable-keepalive --disable-compression --insecure --ipv6 --http2 http://$PROXY_IP/echo
```

```
Summary:
  Success rate:	100.00%
  Total:	18.3526 secs
  Slowest:	0.0382 secs
  Fastest:	0.0002 secs
  Average:	0.0020 secs
  Requests/sec:	16346.4472

  Total data:	41.77 MiB
  Size/request:	146 B
  Size/sec:	2.28 MiB
```

```zsh
oha -c 64 -n 300000 --disable-keepalive --disable-compression --insecure --ipv6 --http2 http://$PROXY_IP/echo
```

```
Summary:
  Success rate:	100.00%
  Total:	12.8269 secs
  Slowest:	0.0474 secs
  Fastest:	0.0002 secs
  Average:	0.0027 secs
  Requests/sec:	23388.4167

  Total data:	41.77 MiB
  Size/request:	146 B
  Size/sec:	3.26 MiB
```

```zsh
oha -c 128 -n 300000 --disable-keepalive --disable-compression --insecure --ipv6 --http2 http://$PROXY_IP/echo
```

```
Summary:
  Success rate:	100.00%
  Total:	11.8855 secs
  Slowest:	0.0488 secs
  Fastest:	0.0002 secs
  Average:	0.0051 secs
  Requests/sec:	25240.8991

  Total data:	41.77 MiB
  Size/request:	146 B
  Size/sec:	3.51 MiB
```
