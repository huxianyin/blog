---
layout: post
title:  "ACT-R Tutorial Unit3"
date:   2020-11-3 01:07:14 +0800
categories: 認知科学
tags: ACTR_Tutorial ACT-R
---
#### Attention
<img src="{{site.baseurl}}/assets/figs/post-20-13-03/demo.gif" width="500px">
___

#### 1. Visual Locations
___
- ##### Visual-location requests
  - <span style="color:red">:attended</span>     该元素是否已经被看过
    - possible value :  new ,   nil  ,   t
    - 当多个符合条件，选择最近被加入 visual icon的 (chose the newest one )
    - 当没有符合条件的，then a failure will be signaled for the visual-location buffer.
    - Finsts
      - 数の制限：A limit of elements can be attented.   "Attended"とタグされるアイコンの数の上限はFinst数　
      - 時間の制限：there is also a time limit on how long an item will remain marked as attended t
      - parameter in the system :
        - :visual-num-finsts   = 4
        - :visual-finst- span   = 3
-  ##### Visual-location slots
    - (chunk-type visual-location screen-x screen-y distance height width size color kind value)
    - (the screen resolution of the experiment windows is assumed to be 72 pixels per inch
    - unit : pixels
    - screen-x  :  left to right
    - screen-y  :  top to bottom
    - distance  = 1080   (15 inches)
    - kind :  text, line , ...
- ##### Visual-location request specification
    - Exact values
     ```lisp
	 +visual-location>
			isa visual-location
			screen-x 50
			screen-y 124
			color black
			- kind text
	 ```
    - General values
     ```lisp
	 +visual-location>
			> screen-x 50
			<= screen-y 124
	 ```
    - Production variables
      ```lisp
	   (p find-by-color
    		=goal>
        	target =color
    ==>
      	+visual-location>
    		color =color)
	  ```
    - Relative values
		```lisp
	   ;// code1 : 从左到右依次看过去的代码：
	  +visual-location>    
			:attended nil
			screen-x lowest
		```
		```lisp
		;// code2 : 先考虑绝对条件，再考虑相对条件的代码
		+visual-location>
			width highest
			screen-x lowest
			color red
		```
	 - lowest, highest, nearest
     - 当查找符合条件的item，ACT-R会优先看有绝对数值的条件，然后再看相对条件，比如code2中，会先考虑红色，再考虑最宽的，再考虑位置最靠左边的

    - The current value
      ```lisp
	   ;// 查找位于当前元素的正上方，且拥有与当前元素不同颜色的元素
         +visual-location>
           screen-x current
           < screen-y current
           - color current
		 ```
		- It is also possible to use the special value <span style="color:red"> :current</span> in a visual-location request.
		- If the model does not have a currently attended object (it has not yet attended to anything) then the tests for current are ignored. 如果当前没有正在看的元素（current），则包含current的条件将会被忽视
	- Request variables
      ```lisp
	   ;// 查找一个元素，与当前元素位于统一垂直线上，且位置低于100，且x不等于y,  返回符合条件中位置最高的元素
         +visual-location>
			   screen-x current
			   screen-x &x
			   > screen-y 100
			   screen-y lowest
			   - screen-y &x
		```
	- The <span style="color:red"> :nearest </span> request parameter
	```lisp
       +visual-location>
        	 :nearest current
	```
  - 用到了nearest的条件将会被最后考虑

- ##### Ordered Search
    - 从左到右依次看过去，不会因为finsts的限制而陷入loop
    - That will always be able to find the next word to the right of the currently attended one regardless of how many finsts are available and in use.
   ```lisp
       +visual-location>
         	> screen-x current
        	 screen-x lowest
	```
    - By using the relative constraints along with the :nearest request parameter and the current indicator a variety of ordered search strategies can be implemented in a model.

#### 2. The Sperling Task
___
  - for study iconic visual memory
    - https://study.com/academy/lesson/iconic-memory-sperlings-partial-report-experiment.html
  - In the Sperling experiment subjects are briefly presented with a set of letters and must try to report them. Subjects see displays of 12 letters.
  - 闪过一个画面，该画面仅维持50ms，实验者被要求回忆某一行的字母, 在原始的任务中，需要被回忆的那一行被用不同频率的声音读出来)
  - Original one: used a tone with a different frequency for each row and the model will hear simulated tones while it is doing the task.  the display was only presented for 50 ms.
  - ACT-R で実行：
  	- <img src="{{site.baseurl}}/assets/figs/post-20-11-03/demo.gif" width="500px">


