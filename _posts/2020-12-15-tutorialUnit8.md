---
layout: post
title:  "ACT-R Tutorial Unit8"
date:   2020-12-15 14:42:14 +0800
categories: 認知科学
tags: ACTR_Tutorial ACT-R
---
#### Advanced Production Techniques
___
  - Talk about 2 additional mechanisms in the procedural system
    - procedural partial matching
    - dynamic pattern matching

#### Procedural partial matching
___
- 选择production的时候，不一定会选择到条件完全符合的，其他的production也会有机会被选择
- <span style="color:red">:ppm</span>
- **Condition testing with procedural partial matching**
- conditions must be true: 必须完美匹配的条件
	- test the buffer must contains a chunk
	- all queries
	- all inequality tests on slots (!=, < , >)
	- special conditions specified with eval or bind operators
- conditions must be true: 可以不必完美匹配的条件：
	- equality tests for the slots of the chunk in a buffer
	- eg.  a = 1  ,   a=b
	- the mismatching values must be similar.
	- 只有那些相似的production才会被选择
		- 例如：buffer是 a=1,   p1的条件是a=1.1,  则p1也有可能被选择
	- use <span style="color:red">set-similarities</span> command
- Procedual Partial Matching   VS.   Declearitive Memory Partial Matching
	- in PPM,  only considered when similarity value which is greater than the current maximum similarity difference ([~ :md])
	- by default, all chunks has similarity of [~ :md]. thus  otherwise any chunk will be a match to any other in the PPM
	- this disallowing can control which items are allowed to be partial matched in productions.
	- default is -1.0 (max difference , min similarity)

- **Conflict resolution with procedural partial matching**
	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;Utility_i(t) = U_i(t) + \epsilon + \sum_j{ppm*similarity(d_j,v_j)}" />
		-  <img src="https://latex.codecogs.com/svg.latex?\Large&space;d_j"/> : desired value for slot j in production i

		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;v_j"/> : the actual valuue in slot j in the chunk in the buffer

		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;ppm"/> : global parameter

		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;\epsilon"/> : noise
		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;j"/> : set of slots that production i had a partially matched value
- productions that are not a perfect match receive a reduction in utility
- That adjusted utility is used only for purposes of conflict resolution
	- not for utility learning
	- not for production compilation

- **Simple procedural partial matching model**
	- tutorial/unit8/simple-ppm-model.lisp
	- The conflict set trace parameter
		- <span style="color:red">:cst</span>
		- model debugging aid
		- similar to the activation trace paramter
		- print the matched productions with their utility
```lisp
(P MEDIUM
    =GOAL>
       VALUE  [MEDIUM, SMALL, -0.5]       ;// printed in the trace of conflict set indicating this production is a partial match
      VALUE SMALL                                 ;// binded value is the value in the buffer (not the one in the production)
   ==>
     !OUTPUT! SMALL
   -GOAL>
 )
```
	- Turning on ppm
	- The large production
		- no similarity set, thus will not be considered as a candidate of partial matching
	- Adding noise
		- <span style="color:red">:egs</span> 1
		- 40% of the time production middium will be selected, but printed value is still SMALL
	- **Building Sticks Task alternate model**
		- The original Task is in [Unit6](link)
			- Original  :  sperate production for under strategy and over strategy
			- Alternate:  only has a decide production for each strategy
				- tutorial/lisp/bst-ppm.lisp
				- tutorial/unit8/bst-ppm-model.lisp
				- (bst-ppm-experiment n)
		- Strategy choice
```lisp
(p decide-over
	=imaginal>
      isa                encoding
      goal-length    =d          ;比较goal和c的长度
      b-len             =d
==>
)
```
```lisp
(p decide-under
	=imaginal>
      isa                encoding
      goal-length    =d          ;比较goal和b的长度，partial matching机制可以自动帮助我们调整utility
      c-len             =d
==>
)
```


#### Dynamic Pattern Matching
___
- Useful where
	- instruction or example following are involved
	- flexibility or context dependence are necessary
- **Basic Operation**
	- demo :  /tutorial/unit8/simple-dynamic-model.lisp
- **Arbitrary slots in conditions**
	- the ability to test arbitrary slots in the conditions of a production.
	- 根据上下文，测试任意slot （之前都是只能测试写好在程序中的固定slot）
	- <span  style="color:orange">slot name can also be a variable and dynamiclly instantiated</span>
  - > (p start
  - > =goal>
  - > context  =context     ;context slot保存需要测试的slot名
? =context =x           ; 测试的slot是一个变量
	- can be within or across buffer.
	- multiple times is OK
- **Arbitrary slots in actions**
	- A variable may be used to specify a slot in any
		- modification
		- modification request
		- request action of a production
	- if undefined slot is used, it may cause warning or error (depending on buffer)
- **Extending chunks with new slots**
	- add new slot names to chunk
	- in the trace, it shows :
	- >EXTEND-BUFFER-CHUNK IMAGINAL
- **Constraints on dynamic pattern matching**
	- No Search
	- finding a slot based on its value (NP-hard problems)
```lisp
 (p not-allowed
 =goal>
 isa goal
 =slot 1
```
	- One level of indirection
```lisp
 (p also-not-allowed
 =goal>
 isa example
 first-slot =s1
 =s1        =next-slot
 =next-slot value         ;caused 2 unbounded level
 ==> )
```
		- such a mechanism appears to be realizable within the basal ganglia of the human brain as shown in previous research

- **Example Models**
	- Semantic Model
		- knowledge representation
			- is dangerous
		- unit1:   attribute = dangerous ,   value = true
		- unit8:      dangerous(slot name) true
		- <span style="color:orange"> it is almost always better for the model to retrieve something and then test that result than to rely on retrieval failure.</span>
		- 第一次search之后自动在chunk加上父类的属性，比如 麻雀自动拥有鸟的属性，这样第二次search的时候就会变快（不需要在一遍历和向上搜索）
	- Paired-learning Model
	- 将固定在production当中的 slot名 （如 word ， number）变成变量 =slot
	- 这些变量将由dm中的instruction operator决定 (required =slot)
		- op1 required word
		- op2 required number
	- Dynamic matching and production compilation
		- complied productions can retain some of the dynamic components or replaced with statically matched values
	- Data fit
		- using dynamic matching did not influence the model performance
