---
layout: post
title:  "ACT-R Tutorial Unit1"
date:   2020-03-03 18:53:13 +0800
categories: 認知科学
tags: ACTR_Tutorial ACT-R
---
#### Introduction to ACT-R
<img src="{{site.baseurl}}/assets/figs/post-20-03-03/eg.png" width="500px">
___
#### Contents
------
  - 1.1 Knowledge Representations
    - chuncks
    - productions
------
  - 1.2 The ACT-R Architecture
------
  - 1.3 ACT-R software and Models
    - Programming Language : Common Lisp
------
  - **1.4 Creating an ACT-R Model**
    - control commands
      - clear-all
      - define-model
    - Chunk-type
      - (chunck-type chunck-name slot1 slot2 slot3)
    - Creating chuncks <span style="color:red">add-dm</span>.
      - Example:
        ```lisp
        (add-dm
            (b ISA counter-order first 1 second 2)
            (first-goal ISA count-from start 2 end 4)
        )
        ```
    - Creating productions  <span style="color:red">p</span>.
      - Example 1 :   test condtions and set states of buffers or make retrieval request
      ```lisp
          (p counting-example "example production for counting task"
             =goal>
                 ISA count
                 state incrementing
                 number =num1           ;// "=" means it is a variable
             =retrieval>
                 first =num1
                 second =num2
           ==>                                 ;//  "==>" means take actions
             =goal>
               ISA count
               number =num2
             +retrieval>                    ;// "+" means make request
               ISA count-order
               first =num2
           )
	   ```
      - Example2 : check if request fails
        ```lisp
          (p no-carry "description:个位数加法没有产生进位"
             =goal>
               ISA add-pair
               carry busy
               ten1 =num1
               ten2 =num2
             ?retrieval>             ;// test if retrival failed
               buffer failure
           ==>
             =goal>
               carry nil              ;// eliminate a slot : set it to nil
               ten-ans busy
             +retrieval>
               addend1 =num1
               addend2 =num2
           )
		```
      - Example3 : negative test use " - " in front of a slot on LHS,   clearing buffer use "-" on RHS
        ```lisp
          - (P increment
                 =goal>
                       ISA           count-from
                       count        =num1
                       - end         =num1  ;//  “-” is the negative test modifier.
                 =retrieval>
                       ISA           count-order
                       first          =num1
                       second     =num2
             ==>
                 =goal>
                       ISA          count-from
                       count       =num2
                 +retrieval>
                       ISA           count-order
                       first          =num2
                       !output!    (=num1)   ;// to display information in the trace
          ...
	```
      - Example4 : querying the buffer itself
       ```lisp
            ?retrieval>
               buffer   full ;// if a failure has been noted for the retrieval buffer
             ?retrieval>
			   buffer   failure  ;//if there is not a chunk in the retrieval buffer
             ?retrieval>
			   buffer   empty
             ?retrieval>
			   buffer   requested
             ?retrieval
			   buffer   unrequested
	```
      - Example5: querying the buffer's module
     ```lisp
              ?retrieval>
			      state       free
              ?retrieval>
			      state       busy
              ?retrieval>
			      state       error
				  recently-retrieved t
		```
------
  - 1.5  The count mode
    - setting initial goal :  <span style="color:red">goal-focus</span>.
    - (goal-focus first-goal)
------
  - 1.6 Pattern Matching Exercise
    - "Procedual"
		- Click "Why not" to debug  (or use as a command)
    - "Stepper"
		- Button in the control panel to debug
------
  - 1.7 The addition Model
    - add by count
    - 5 + 2 = 7   -->   5+1 = 6 ,  5+1 = 7
    - update "sum" and "count"
    - until "count" == 2
    - output up-to-date "sum"
------
  - 1.8 The Semantic Model
    - searching the following network to make decisions about whether one category is a member of another category.
  <img src="{{site.baseurl}}/assets/figs/post-20-03-03/eg.png" width="500px">
------
  - 1.9 Building a model following the tutorial code
    - 2-digit addition

___
#### ACT-R プログラムと普通のプログラムの違い
 - The model is not being written as commands for a computer to execute, but as commands for a cognitive processor ( a simulated human mind)

- ACT-R is a very low-level language written to run on a “processor” with many high-level capabilities built into it whereas most languages are a high-level set of operators targeting a very general low-level processor for execution.

- how the sequence of actions is determined.
	- ACT-R :  order is determined based on which one currently matches the current state of the buffers and modules, and that requires satisfying all of the conditions on the LHS(left-hand side) of a production.
	- other programs : execute by the order as written.

    - optimizations and efficient design metrics which are important in normal programming tasks, like efficient algorithms, code reuse, minimal number of steps, etc, are not always good design choices for creating an ACT-R model because such models will not perform “like a person". Instead, one has to consider the task from a human perspective and rely on psychological research and performance data to guide the design of the model.
