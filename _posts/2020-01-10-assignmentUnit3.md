---
layout: post
title:  "ACT-R Assignment Unit3"
date:   2020-01-10 00:43:16 +0800
categories: 認知科学
tags: ACTR_Assignment ACT-R
---
<img src="{{site.baseurl}}/assets/figs/post-20-01-10/result.png" width="300px">


#### Task
___
- The task code: tutorial/lisp/subitize.lisp
- count "x"


#### Model (Hu's solution)
___
```lisp
(clear-all)

(define-model subitize

(sgp :v nil)

(sgp :randomize-time t)

(sgp :show-focus t
     :visual-num-finsts 10
     :visual-finst-span 10)


(chunk-type count count state)
(chunk-type number-fact identity next value)
(chunk-type report count)

(add-dm (one isa chunk)(two isa chunk)
        (three isa chunk)(four isa chunk)
        (five isa chunk)(six isa chunk)
        (seven isa chunk)(eight isa chunk)
        (nine isa chunk)(ten isa chunk)
        (zero isa chunk) (eleven isa chunk)
        (start isa chunk)

        ;for speak
        (n0 isa number-fact identity zero next one value "zero")
        (n1 isa number-fact identity one next two value "one")
        (n2 isa number-fact identity two next three value "two")
        (n3 isa number-fact identity three next four value "three")
        (n4 isa number-fact identity four next five value "four")
        (n5 isa number-fact identity five next six value "five")
        (n6 isa number-fact identity six next seven value "six")
        (n7 isa number-fact identity seven next eight value "seven")
        (n8 isa number-fact identity eight next nine value "eight")
        (n9 isa number-fact identity nine next ten value "nine")
        (n10 isa number-fact identity ten next eleven value "ten")
        (goal isa count state start count zero))


        ;for press key
        ;(n0 isa number-fact identity zero next one value 0)
        ;(n1 isa number-fact identity one next two value 1)
        ;(n2 isa number-fact identity two next three value 2)
        ;(n3 isa number-fact identity three next four value 3)
        ;(n4 isa number-fact identity four next five value 4)
        ;(n5 isa number-fact identity five next six value 5)
        ;(n6 isa number-fact identity six next seven value 6)
        ;(n7 isa number-fact identity seven next eight value 7)
        ;(n8 isa number-fact identity eight next nine value 8)
        ;(n9 isa number-fact identity nine next ten value 9)
        ;(n10 isa number-fact identity ten next eleven value 0)
        ;(goal isa count state start count zero))


(goal-focus goal)




(p attend-letter
    =goal>
        isa        count
        state      start
        count      =num
    =visual-location>
        isa        visual-location
   ?visual>
        state      free
==>
    =goal>
        state      encoding
   +visual>
        cmd        move-attention
        screen-pos =visual-location
    +retrieval>
        identity =num
)


(p encode-count
    =goal>
         isa    count
         state encoding
         count  =num
    =retrieval>
        identity =num
        next =num_next
    =visual>              ;visual 一定要有东西才能 encode并且将注意力转向下一个目标 如果没有这一条，将会重复计数
==>
    =goal>
        isa count
        state start
        count =num_next
    ;-visual>   不影响结果
    +visual-location>
        :attended nil
        ;:nearest current  不影响结果
)

(p start-report
    =goal>
        isa count
        count =num
    ?visual-location>
        buffer  failure
==>
   =goal>
        isa count
        state report
    +retrieval>
        identity =num
)


(p do-report
    =goal>
        isa count
        state report
        count =num
    =retrieval>
        identity =num
        next =num_next
        value =val
    ?vocal>
        state free
==>
    +vocal>
        cmd speak
        string =val
    -goal>
)
)
```
#### Result
___
- Run the model
```console
? (subitize-experiment)
```
- Fit human data
  - CORRELATION:  0.986
  - MEAN DEVIATION:  0.194

|  Items |  Current Participant | Original Experiment  |
| ---- | ---- |---- |
|  1    |     0.54  (T  )        |       0.60
|    2   |      0.74  (T  )      |       0.65
|    3   |      0.89  (T  )      |       0.70
|    4    |     1.01  (T  )      |       0.86
|    5     |    1.23  (T  )      |       1.12
|    6      |   1.48  (T  )      |       1.50
|    7       |  1.69  (T  )      |       1.79
|    8  |       1.87  (T  )      |       2.13
|    9   |      1.95  (T  )      |       2.15
|  	 10   |      2.16  (T  )     |       2.58
