---
layout: post
title:  "ACT-R Tutorial Unit5"
date:   2020-11-14 22:36:14 +0800
categories: 認知科学
tags: ACTR_Tutorial ACT-R
---
#### Activation and Context
<img src="{{site.baseurl}}/assets/figs/post-20-11-14/banner.png" width="500px">

#### 1. Spreading Activation
___
- <img src="https://latex.codecogs.com/svg.latex?\Large&space;B_i = A_i = B_i + \sum_{k}{\sum_j{}{W_{kj}S_{ji}+\epsilon}}" title="" />
- <img src="https://latex.codecogs.com/svg.latex?\Large&space;S_{ji} = S-ln(fan_i)" title="" />

-  <img src="https://latex.codecogs.com/svg.latex?\Large&space;S" title="" />: the maximum associative strength ( set with <span style="color:red">:mas</span> parameter)

	- by default <span style="color:red">:mas</span> = nil  to disable spreading activation

	- enable spreading activation , set  <span style="color:red">:mas</span> to a positive value

	- commonly, set high enough to make sure all of the <img src="https://latex.codecogs.com/svg.latex?\Large&space;S_{ij}>0" title="" />

- default, only <span style="color:blue">imaginal buffer</span> serves as a source of activation  
	-  <img src="https://latex.codecogs.com/svg.latex?\Large&space;W_{imaginal}=1" title="" /> , set with <span style="color:red">:imaginal-activation</span>

	- for other buffers :set with <span style="color:grey">:[bufferName]</span>-activation


#### 2.The Fan Effect
___
- study the following:
	- <img src="{{site.baseurl}}/assets/figs/post-20-11-14/fan.png" width="300px">
	- studied sentence : targets
	- new sentence :  foils
	- findings:
		- > more fan , more respond time
		- > foil take longer to respond than target

#### 3.Fan Effect Model
___
- fan-model.lisp  (experiment code :  fan.lisp)
- test for 1 sentence :
	- > ? (fan-sentence 'lawyer'  'store' t 'person')
- test for the whole experiment :
 - > ? (fan-experiment)
 - fitted parameter:
	- latency factor (<span style="color:red">:lf</span>)  = 0.63
	- maximum associative strength (<span style="color:red">:max</span>) = 1.6
	- <span style="color:red">W</span> = 1.0
- compare the result with human data :
 	- R = 0.864 after parameter tuning
	- <img src="{{site.baseurl}}/assets/figs/post-20-11-14/human.png" width="300px">
- **Model Representations**
	- words : ( base-level activation  = 10 )
		- <img src="{{site.baseurl}}/assets/figs/post-20-11-14/word.jpeg" width="300px">
    - sentence : ( base-level activation  = 0 )
		- <img src="{{site.baseurl}}/assets/figs/post-20-11-14/sentence.jpeg" width="300px">

- **Perceptual Encoding**
	- the study of fan effect verified that participants only fixate those two words from the sentence (person and location)
	- so the model is also modeled to only reads the 2 words and keep simple
	- find-person
	- ↓
	- attend-visual-location
	- ↓
	- retrieve-meaning
	- ↓
	- encode-person
	- ↓
	- attend-visual-location
	- ↓
	- retrieve-meaning
	- ↓
	- encode-location
- **Determining the Response**
	- after encoding, the imaginal buffer will have a representation for the sentence:
		- Imaginal :
			- ISA = comprehend-sentence
			- arg1 = Lawyer
			- arg2 = Store
	- then retrieve a knowledge from DM according to the info in imaginal buffer
		- 根据人物回忆 和 根据地点回忆 ， 两个production competing with each other and one would be randomly selected
		- 这样做是为了，当屏幕上出现的是 foil 的时候，总是会想起一个事实，而不是返回failure
		- 如果根据buffer failure判定foil，则respond time will only depend on retrieve threshold
		- in fact, human data clearly shows that the fan of the items affects the time to respond to both targets and foils
		- 简化模型：
          - 只运行 retrieve-from-person  数次
          - 只运行 retrieve-from-location 数次
          - 平均结果
```lisp
(P retrieve-from-person
	=imaginal>
		ISA    comprehend-sentence
		arg1   =person
		arg2   =location
	?retrieval>
		state  free
		buffer empty
==>
	=imaginal>  ; prevent strict harvesting
	+retrieval>
		ISA    comprehend-sentence
		arg1   =person     ;!!!!!!!!!!!!!!!!!!!!!!only retrieve one slot
```
- respond  :  3 productions
	- yes
	- mismatch-person
	- mismatch-location

