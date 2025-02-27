---
title: Set Session Limits
---

This guide provides instructions on setting [session](/video-miners/terminology#session) limits to manage transcoding capacity. You will be able to maximize work received while also protecting against performance degradation due to overload. Once you have completed the benchmark transcoding via the `livepeer_bench` tool, provided with the [Livepeer installation](/installation/install-livepeer/), and the [common output rendition configuration](https://github.com/livepeer/go-livepeer/blob/master/cmd/livepeer_bench/transcodingOptions.json) `transcodingOptions.json` you stored during the benchmarking, you can work with this guide to fine tune your configuration:

- Calculate session limits
- Evaluate hardware capacity
- Benchmark transcoding for a range of concurrent sessions 
- Evaluate bandwidth
- Derive a session limit
- Set session limits

> **Note:** It is recommended you complete the steps in [Benchmark transcoding](/video-miners/guides/benchmarking) before proceeding.

## Livepeer Session Limits

For orchestrators and transcoders, the **default limit of concurrent sessions is set to 10**. When this limit is exceeded, the orchestrator returns an error, `OrchestratorCapped`, to the broadcaster and transcoders and they will stop receiving work from orchestrators. The session limit should then be set depending on available hardware and bandwidth.

## Calculating session limits

The session limit is an estimate and may require adjustments after running live on the network.

The bandwidth and computational power needed to transcode a video stream varies with the source video and requested output configuration. 

Session limits are passed through the `-maxSessions` parameter to the node where they should be set based on taking the minimum of:

- Transcoding Hardware, and 
- Bandwidth  

## Set Session Limits

The following steps require that incoming streams are configured with the common web video streaming [Adaptive Bitrate (ABR) ladder](/video-miners/terminology) on the Livestream network. Session limits can be similarly calculated for a different ABR ladder.

1. Evaluate hardware

    The recommendation for determining a hardware level limit is to use the concurrent sessions value of the last log indicating that the real-time duration ratio `X` was less than or equal 0.8. This leaves a ~20% buffer for upload/download within real-time.

- You can use the following script with the  `livepeer_bench` tool to benchmark transcoding for a range of concurrent sessions:

```bash
#!/bin/bash
for i in {1..20}
do
    ./livepeer_bench \
        -in bbb/source.m3u8 \
        -transcodingOptions transcodingOptions.json \
        -nvidia 0 \ # Only required for transcoding with Nvidia GPUs
        -concurrentSessions $i |& grep "Duration Ratio" >> bench.log
done
```

- Transcoding with multiple identical Nvidia GPUs: 

    Benchmarking with a single GPU should suffice and then you can multiply the limit for a single GPU by the number of available GPUs. If you are transcoding multiple different Nvidia GPUs, you should benchmark each unique GPU and determine a limit that is specific to that GPU.

- Adjust the loop range (`{1..20}`) to reflect the maximum number of concurrent sessions you want to benchmark. If at 20 maximum concurrent sessions real-time duration is still below 1.0, you should increase the maximum number of concurrent sessions. 

- View the final output in a file called `bench.log`.

**For Example:**

```bash
| * Real-Time Duration Ratio * | <Ratio> |    // Concurrent Session Count 1
| * Real-Time Duration Ratio * | <Ratio> |    // Concurrent Session Count 2
...
| * Real-Time Duration Ratio * | <Ratio> |    // Concurrent Session Count 20
```

>   **Note:** If you are transcoding with a CPU, you will likely want to lower the value of the maximum number of concurrent sessions (i.e. 5).

2. Evaluate bandwidth

The most [common output rendition configuration](https://github.com/livepeer/go-livepeer/blob/master/cmd/livepeer_bench/transcodingOptions.json) found on the network is (assuming source is `1080p30fps`):

```bash
+------------------------+----------+
| Resolution & Fps       | Bitrate  |
+========================+==========+
| 1080p30fps             | 6000kbps |
+------------------------+----------+
| 720p<source>fps        | 3000kbps |
+------------------------+----------+
| 480p<source>fps        | 1600kbps |
+------------------------+----------+
| 360p<source>fps        | 800kbps  |
+------------------------+----------+
| 240p<source>fps        | 250kbps  |
+------------------------+----------+
```

A single stream requires an approximation of:

- _Download Bandwidth_ = **6 Mbps** Per Stream. This is about 6000 kbps for downloading the source rendition. 

- _Upload Bandwidth_ = **5.6 Mbps** Per Stream. This is about 5600 kbps for uploading the output rendition.

- To estimate the number of streams you can process, divide the above from your network provider's limits. 


**For example:**

 A typical broadband connection with `upstream/downstream` speed of `100 Mbps` should reliably be able to serve/process ~16 streams. 

 However, as not all streams' segments may be processed at the same time, you may be able to extend this by an additional ~20%. 

> **Note:** [Bandwidth Requirements](/video-miners/reference/bandwidth) provides further information about testing your available upload/download bandwidth.

3. Derive a session limit based on hardware and bandwidth

Once you have calculated a hardware level limit and a bandwidth level limit, the minimum of the two can be used as your session limit. This is set via the `-maxSessions` flag.

Session management in orchestrators and transcoders is still constantly being improved. Your mileage may vary with this approach; you may find that your orchestrator or transcoder performance may be affected with a higher session limit. 

Further adjusting the session limit values after performing work on the network may be necessary.

4. Set session limits

The `-maxSessions` flag is used to set session limits on both orchestrators and transcoders.

**For Example:** 

For a combined orchestrator and transcoder, set the session limit to 30:

```bash
livepeer \
    -orchestrator \
    -transcoder \
    -maxSessions 30
```

> **Note:** For the purpose of this example, other flags have been omitted: