---
layout: post
title:  "ACT-R Tutorial Unit4"
date:   2020-11-10 00:36:14 +0800
categories: 認知科学
tags: ACTR_Tutorial ACT-R
---
#### Activation of Chunks and Base-Level Learning
-  Introduce the subsymbolic part of ACT-R
- <img src="{{site.baseurl}}/assets/figs/post-20-11-10/result2.png" width="500px">


#### 1. Introduction
___
- enable subsymbolic components:
> (spg :esc t)

#### 2.Activation
___
- any chunk associated with a value : activation
- reflects :
	- the degree to which past experiences
	- the degree to which current context indicate that chunk will be useful
- retrieval threshold:
- > (sgp :rt -0.5)
- sets the minimum activation a chunk can have and still be retrieved.

- Activation  = base level + context + noise

#### 3.Base-level Learning
___
- <img src="https://latex.codecogs.com/svg.latex?\Large&space;B_i = ln(\sum_{j=1}^n{t_j^{-d}})" title="\Large B_i = ln(\sum_{j=1}^n{t_j^{-d}})" />
- $  $
- d : decay rate
- n : The number of presentations for chunk i.
- t  : The time since the jth presentation.
- two types of presentation:
	- **initial entering into DM**
	> add-dm

	- When the chunk is cleared from a buffer. ( will be add into DM)
		- **merge with another that is already in DM**
		- If a chunk has the same set of slots and values as a chunk which already exists in the DM
		- preexisting chunk in DM is credited with a presentation.
		- repeatedly attending the same visual stimuli would result in strengthening a single chunk that represents that object. (重复看到同一个刺激会加强对于该刺激的记忆)

#### 4.Optimized Learning
___
- take time for computation
- an approximation approach
	- turning on the optimized learning parameter - :ol.　　(default  is  on)
	- 使用条件： the presentations are approximately uniformly distributed over the time since the item was created.
- <img src="https://latex.codecogs.com/svg.latex?\Large&space;B_i = ln(\frac{n}{1-d}) - d * ln(L))" title="\Large B_i = ln(\frac{n}{1-d}) - d * ln(L)" />
- n: The number of presentations of chunk i.
- L: The lifetime of chunk i (the time since its creation).
- d: The decay parameter.


#### 5.Noise
___
- logistic distribution characterized by a parameter s
- <img src="https://latex.codecogs.com/svg.latex?\Large&space;\sigma^2 = s^2*(\pi^2/3)" title="\Large \sigma^2 = s^2*(\pi^2/3)" />
- two source of noise
	- permanent noise (s is set by <span style="color:red"> :pns</span>))
	- instantaneous noise  ( recomputed at each retrieval attempt )   ( s is set by <span style="color:red"> :ans</span>)
	- typically turn permanent noise off
> (sgp :pns nil)
 (sgp :ans t)

#### 6.Probability of Recall
___
- <img src="https://latex.codecogs.com/svg.latex?\Large&space;recallP_i = \frac{1}{1+e^{\frac{\tau-A_i}{s}}}" title="\Large recallP_i = \frac{1}{1+e^{\frac{\tau-A_i}{s}}}" />
- In fact, when τ = Ai, the probability of recall is .5
- s controll the sensitivity of recall
	- s 接近0 ：  概率在0，1之间反复横条
	- s 较大  ：  平滑的sigmoid曲线


#### 7.Retrieval Latency
___
- The activation of a chunk also determines how quickly it can be retrieved.
- <img src="https://latex.codecogs.com/svg.latex?\Large&space;Time = F*e^{-A}" title="\Large Time = F*e^{-A}" />
- A: The activation of the chunk which is retrieved.
- F: The latency factor (set using the <span style="color:red"> :lf </span>)).
- if no chunk match or no chunk exceed threshold, then time it takes for the failure to be signaled is :
	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;F*e^{-\tau}" title="\Large F*e^{-\tau}" />
	- τ: The retrieval threshold.

#### 8. The Paired-Associate Example
___
- paired stimuli prented for 5s each
	- word ----- number
	- zinc ---- 9
	- xray ---- 8
	- girl  ---- 7
	- boy ---- 6
- first time presented:  learn the pairs
- 随着练习次数增加（trial），回答准确率和反应速度上升
- モデルの流れ図：
- <img src="{{site.baseurl}}/assets/figs/post-20-11-10/flowchart.png" width="250px">
- detect-study-item 和 associate production:  加强记忆
- ```lisp
(p detect-study-item
    =goal>
		isa goal
		state read-study-item
	=visual-location>
    ?visual>
    	state    free
 ==>
   	+visual>
    	cmd   move-attention
    	screen-pos =visual-location
	=goal>
    	state    attending-target
)
```

- ```lisp
(p associate
    =goal>
      isa      goal
      state    attending-target
    =visual>
      isa      visual-object
      value    =val
   =imaginal>
      isa      pair
      probe    =probe
    ?visual>
      state    free
  ==>
   =imaginal>
      answer   =val    ; 将 probe 和 answer 关联起来
   -imaginal>          ; 关联好的pair存储到DM中
   =goal>
      state    start  
   +visual>
      cmd      clear
)
```
- The associate production fires after the model reads the number which is associated with the probe
- 如果没有开启 activation component，则从第二个trial开始，模型的正确率将始终保持100%，且反应时间也没有变化
- <img src="{{site.baseurl}}/assets/figs/post-20-11-10/result1.png" width="500px">

- 开启 activation component 之后，もっと人間っぼく振舞う
- <img src="{{site.baseurl}}/assets/figs/post-20-11-10/result2.png" width="500px">


#### 9. Parameter estimation
___
- 4 parameters should be set
	- the retrival threshold  = -2 (default)
		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;\tau   = -2" title="\Large \tau   = -2" />
	- the instantaneous noise = 0.5 (determines how quickly probability of retrieval changes as we move past the threshold.)
		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;s=0.5" title="\Large s=0.5" />
	- latency factor is set at 0.4.   (determines the magnitude of the activation effects on latency.)
		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;F=0.4" title="\Large F=0.4" />
	- decay rate for base-level learning is set to the value 0.5 (recommend it be set for most tasks )
		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;d=0.5" title="\Large d=0.5" />


#### 10. The Activation trace
____
> (sgp :act t)

#### 11. The :ncnar Parameter
___
- normalize chunk names after run
- do not affect model's performance
- affect actual time it takes to run the simulation, turn off can save time