#### 4.Analyzing the Retrieval of the Critical Study Chunk in the Fan model
___
- 按理说，production只检测了某一个属性，如person或者location
	- 因此会有多个chunk符合条件
	- 但是根据spreding activation，两个属性都符合的chunk拥有更高的activation
- **A note on chunks in buffers and the <span  style="color:red">:dcnn</span> parameter**
	- :ncnar     normalize chunk names after run
	- :dcnn    dynamic chunk name normalizing
	- oftern both set to t
	- sometimes it may be useful to debug for tracking chunks before merging
- **A simple target trial**
	- >laywer is in the store
	- location : 1fan  ,   person : 1fan
	- <span style="color:red">:act</span>  (activation trace parameter ) enabled to see the activation values when retrieve
	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;A_i = B_i + \sum_{k}{\sum_j{}{W_{kj}S_{ji}+\epsilon}}". title="" />

	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;B_i = 0"  , "S=1.6" title="" />

	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;S_{ji} = S-ln(fan_i)=1.6 - ln(2) = 0.907" title="" />

	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;W_{i1} = 0.5 ,  W_{i2} =0.5" title="" />

	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;A_i = 0 + 0.5 * 0.907  + 0.5 * 0.907 + 0 = 0.907"  title="" />

	- Time to retrieval the i-th chuk is :
		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;Time = Fe^{-A_i}  = 0.63 * exp(-0.907) = 0.254" title="" /> (s)

- **A different target trial**
	- > The hippie is in the bank
	- location : 2 fan,  person : 3 fan
	- it will take longer to respond ! (because s is lower, spreding activation is less)
	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;S_{hippie} = 1.6 -ln(4) = 0.214" />

	- <img src="https://latex.codecogs.com/svg.latex?\Large&space;S_{bank} = 1.6 - ln(3) = 0.501"  />

- **A foil trial**
	- > The giant is in the bank
	- person fan = 3 , location fan = 2
	- use retrieval-from-person:
		- matched studied sentences (3) only receive spread activation from "giant"
		- thus activation is less than target
		- thus it takes longer to respond for foils than targets
	- use retrieval-from-location:
		- different respond time
	- Use 2 productions and average the result is necessary! 不然模型的结果将会局限于只是用某一个item来搜索记忆的策略

#### 5.Partial Matching
___
- modeling errors human make !
	- commision  (回忆错误)

	<img src="https://latex.codecogs.com/svg.latex?\Large&space;A_{wrongChunk}  > A_{rightChunk}" />

	- omission (想不起来任何)  

	<img src="https://latex.codecogs.com/svg.latex?\Large&space;RT > A_i " />

	- A case :
		- probe :   The giant is in the bank
		- a studied sentence : The titan is in the bank
	- enable partial matching:
		- (sgp <span style="color:red">:mp</span> t)
		- 模型会自动考虑 buffer中的chunk 和 DM中的chunk的相似程度
		- With partial matching ,   the chunk might not have the exact slot values as specified in the retrieval request
		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;A_i = B_i + \sum_{k}{\sum_{j}{W_{kj}S_{ji}}} + \sum_{i}{PM_{li}} + \epsilon" />

		- specification elements <img src="https://latex.codecogs.com/svg.latex?\Large&space;l"  />  :  over all the slot values

		- Match Scale <img src="https://latex.codecogs.com/svg.latex?\Large&space;P"  /> : a constant across all slots and is set by <span style="color:red">:mp</span> = 0

		- Match Similarities <img src="https://latex.codecogs.com/svg.latex?\Large&space;M_{il}"  /> : similarity between two slot value

			- maximum similarity  <span style="color:red">:ms</span> = 0
			- maximum difference   <span style="color:red">:md</span> = -1.0
		- 实际上的计算：
			- 符合条件： 不给惩罚
			- 不符合条件： 给予惩罚，降低激活值

#### 6.Grouped Recall
- A example for illustrate partial matching
- Only use the cognitive part
	- no perceptual or motor is used
	- is useful when "timing" is not important
	- parameters:
		- s = 0.15
		- retrieve threshold = -0.5
		- base-level activation = 0
		- P = 1
		- spreading activation is disabled
- Group task
	- list should be recalled: 1,2,3,4,5,6,7,8,9
	- model response :  1,2,3,4, [~ 6,5], 7,8
	- activation is just :  sum(similarities ) + noise
