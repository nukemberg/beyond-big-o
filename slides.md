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
---

# Beyond Big O
## Avishai Ish-Shalom

---
layout: two-cols-header
image: ./public/images/Deming.jpg
class: big-quote
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

# Big O

Algorithmic time/memory complexity

<img src="/images/linear-o.svg" />

---

# But in reality...
<img src="/images/actual-linear.svg" />

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

# Wat
- Implementations matter
- Hardware matters: chokepoints, saturation, characteristic sizes, etc
- IRL everything has a distribution

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

# The interaction buffer
<img src="/images/interaction-buffer.svg">

---
layout: image-right
image: https://raw.githubusercontent.com/nukemberg/the-math-of-reliability/master/images/queue-latency.svg
backgroundSize: 100%
---

# Queueing 101


- Latency rises non-linearly with utilization
- Heavily depends on variance
- Head-of-line blocking

<div class="queue-equation" style="margin-top: auto; display: relative; position: 0px; ">
$$
\frac{\rho}{1 - \rho}
$$
</div>

---


# Quantum sizes

- Messages must fit in fixed size
- Split large messages
- Small messages either
   - Waste capacity
   - Amplify throughput 

---

# Batching 101
- Queue small tasks, try to bin pack into larger containers
- Larger queue -> better packing efficiency
- Larger queue -> higher latency

<img src="/images/batching.svg" />

---

# Errors
Sometimes messages vanish into the void

- Lost messages reduce throughput
- We may need to _retry_
- Extra overhead to detect loss

<img src="/images/interaction-buffer-errors.svg" />
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
- Consumes throughput
- And can head-of-line block

---

# Layers
Sometimes we need to cross several layers (e.g. caches, SSTables)
- Read amplification
- Latency amplification

<img src="/images/layers-traversal.svg" style="padding-bottom: 10%; display: block; margin-left: auto;" />

---

# Layers

<img src="/images/interaction-buffers-layers.svg" />


---

# All together now!


---
layout: end
---
# May the perf be with you
