# 常用压测方法
- ab (Apache HTTP server benchmarking tool)[http://httpd.apache.org/docs/2.0/programs/ab.html]
- ab -n 100 -c 10 http://www.qiyuan.cool/admin/  # 访问100次，并发10数(-t 100 持续时间)
- ulimit -n 查看服务器可接受的并发数；ulimit -n 500 可以修改并发数为5 00但重启服务器修改失效;

- Siege (is an open source regression test and benchmark utility)[https://www.linode.com/docs/tools-reference/tools/load-testing-with-siege/]
- siege -c 100 -t 3s www.qiyuan.cool/admin/ # 并发100， 持续3秒

- wrk (is a modern HTTP benchmarking tool)[https://github.com/wg/wrk]
- wrk -c100 -d5s -t4 http://www.qiyuan.cool/admin/   # 100个链接，4个线程，持续5秒
- git clone https://github.com/wg/wrk.git -> cd wrk && make  -> cp wrk /usr/local/bin 