- Error of Commission
	- ```lisp
		+retrieval>
			isa item
			group =group
			position second
			:recently-retrieved nil   ;  not a slot of the chunk  不计算相似度
```
	- 搞错5和6的顺序
		- item5 :    noise ( - 0.5 )  + similarity (0 ) = -0.5
		- item6 :    noise( 0.1 ) + similarity (-0.5)  = -0.4
		- 由于noise的影响， [$ A_{item6}]  >  [$ A_{item5}]
		- similarity is defined:
		- > (set-similarities (first second -0.5) (second third -0.5))
		- for  those different values that is not specified by "set-similarities" command, will have the default value of maximum difference which is -1.0, in this case : (first third -1.0)
- Error of Omission
	- 没有回忆起9
	- due to random noise,  <img src="https://latex.codecogs.com/svg.latex?\Large&space;A_{9} < rt" />

	- 没有其他chunk可以选择，因为item9 是唯一符合 :recently-retrieved nil 这个条件的

#### 7.Simple Addition
___
- human data (respond time)  :  (4-year-old kids)
	- <img src="{{site.baseurl}}/assets/figs/post-20-11-14/addition_human.png" width="300px">
	- strategy:
		- recall the result
		- if can't recall, then count

- **A Modification Request**
	- ```lisp
	(P harvest-arg2
		=retrieval>
		=imaginal>
			isa				 plus-fact
			addend2		nil
		?imaginal>
			state free
==>
		*imaginal>     ;// * means  a modification request. does not clear the buffer automatically
			addend2     =retreival
	)
	```
	- modification request can be done to <span style="color:orange">goal</span> and <span style="color:orange">imaginal</span> module, in the same way
	- For <span style="color:orange">goal</span> module:
		- using " = " to modify:
			- > 0.050 PROCEDURAL MOD-BUFFER-CHUNK GOAL
		- use "*" to modify:
			- > 0.050 PROCEDURAL MODULE-MOD-REQUEST GOAL
			0.050 GOAL MOD-BUFFER-CHUNK GOAL
		- use "=" , the goal module do modification directly
		- use * ,  procedule will make a request, then goal module do the modification
- **if one is trying to compare a model’s actions to human brain activity then knowing “where” an action occurred is important**
- For <span style="color:orange"> imaginal </span>module:
	- use * , there will be 200ms time cost to do the request
	- The *imaginal action is the recommended way to make changes to the chunk in the imaginal buffer because it includes the time cost for the imaginal module to make the change.

- **An indirect request**
	- ```lisp
	(P harvest-answer
		=retrieval>
			ISA  	plus-fact
			sum 	=number
		=imaginal>
			ISA 	plus-fact
		?imaginal>
			state 	free
==>
		*imaginal>
		   sum         =number
		+retrieval>    =number   ;!!!! an indirect request 实际上这里是 和=number绑定的chunk整体
)
	```
	- if the variable =number is bound to the chunk eight
		- chunk eight
			- value     8
			- name    "eight"
	- actual request made as the indirect request is:
		- +retrieval>    
			- value  8
			- name  "eight"
	- Parameters to be adjusted
		- retrieval threshold :rt
		- activation noise :ans
		- match scale :mp
		- base-level activation values of the chunks.

- Initial model
	- make sure the model can do the task perfectly with subsymbolic components diabled
	- assumptions:
		- know numbers of 0-9
		- have encountered the addition facts for problems with addends from 0 to 5.
		- do not try problem solving strategy, only recall the addition facts
		- it is important ： specific strategy  or include different strategies sometimes
		- 对于某个人，这个模型最好和这个人采取相同的解决问题的策略，才能更好的建模

- Making errors
	- retrieve incorrect number ❌
	- retrieve incorrect additional fact  or fail to  ✅
		- +retrieval>
			- ISA　　　　　plus-fact
			- addend1　　  =val1
			- addend2          =val2
- Setting similarities
	- it is not a reasonable paramter to be fit
	- similarity between numbers :
		- <img src="https://latex.codecogs.com/svg.latex?\Large&space;similarity(a,b) = - 0.1 * |a-b|" />

	- chose match scale:
		- make sure that we pick a value here which ensures that the similarity will make a difference in the activation values.
		- <span style="color:red">:mp=5</span>

- Activation noise
	- The more noise , there is the less likely to respond correctly
	- experience of the auther:
		- an activation noise value in the range of 0.0-1.0 has been a good setting
		- for most of those the value tends to fall somewhere between 0.2 and 0.5.
	- <span style="color:red">:ans = 0.5</span>
- Retrieval threshold and base-levels
	- base level activation of the number facts should be set high enough
		- 不然在encode阶段就会失败，这显然不合理
	- > (set-base-levels
      (zero 10) (one 10) (two 10) (three 10) (four 10) (five 10) (six 10) (seven 10) (eight 10) (nine 10) )

