### Changes description
* http-filter-example is extended to be `StreamFilter` instead of `DecoderFilter`
* Filter `encodeData` method has `sleep(1);` line which emulates content processing with the filter

### Build envoy
```
bazel build //http-filter-example:envoy
```

### Run envoy
```
bazel-bin/http-filter-example/envoy -c forward-proxy.yaml
```

### configure upstream server
it could be HTTP/1.1 or HTTP/2 server, which has default page with two images

### Start `wget2` with `time`
```
time wget2 -v --recursive --no-parent --http2 --no-check-certificate https://localhost:10000
```

The issue description:

Last command takes 3 seconds instead of expecting 2

Expected behaviour:
* filter processing root page (index.html) in 1 second and return it to the wget2 client
* `wget2` requests both images in the same time,
* filter processing both images simultaneously in different threads (workers) in 1 second
* result time is about 2 seconds

Real behaviour:
* filter processing root page (index.html) in 1 second and return it to the wget2 client
* `wget2` requests both images in the same time,
* filter processing both images sequentially in 2 seconds
* result time is about 3 seconds

The goal ist configure (or change) envoy to make it possible to process resources inside one HTTP/2 stream in different workers (or threads) simultaneously

