---
layout: post
title:  "ACT-R Assignment Unit2"
date:   2020-09-22 16:37:13 +0800
categories: 認知科学
tags: ACTR_Assignment ACT-R
---
#### The Assignment in the ACT-R Tutorial Unit2
<img src="{{site.baseurl}}/assets/figs/post-20-09-22/screenshot.png" width="500px">
![tag](https://img.shields.io/badge/buidling-ACT_R7.0-green.svg)

#### Task
  - The task code: tutorial/lisp/unit2.lisp
  - After 3 letters presented, report the differerent one
___
#### Run
```console
   ? (unit2-experiment)
```

#### Model (My Answer)
```lisp
(clear-all)

(define-model unit2

(sgp :v nil :show-focus nil)


(chunk-type read-letters state)
(chunk-type array letter1 letter2 letter3 decision)

(add-dm
 (start isa chunk)
 (attend isa chunk)
 (respond isa chunk)
 (find-next isa chunk)
 (judge isa chunk)
 (done isa chunk)
 (goal isa read-letters state start))


(P find-unattended-letter
   =goal>
      ISA         read-letters
      state       start
   ?imaginal>
   	  state       free
 ==>
   +visual-location>
      :attended    nil
   =goal>
      state       find-location
   +imaginal>
   	  isa         array
   	  letter1     nil
   	  letter2     nil
   	  letter3     nil
)


(P find-next-letter
   =goal>
      ISA         read-letters
      state       find-next
 ==>
   +visual-location>
      :attended    nil
   =goal>
      state       find-location
)


(P attend-letter
   =goal>
      ISA         read-letters
      state       find-location
   =visual-location>
   ?visual>
      state       free
==>
   +visual>
      cmd         move-attention
      screen-pos  =visual-location
   =goal>
      state       attend
)

(P encode-letter-first
   =goal>
      ISA         read-letters
      state       attend
   =visual>
      value       =letter
   ?imaginal>
      state       free
   =imaginal>
      isa         array
      letter1     nil
      letter2     nil
      letter3     nil
==>
   =goal>
      state       find-next
   +imaginal>
      isa         array
      letter1      =letter
      letter2     nil
      letter3     nil
)


(P encode-letter-second
   =goal>
      ISA         read-letters
      state       attend
   =visual>
      value       =letter
   ?imaginal>
      state       free
   =imaginal>
      isa         array
   	  letter1     =letter1
   	  letter2     nil
   	  letter3     nil
==>
   =goal>
      state       find-next
   +imaginal>
      isa         array
      letter1      =letter1
      letter2      =letter
      letter3      nil
)

(P encode-letter-final
   =goal>
      ISA         read-letters
      state       attend
   =visual>
      value       =letter
   ?imaginal>
      state       free
   =imaginal>
   	  isa         array
   	  letter1     =letter1
   	  letter2     =letter2
   	  letter3     nil
==>
	=goal>
		state     judge
    +imaginal>
    	isa       array
    	letter1   =letter1
    	letter2   =letter2
    	letter3   =letter
)

(P judge-diff-letter-3
	=goal>
		isa       read-letters
		state     judge
	?imaginal>
		state     free
	=imaginal>
		isa       array
		letter1   =letter1
		letter2   =letter1
		letter3   =letter
==>
	=goal>
		state     respond
	+imaginal>
		isa       array
		decision  =letter
)

(P judge-diff-letter-2
	=goal>
		isa       read-letters
		state     judge
	?imaginal>
		state     free
	=imaginal>
		isa       array
		letter1   =letter1
		letter2   =letter
		letter3   =letter1
==>
	=goal>
		state     respond
	+imaginal>
		isa       array
		decision  =letter
)

(P judge-diff-letter-1
	=goal>
		isa       read-letters
		state     judge
	?imaginal>
		state     free
	=imaginal>
		isa       array
		letter1   =letter
		letter2   =letter1
		letter3   =letter1
==>
	=goal>
		state     respond
	+imaginal>
		isa       array
		decision  =letter
)



(P respond
   =goal>
      ISA         read-letters
      state       respond
   =imaginal>
      isa         array
      decision    =letter
   ?manual>   
      state       free
==>
   =goal>
      state       done
   +manual>
      cmd         press-key
      key         =letter
)

(goal-focus goal)
)
```