- Adjusting the parameters
	- search on only one parameter at a time.
	- mp = 16
	- ans = 0.7
- Adjusting the model
	- a fact of children
		- correct on small problem
		- when they respond incorrectly, the answers are more often smaller than the correct answer
			- 2+5 = 6  is more often than 2+5 = 8
			- 一般来说会少数，不会多数
		- There seems to be a bias for the smaller answers.
	- solution : increase base level activation of small adiition facts (sum < 4)
		- > (set-base-levels
     (f00 .1)(f01 .1)(f02 .1)(f03 .1)(f04 .1) (f10 .1)(f11 .1)(f12 .1)(f13 .1)
     (f20 .1)(f21 .1)(f22 .1)
     (f30 .1)(f31 .1)
     (f40 .1))
		- fitted value = 0.65
- other things can do :
	- modifying the similarities used to something other than linear


#### 8. Learning from experience
___
- 1-hit Blackjack
	- <img src="{{site.baseurl}}/assets/figs/post-20-11-14/blackjack.gif" width="300px">
	- card :  1 ~ 10
	- How to win :  collect cards whose sum is less than or equal to 21 and greater than the sum of the opponent’s cards.
	- 合計が21に近いカード，相手より大きい
	- 特殊规则：card 1
		- if sum < 21 , then it can be counted as 11
		- else, counted as 1
	- hit ： add a card
	- stay : not add a card
	- without knowing opponent's choice, act as quickly
	- 相手：a fixed strategy , but is not known by the model

- General modeling task description
	- no visual and aural, all game state is in goal buffer
	- 10 seconds to decide  "S" or "H"
		- if no key is pressed, considered as stay
	- after 10sec, the goal buffer is updated to refect game's action and the outcome of the game
	- use the info to decide if it should be learned

- Goal chunk specifics
	- start:
		- <img src="{{site.baseurl}}/assets/figs/post-20-11-14/start.png" width="300px">
	- after 10 sec:
		- <img src="{{site.baseurl}}/assets/figs/post-20-11-14/after.png" width="300px">
	- play 100 times
	- wining rate in each 5 hand group is computed

- Starting model
	- simple strategy of learning:
		- recall a similar experience
		- if a similar chunck and wined , recall the action
		- based on the feedback , create a chunk that holds the learned info for this hand
	- an overwritten action using "@"
	- ```lisp
	(p remember-game
	=goal>
		isa game-state
     	state retrieving
   =retrieval>
		isa learned-info
     	action =act
   ?manual>
     state free
 ==>
   =goal>
     state nil
   +manual>
	 cmd press-key
	 key =act
   @retrieval>   ;//  "@" : overwrite action  ,  erase and do not sent to DM
   ;=retrieval>
   ;  mc1 nil
   ;  action nil    ;// prevent merging and strengthen the bad play
 )
	```
	- modification action using "=" ,  erase all
	- only the slots and values specified in the overwrite action will remain in the chunk in the buffer , all other slots and values are erased.
	- When it is used without any modifications,
		- the buffer will be empty
		- the chunk which was there is not cleared and sent to declarative memory
	- To prevent that chunk from merging back into DM and strengthening the chunk which was retrieved.
		- the chunk retrieved may not be the best choice (noise or have not enough experience)
		- so , it should be erased and the model should wait for the feedback before creating a new chunk
			- win :  create
			- lose : not create
	- set similarity: use hook function
		- [$ similartiy(a,b) = -\frac{abs(a-b)}{max(a,b)}]
		- 该公式符合人类心理
		- less difference more similar.
		- larger numbers are more similar than smaller numbers for a given difference.
	- 2 set of parameters:
		- control how the model is configured
		> (sgp  
		:esc t      ; enable subsymbolic component
		:bll .5      ; base level learning with a decay rate of 0.5
		:ol t         ;optimized learning equation to speed up
		:sim-hook "1hit-bj-number-sims"    ;hook function of calculating similarities scores
		:cache-sim-hook-results t    ; to cache the similarity values
		:er t            ; Randomness is enabled
		:l_ 0)          ; latency factor. all retrievals complete immediately becase we have a time limit of 10 sec but do not have to fit latency data.
		retrievals complete immediately becase we have a time limit of 10 sec but do not have to fit latency data.
		- how the mechanisms used in the model
		- >(sgp :v nil :ans .2 :mp 10.0 :rt -60)
		- rt is very low and match scale is high (10) so that the model should always be able to retrieve some relevant chunk if there are any
