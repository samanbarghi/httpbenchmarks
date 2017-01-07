#Http benchmarks

This is a brief explanation on how I setup various webservers to compare with uThreads webserver. Note that this benchmark is testing the how many requests/second webserver can handle, thus we are not going to read data from disk, so all requests are server from the memory for all web servers.

##wrk

First, you need to install [wrk](https://github.com/wg/wrk) which is a http  benchmarking tool. All benchmarks are run with
```
 wrk -d 15s -t $T -c 200 http://server:port
```

where `$T` is the number of threads and server and port is determined from the webserver. Usually running _wrk_ with the same number of worker threads as the webserver is enough. However, if the CPU utilization of _wrk_ is 100% while the CPU utilization of webserver threads are less that 100%, the number of _wrk_ threads should be increased. The goal of the benchmark is to push the webserver to the max CPU limit.

# uThreads webserver
To run uThreads webserver, simply compile and install uThreads and do a `make test` and run the webserver from `$ROOT_DRIECTORY/bin/webserver`. You can modify the number of threads (kernel threads) by passing it as the first argument.
```
./bin/webserver $T
```
where `$T` is the number of threads. Note, that the number of threads here always mean _number of worker threads + 1_, which means if you pass 4, the number of worker threads will be 3 and there will be 1 poller thread. For small number of threads (less than 8) it is safe to ignore the poller thread and if you want to test the system with 3 threads e.g., pass 4 to run it with 4 worker threads. To be more fair, you can always bind the threads to fix number of cores for all the benchmarks.

# nodejs

You can find the nodejs code that I used under _nodejs_ directory, simply run `node server.js`. Since a single nodejs process is single threaded, I do not run experiments with more number of threads.

# fasthttp
You can find the fasthttp code under _fasthttp_. First, build the code `go build server.go` and then run it using
```
GOMAXPROCS=$T ./server
```
where `$T` is the number of threads.

# cpollcppsp

You can download it from [here](https://sourceforge.net/projects/cpollcppsp/), and also download _libhoard_ from [here](https://github.com/emeryberger/Hoard). _libhoard_ is a more efficient memory allocater said to improve the performance of cppsp. After compiling both, under www folder in cppsp, create a file called _hello.cppsp_ and put _Hello Wrold!_ in the file. Then, issue the following:
```
LD_PRELOAD=/path_to_libhoard/libhoard.so ./run_example -t $T
```
where `$T` determines the number of threads. You can access the server through `http://localhost:16969/hello.cppsp`. Make sure you call the URL once with your webbrowser, as cppsp loads the content of the file into cache and the very first request is slow to server. After that, the file content is being served from the cache in memory.