#### 3. Visual Attention
___
  - Buffer Stuffing
    - 一开始，模型会自动填充 visual-location buffer ，default是画面最左且最新的元素  :attended new and screen-x lowest. leftmost new item on the screen.
    - 作用： Using buffer stuffing allows the model to detect changes to the screen automatically. 自动监测画面更新
    - buffer stuffing will only occur if the buffer is empty.
    - it occurs for both visual and aural percepts
    - a simple approximation of a bottom-up mechanism of attention.
    - "NIL" :  this setting of the chunk in the buffer was not the result of a production’s request.
     >0.000 VISION SET-BUFFER-CHUNK VISUAL-LOCATION VISUAL-LOCATION1 NIL
  - Testing and Requesting Locations with Slot Modifiers
    - strict harvesting :  automatically clear buffers, otherwise it is modified on RHS

#### 4. Auditory Attention
___
- 2 buffer
	- aural-location :  holds the location of an aural message
	- aural  : holds the representation of a sound attented
- Only 2 steps to encode a sound ( in visual there is 3 )
	- There is a delay between the initial onset of the sound and the audio-event for that sound (in visual it is immediately)
	- depend on the type of sound : tone, digit, word, ...
	- tone : 0.5s
- the following production responds to the appearance of an audio-event in the aural-location buffer:
	-
	```lisp
	 (p detect-sound
	 	=aural-location>    ;// test if there is a chunck in the aural-location buffer
	 	?aural>
	 		state free
	 ==>
	 	+aural>        ;// request to shift attention and encode the event provided
	 		event  =aural-location  
	 )
	```
- it taks <span style="color:orange">285ms</span> from  when the request was made until the sound was attended and encoded

  - chunk type of the sound has slots :  kind, content, event
  	- kind : indicate type of sound (digit, word, customized type)
  	- content : representation of the sound heard
      	- for tone : frequency
      	- for words : strings
      	- for digits : number
  	- event : contains a chunk that relates to the event that was used to attend the sound
 -
	```lisp
		 (p sound-respond-low
		 	=goal>
		 		isa    read-letters
		 		tone   nil
		 	=aural>
		 		isa   sound
		 		content 500    ;// a 500 Hz sound is considered low
		 ==>
		 	=goal>
		 		tone low
		 		upper-y 500
		 		lower-y 520
		 )
	```



#### 5.  Typing and Control
- goal module creates a new chunk immediately in response to a request (imaginal module takes time to create new chunk)
	-
```lisp
(p start-report
	 	=goal>
	 		isa read-letters
	 		tone =tone
	 	>visual>
	 		state free
	 ==>
	 	+goal>    ;// create a new chunk to be placed into the goal buffer
	 		isa report-row
	 		row =tone
	 	+retrieval>
	 		status =tone
		)
	```
- only want this production to apply when there is nothing else to do !
- make the production with a low utility
- set production parameter (spp)
	- (spp start-report :u -2)  
	- the default utility is 0
- Let the sound to be processed as soon as possible
	- (spp detected-sound :u 10)
	- (spp sound-respond-low :u 10)
	- (spp sound-respond-high :u 10)
	- (spp sound-respond-middium :u 10)
	```lisp
	...
	==>
	 	+mannual>
	 		cmd  press-key
	 		key  =val
	```
___
#### 6. Declarative Finsts
___
  - For a model to exhaustively search DM for items without repeating
    - use finst   :recently-retrived
    - default value
      - number of finst : 4
      - finst span : 3s
    - change the default value:
	  - sgp command =  set general paramter
      - (sgp :v t <span style="color:red"> :declearative-finst-span</span>10)   
      - (sgp :v nil <span style="color:red"> :declearative-num-finsts</span> 5)   not to print out traces

#### 7. Data Fitting
___
- 取消随机种子，运行多次得到模拟数据
- (sgp <span style="color:red">:seed</span> (100,0))
- improve speed of runing the model
- turn off the trace
	<span style="color:red">:v</span> nil
- run in a virtual window instead of having a real window to watch
- compare with human data
- metric : number of items correctly recalled
- conditions :  0s,  0.15s,  0.3s , 1.0s  (delay of sound stimuli)
- result:
<img src="{{site.baseurl}}/assets/figs/post-20-11-03/result.png" width="500px">
