---
layout: post
title:  "ACT-R Tutorial Unit6"
date:   2020-11-20 14:42:14 +0800
categories: 認知科学
tags: ACTR_Tutorial ACT-R
---
#### Utility Learning
<img src="{{site.baseurl}}/assets/figs/post-20-11-20/bst.gif" width="500px">

#### The Utility Theory
___
- noise is controlled by utility noise parameter <span style="color:red">:egs</span>
- noise is a logistic distribution with a mean of 0 and a variance of

	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;\sigma^2 = \frac{\pi^2}{3}s^2" />

- If multiple production matches, then the probability to fire is :

	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;Probability(i) = \frac{e^{U_i / \sqrt{2}s}}{\sum_j{e^{U_j / \sqrt{2}s}}}" />

    - implemented in program ：chose the one with highest utility

#### Building Sticks Example
___
  - we have unlimited building sticks of three lengths
  - goal :  create a target stick of a particular length
  - 2 basic strategy:
    - undershoot : start with a smaller one : add others
    - overshoot : start with a "too long" one ,then saw off others

- The cognitive model doing the bst task:
	- <img src="{{site.baseurl}}/assets/figs/post-20-11-20/bst.gif" width="200px">
	- over  = abs(G - B)
	- under = abs(G - C)
	- chose the one that is more closer ( 25 more closer than the other)
    - if there is a clear difference , than there will be 3 productions can fire:
      - decide-under / decide-over
      - force-under
      - force-over

#### Utility Learning
___
- if enabled utility learning, the utility will be updated as the model runs based on rewards
- <img src="https://latex.codecogs.com/svg.latex?\Large&space;U_i(n) = U_i(n-1) + \alpha (R_i(n) - U_i(n-1))" />

  - <img src="https://latex.codecogs.com/svg.latex?\Large&space;\alpha" /> : learning rate  ,  default  = 0.2  <span style="color:red">:alpha</span>

  - <img src="https://latex.codecogs.com/svg.latex?\Large&space;R_i(n) = reward - time" />
  - time is from production selected to reward received.

  - give less reward to more distant productions
  - the reinforcement goes back to all of the productions which have been selected between the current reward and the previous reward.

- ways to provide reward:
	- attaching rewards
		- use trigger-reward command (See [CodeUnit6](link))
		- rewards will be applied after the corresponding production fires
		- > (spp read-done :reward 20)
		- > (spp pick-another-strategy :reward 0)
	- consider the following situation:
      - > ...  --> pick-another-strategy  --> p1 --> p2 --> p3 --> read-done
      - in this case , only p1 p2 p3 which fire after pick-another-strategy will receive rewards
      - read-done will receive its own reward
      - pick-another-strategy won't receive any reward
      - production before pick-another-strategy  will receive negative rewards

#### Learning in the Building Sticks Task
___
  - enable utility learning  <span style="color:red">:ut</span>
  - turns on the utility learning trace  <span style="color:red">:ult</span>

#### Additional Chunk-type Capabilities
___
-  **Default chunk-type slot values**
	- ```lisp
	+visual>
   		isa move-attention    ;// is equivalent to "cmd move-attention" 因为cmd的默认值就是move-attention
   		screen-pos =c
	```
    - 使用isa关键字，不仅能够定义chunk type ，同时也能创建chunk（通过使用默认值）
      - 没有默认值 : (**chunk-type** *_name* *_slot1*  *_slot2*)
      - 给特定slot设定默认值： (**chunk-type** *_name* ( *_slot1* 1) ( *_slot2* 2))

- **Chunk-type hierarchy** 面向對象 继承制 (subtype)
	- use (:include typeA)
	- line is a subtype of visual-object
	- subtype contain all the slots that its parent has
	- can contain additional slots
	- can use different default value for some slots
	- 一个子type能够继承多个父type
	- > (chunk-type (d (:include a) (:include b)) slot4)   d 继承a和b
	- the parent types also gain access to all of the slots from their children.
	- 当使用父type来创建chunk的时候，也能够使用子type里的属性，比如：
	- > (define-chunks (isa visual-object line_start 1 line_end 2) )
	- line_start 和 line_end 是 子type “line”的属性
