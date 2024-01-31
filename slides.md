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
background: ./images/linear-o.png
---
# Big O

Algorithmic time/memory complexity

<img src="/images/linear-o.svg" class="" />

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
- IRL everything has a distribution
- Hardware matters: chokepoints, saturation, characteristic sizes, etc

---
layout: center
class: text-center
---

# All models are wrong
## But some are useful

---
layout: image-right
image: ./public/images/Deming.jpg
---

>  Experience by itself teaches nothing... Without theory, experience has no meaning. Without theory, one has no questions to ask. Hence, without theory, there is no learning.

\- W. Edwards Deming

<style>
blockquote {
  font-size: 30px
}
</style>

---
