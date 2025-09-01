---
type: "source-note"
title: "Interactive Guide: Mastering Rate Limiting"
id: 20250613090636
created: 2025-06-13T09:44:36
source: "web"
url: "https://blog.sagyamthapa.com.np/interactive-guide-to-rate-limiting"
tags:
  - "source-note"
processed: false
archived: false
---
![An Interactive Guide To Rate Limiting](https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/a-hCmlnehyU/upload/9b95d9fd3e800c60d20595e644eb1e2d.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp)

An Interactive Guide To Rate Limiting

Photo by [Makarios Tang](https://unsplash.com/@makariostang?utm_source=Hashnode&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=Hashnode&utm_medium=referral)

## An Interactive Guide To Rate Limiting![Sagyam Thapa's photo](https://cdn.hashnode.com/res/hashnode/image/upload/v1748582669746/55240e72-142e-489a-b93c-0ce2f52775fb.jpeg?w=200&h=200&fit=crop&crop=faces&w=500&h=500&fit=crop&crop=entropy&auto=compress,format&format=webp&auto=compress,format&format=webp)

Sagyam Thapa's photo

[Sagyam Thapa](https://hashnode.com/@Sagyam)

·

2 min read

## Introduction

Rate limiting is a must have strategy in every back-end app. It prevent one user from overusing a resource and degrading the quality of service for other users. Here are some benefits of rate limiting

- It presents resource starvation
- Reduces server hosting cost
- Provides basic protection against [DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack)

I have made four interactive app that let’s you play around with common rate limiting algorithms.

## Token bucket

### Working:

- A bucket holds fixed number tokens
- Tokens are added to bucket at fixed rate
- When a request comes in:
	- If a token is available, it’s removed from the bucket and the request is allowed.
	- If no tokens are available, the request is rejected or delayed.
- Allows for occasional short burst if tokens are available

I have created an [app](https://tools.sagyamthapa.com.np/token-bucket) that let’s you play with leaky bucket algorithm.

## Leaky bucket

### Working

- Think of it as a bucket leaking at a fixed rate
- Incoming requests are added to the bucket
- Requests are processed (or "leak") at a **constant rate**
- If the bucket is full when a new request arrives, the request is dropped
- Smooths out bursts; outputs requests at a steady rate
	I have made an [app](https://tools.sagyamthapa.com.np/leaky-bucket) that let’s you play with leaky bucket algorithm.

## Fixed window counter

### Working:

- Time is divided into fixed size windows (e.g., 1 minute)
- A counter tracks the number of requests per client/IP in the current window
- If the count exceeds the limit, further requests are rejected until the next window
- Simple and efficient, but allows burst traffic spike at end/start
	I have created an [app](https://tools.sagyamthapa.com.np/fixed-window) that let’s you play with fixed bucket algorithm.

## Sliding window counter

### Working:

- Keeps a timestamped log of each request
- When a request comes in, logs are checked to count how many requests were made in the last `X` seconds
- If under the limit, the request is allowed and logged; otherwise, it’s rejected
	I have created an [app](https://tools.sagyamthapa.com.np/sliding-window) that let’s you play with sliding bucket algorithm.