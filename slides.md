---
theme: default
# background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: prism
lineNumbers: false
info: |
  Beyond big O

  A performance model that actually helps you IRL
drawings:
  persist: false
transition: slide-left
title: Beyond Big O
Author: Avishai Ish-Shalom
mdc: false
layout: intro
colorSchema: dark
---

# Beyond Big O
## Avishai Ish-Shalom

---
title: Same algorithm, different performance
---
# Explain dis

```cpp {all|2,3}{at:0}
for(int i = 0; i < n; ++i) {        
  for(int j = 0; j < n; ++j) {                
    for(int k = 0; k < n; ++k) {      
         C[i][j] += A[i][k] * B[k][j];            
      } 
   }    
}
```

***

```cpp {all|2,3}{at:0}
for(int i = 0; i < n; ++i) {        
   for(int k = 0; k < n; ++k) {      
      for(int j = 0; j < n; ++j) {                
         C[i][j] += A[i][k] * B[k][j];            
      } 
   }    
}
```

x5 faster!!!
---

# Big O

Algorithmic time/memory complexity

<img src="/images/linear-o-dark.svg" />

---

# But in reality...
<img src="/images/actual-linear-dark.svg" />

---
disabled: true
title: List insert example
---

# In reality....
<img src="/images/list-insert-latency-dark.svg" />
---

# Wat
- Implementations matter
- Hardware matters: chokepoints, saturation, characteristic sizes, etc
- IRL everything has a distribution

---
layout: two-cols-header
image: ./public/images/Deming.jpg
class: big-quote
title: Why performance theory
---

# Performance "theory"?
## wat??

::left::

> Experience by itself teaches nothing...\
> Without theory, experience has no meaning.\
> Without theory, one has no questions to ask.\
> Hence, without theory, there is no learning.
>  <footer><cite>W. Edwards Deming</cite></footer>

::right::
<img src="/images/Deming.jpg" />

---
layout: fact
---

# All models are wrong
## But some are useful

---

# Beyond Big O

We need a model which is
- Rooted in hardware and physical/logical structure
- Accounts for distributions
- Follows _implementations_, not _algorithms_

---

# Computers be like

<img src="/images/computer-parts.svg" />

---

# Numbers every developer should know

<LatencyNumbers />

<div class="absolute bottom-0 right-0">
<QRCode value="https://colin-scott.github.io/personal_website/research/interactive_latency.html" height=100 width=100 />
</div>


---

# The interaction buffer
<img src="/images/interaction-buffer-dark.svg">

---

# A crash course in Queueing theory
<img src="/images/queue-dark.svg">

---

# Tasks should be independent, but...
Head of line blocking

<img src="/images/head-of-line-blocking.svg" />

---
layout: image-right
image: /images/queue-latency-dark.svg
backgroundSize: 100%
---

# Queueing latency

- Latency rises non-linearly with utilization
- Head-of-line blocking

<div class="queue-equation" style="margin-top: auto; float: left;">
$$
Latency \propto \frac{\rho}{1 - \rho}
\\
\rho = Utilization
$$
</div>

---

# Variance and Queueing

- The variance of both _service center_ and _arrival rate_ impact queueing
- Higher variance &rarr; much higher latency

<div class="queue-equation" style="margin-top: auto; float: left;">
$$
Latency \propto \sigma^2
$$
</div>

<QueueModel model="kingman"></QueueModel>

---

# Multiple workers/Parallelization
- Shared queue, multiple workers &rarr; lower mean latency
- Stateful load balancing
- Better batching

<QueueModel model="mmc"></QueueModel>

---

# Quantum sizes

- Messages must fit in fixed size
- Split large messages
- Small messages either
   - Waste capacity
   - Amplify throughput 

---

# Fragmentation

- _N_ messages, instead of 1
- More chances for errors
- Wait for re-assembly 

---

# Batching 101
- Queue small tasks, try to bin pack into larger containers
- Amortize setup fees
- Larger queue -> better packing efficiency
- Larger queue -> higher latency

<img src="/images/interaction-buffer-batching-dark.svg" />

---

# Batching 102
Some tasks are related
- Mutations of the same record
- Reads from the same block
- Sequential reads

Example: I/O scheduler

---

# Batching 103
Tradeoffs, tradeoffs everywhere
- Can increase processing time (need to wait for all)
- Can amplify failures

---

# Errors
Sometimes messages vanish into the void

- Lost messages reduce throughput
- We may need to _retry_
- Extra overhead to detect loss

<img src="/images/interaction-buffer-errors-dark.svg" />
---

# Timeouts
- Latency can be potentially infinite
- And we have head-of-line blocking
- Which kills throughput
- Timeouts convert latency into errors

---

# Retries
If it first you don't succeed...
- Retries convert errors into latency
- But not everything is retryable...
- Consumes throughput, increases load

---

# Layers
Sometimes we need to cross several layers (e.g. caches, SSTables)
- Read amplification
- Latency amplification

<img src="/images/layers-traversal-dark.svg" style="padding-bottom: 10%; display: block; margin-left: auto;" />

---

# Mo layers

<img src="/images/layers-latency-riddle.svg" />

---

# The curse of latency amplification
<img src="/images/layers-latency-amplification.svg" />

---

# Backpressure

<img src="/images/interaction-buffers-layers-dark.svg" />

---
layout: section
---

# Let's get practical

---


<img src="/images/simple-web-system.svg">

---

<img src="/images/simple-web-system-1.svg">

---

<img src="/images/simple-web-system-2.svg">

---

<img src="/images/simple-web-system-3.svg">

---

# So what have we learned?
Amplifications!
- Latency (queueing, timeouts, retries, batching)
- Throughput (fragmentation, retries)
- Data (quantum sizes, layers)

---

# What to do?
- Know your queues
- Reduce your variance
- Control timeouts, retries
- Latency/Throughput tradeoffs
- Beware of layers
- Backpressure & load management

Every one of those is a topic in itself

---
layout: end
---

# May the perf be with you

---