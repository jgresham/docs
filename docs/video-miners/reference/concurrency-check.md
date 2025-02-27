---
title: Assess concurrent stream capabilities
sidebar_position: 7
---

Once you have confirmed that your [hardware](/video-miners/reference/gpu-support) is supported by Livepeer, you should assess how many concurrent streams it can support.

## Hardware functionality and constraints

The Livepeer protocol enables those with the excess hardware and bandwidth available to earn additional revenue by advertising video encoding services on an open marketplace, and using their idle hardware to perform the work. There are a number of different types of hardware capable of encoding video in a performant and cost-effective manner, each with its own unique capabilities and terms of use. 

Some of these terms restrict users from utilizing their own hardware to its full capacity through artificial restrictions. While googling for open-source patches reveals workarounds to these limitations, Livepeer encourages operators on the network to read and comply with the terms of service and usage policies of the hardware that they are using.

### NVIDIA

If you are using an NVIDIA card, check you are running the latest driver version on your NVIDIA GPU, or update your driver before proceeding to benchmarking. You can access the GPU configuration, Display Adapters, and drivers for your operating system either directly or through your NVIDIA Control Panel. 

Concurrent session caps for NVIDIA hardware can be found [here](https://developer.nvidia.com/video-encode-decode-gpu-support-matrix).

## Testing 

You can test the performance of your card using the `livepeer_bench` [benchmarking tool](/video-miners/guides/benchmarking).
