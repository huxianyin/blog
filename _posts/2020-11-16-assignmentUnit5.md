---
layout: post
title:  "ACT-R Assignment Unit5"
date:   2020-11-16 22:36:14 +0800
categories: 認知科学
tags: ACTR_Assignment ACT-R
---
#### Blackjack
<img src="{{site.baseurl}}/assets/figs/post-20-11-14/blackjack.gif" width="500px">


#### Task description
____




#### Model
___
```lisp
(clear-all)

(define-model 1-hit-model

  ;; do not change these parameters
  (sgp :esc t :bll .5 :ol t :sim-hook "1hit-bj-number-sims" :cache-sim-hook-results t :er t :lf 0)

  ;; adjust these as needed
  (sgp :v nil :ans .2 :mp 15.0 :rt -5)

  ;; This type holds all the game info

  (chunk-type game-state mc1 mc2 mc3 mstart mtot mresult oc1 oc2 oc3 ostart otot oresult state)

  ;; This chunk-type should be modified to contain the information needed
  ;; for your model's learning strategy

  (chunk-type learned-info MSTART OSTART action)

  ;; Declare the slots used for the goal buffer since it is
  ;; not set in the model defintion or by the productions.
  ;; See the experiment code text for more details.

  (declare-buffer-usage goal game-state :all)

  ;; Create chunks for the items used in the slots for the game
  ;; information and state

  (define-chunks win lose bust retrieving start results)

  ;; Provide a keyboard for the model's motor module to use
  (install-device '("motor" "keyboard"))

  (p start
     =goal>
       isa game-state
       state start
       MSTART =c
       OSTART =o
    ==>
     =goal>
       state retrieving
     +retrieval>
       isa learned-info
       MSTART =c
       OSTART =o
     - action nil)

  (p cant-remember-game-hit
     =goal>
       isa game-state
       state retrieving
     ?retrieval>
       buffer  failure
     ?manual>
       state free
    ==>
     =goal>
       state nil
     !output! ("not remember and hit")
     +manual>
       cmd press-key
       key "h")


  (p cant-remember-game-stay
     =goal>
       isa game-state
       state retrieving
     ?retrieval>
       buffer  failure
     ?manual>
       state free
    ==>
     =goal>
       state nil
     !output! ("not remember and stay")
     +manual>
       cmd press-key
       key "s")

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
     !output! (I =act)
     +manual>
       cmd press-key
       key =act

     @retrieval>)


  (p results-should-hit-if-win
     =goal>
       isa game-state
       state results
       mresult win
       MSTART =c
       OSTART =o
     - MC3 nil
     ?imaginal>
       state free
    ==>
     ;!output! (I =outcome)
     =goal>
       state nil
     +imaginal>
       MSTART =c
       OSTART =o
       action "h")


(p results-should-stay-if-win
     =goal>
       isa game-state
       state results
       mresult win
       MSTART =c
       OSTART =o
       MC3 nil
     ?imaginal>
       state free
    ==>
     ;!output! (I =outcome)
     =goal>
       state nil
     +imaginal>
       MSTART =c
       OSTART =o
       action "s")

(p results-no-act-if-lose
     =goal>
       isa game-state
       state results
       mresult lose
     ?imaginal>
       state   free
    ==>
     ;!output! (I =outcome)
     =goal>
       state nil
	   +imaginal>
       MSTART nil
       OSTART nil
       action nil)


  (p results-should-stay-if-bust
     =goal>
       isa game-state
       state results
       mresult bust
       MSTART =c
       OSTART =o
     ?imaginal>
       state free
    ==>
     ;!output! (I =outcome)
     =goal>
       state nil
     +imaginal>
       MSTART =c
       OSTART =o
       action "s")

  (p results-should-stay-if-over-eighteen
     =goal>
       isa game-state
       state results
       ;mresult bust
       > MSTART 18
       MSTART =c
       OSTART =o
     ?imaginal>
       state free
    ==>
     ;!output! (I =outcome)
     =goal>
       state nil
     +imaginal>
       MSTART =c
       OSTART =o
       action "s")


  (p results-should-hit-if-less-twelve
     =goal>
       isa game-state
       state results
       ;mresult bust
       < MSTART 12
       MSTART =c
       OSTART =o
     ?imaginal>
       state free
    ==>
     ;!output! (I =outcome)
     =goal>
       state nil
     +imaginal>
       MSTART =c
       OSTART =o
       action "h")

  (spp results-should-stay-if-over-eighteen :u 10)
  (spp results-should-hit-if-less-twelve :u 10)


  (p clear-new-imaginal-chunk
     ?imaginal>
       state free
       buffer full
     ==>
     -imaginal>)
  )
```

#### Strategy Desicription
___
- if m_start < 12 ,  learn to hit
- if m_start > 18,   learn to stay
- learn any chunks that won
- do not learn when burst or lose

#### Results
___
- > ? (onehit-hands 5)
  - (3 2 0 0)
- > ? (onehit-blocks 2 5)
  - ((1 4 0 0) (2 3 0 0))
- > ? (onehit-learning 100)
  - ((0.3616 0.39159998 0.4048 0.3828) (0.328 0.338 0.39000002 0.35999998 0.39200002 0.398 0.35999998 0.384 0.41 0.406 0.41799998 0.398 0.42800003 0.378 0.402 0.366 0.41799998 0.354 0.394 0.382))

  - Original Results:
    - <img src="{{site.baseurl}}/assets/figs/post-20-11-16/original.png" width="500px">

  - Hu's strategy implemented 		
    - <img src="{{site.baseurl}}/assets/figs/post-20-11-16/hu.png" width="500px">
