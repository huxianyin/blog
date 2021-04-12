---
layout: post
title:  "ACT-R Assignment Unit7"
date:   2020-12-06 22:36:14 +0800
categories: 認知科学
tags: ACTR_Assignment ACT-R
---
#### 英語過去式を生成する認知モデル
- <img src="{{site.baseurl}}/assets/figs/post-20-12-06/word.png" width="500px">

#### Task
___
- Generate English Past Tense
- **Hint**
	- 最终，model学习到2种类型的production：
		- General: 看到一个动词，在动词后面+ed
		- Specific: 看到 have, 回忆出 had
	- The key to a successful model is to implement two different retrieval strategies
	- the productions you write should have no explicit reference to either of the suffixes, ed or blank
- Reasonable
	- verb 不变
	- stem = verb /  corret irregular form
	- nerver create irregular form for a regular verb
- <span style="color:red"> Use only one strategy at a trial </span>
	- reason : language generation is a rapid process and not something for which a lot of time per word can be allocated

- **task implemented**
	- past-tense.lisp
	- past_tense.py  

- **Initialization**
    - add correct past tenses to DM (很多动词的正确过去式的知识）
    - creates goal to indicate that should generate past tense of verb found in the **imaginal** buffer

#### Knowledge Representation
___
   - Past Tense Verbs:
   - <img src="{{site.baseurl}}/assets/figs/post-20-12-06/word.png" width="500px">
   - Example:
	- Use
		- verb = use,  stem = use, suffix = ed
	- Have
		- verb = have, stem = had, suffix = blank

#### How to run the model
___
 - goal has a state of start
 - imaginal buffer with verb slot filled with a verb
 - The model then has to fill in the ![image](* stem) and **suffix** slots of the chunk in the imaginal buffer
 - goal has a state of done
 - use the verb with 3 different productions (each has different reward)
   - An irregular inflection  
     - stem is not null,   suffix is blank
     - reward: high
   - An regular inflection
     - stem is equal to verb, suffix is not null and is not blank
     - reward : middum
   - Non-inflected
     - neither the stem nor suffix slots are set in the chunk.
     - reward : low

#### Useful Cmmand
___
   - > (past-tense-trials 5000 nil nil)
   - optional parameter1 :
     - t or nil
     - whether or not you want the model to continue from where it left off
   - optional parameter2 :
     - t or nil
     - if show the trace
   - return :  results of last 100 verbs generated
     - proportion correct of irregular verbs  (不规则形式正确回忆概率，包含回忆失败的）
     - proportion of irregular verbs that are inflected regularly（不规则形式加ed）
     - proportion of irregular verbs that are not inflected at all.  （没回忆起的不规则动词）
     - proportion of inflected irregular verbs that are inflected correctly （回忆起来的不规则动词中正确回忆的概率）
   - See graphcal results:
     - > (past-tense-results)

#### Model
___

```lisp
(clear-all)

(define-model past-tense
   (sgp
     :V t
     :esc T
     :ul t
     :OL 6   
     :BLL 0.5  
     :ANs 0.1   
     :EGs 0.2   
     :mas   3.5
     :imaginal-activation 1.0
     :LF 0.5
     :RT 0.5
     :epl t
     :alpha 0.1
     :pct t
     :iu 5    
     :ncnar nil
     :do-not-harvest imaginal)

  (chunk-type past-tense verb stem suffix)
  (chunk-type goal state)

  (define-chunks
      (starting-goal isa goal state start)
      (start isa chunk) (blank isa chunk) (done isa chunk) (wait-retrieve isa chunk)
      (wait-analogy isa chunk)
  )


  (declare-buffer-usage goal goal state)
  (declare-buffer-usage imaginal past-tense :all)

(p retrieve-past-tense
  =goal>
    isa goal
    state start
  =imaginal>
    isa past-tense
    verb =word
    stem nil
    suffix nil
==>
  +retrieval>
    isa past-tense
    verb =word
  =goal>
    isa goal
    state wait-retrieve
)

(p analogy
  =goal>
    isa goal
    state start
  =imaginal>
    isa past-tense
    verb =word
    suffix nil
    stem   nil
==>
  +retrieval>
    isa past-tense
    - verb nil
    - stem nil
    - suffix nil
  =goal>
    isa goal
    state wait-analogy
)

(p retrieve-success
  =goal>
    isa goal
    state wait-retrieve
  ?imaginal>
    state free
  =imaginal>
    isa past-tense
    verb =word
  =retrieval>
    isa past-tense
    suffix =su
    stem   =st
==>
  +imaginal>
    isa past-tense
    verb   =word
    suffix =su
    stem   =st
  =goal>
    isa goal
    state done
)


(p analogy-success-add-suffix
  =goal>
    isa goal
    state wait-analogy
  ?imaginal>
    state free
  =imaginal>
    isa past-tense
    verb =word
  =retrieval>
    isa past-tense
    verb =a
    stem =a
    suffix =s
==>
  +imaginal>
    isa past-tense
    verb   =word
    suffix =s
    stem   =word
  =goal>
    isa goal
    state done
)


(p analogy-success-giveup
  =goal>
    isa goal
    state wait-analogy
  =imaginal>
    verb =word
  ?imaginal>
    state free
  =retrieval>
    isa past-tense
    suffix =s
    verb =a
    stem =b
    !safe-eval! (not (eq =a =b));(a!=b)
     !safe-eval! (not (eq =a =word));(a!=word)
==>
  =goal>
    isa goal
    state done
  +imaginal>
    isa past-tense
    verb =word
    suffix nil
    stem nil
)

(p analogy-success-match-and-copy
  =goal>
    isa goal
    state wait-analogy
  =imaginal>
    verb =word
  ?imaginal>
    state free
  =retrieval>
    isa past-tense
    suffix =s
    verb =word
    stem =b
==>
  =goal>
    isa goal
    state done
  +imaginal>
    isa past-tense
    verb =word
    suffix =s
    stem =b
)


(p retrieve-fail
  =goal>
    isa goal
    state wait-retrieve
  ?retrieval>
    buffer   failure
  ?imaginal>
    state free
  =imaginal>
==>
  =goal>
    isa goal
    state done
)


(p analogy-fail
  =goal>
    isa goal
    state wait-analogy
  ?retrieval>
    buffer   failure
  ?imaginal>
    state free
  =imaginal>
==>
  =goal>
    isa goal
    state done
)

;;; When there is a stem and no suffix we have an irregular
(p past-tense-irregular
   =goal>
     isa    goal
     state  done
   =imaginal>
     isa    past-tense
     verb   =word
     stem   =past
     suffix blank
   ==>
   =goal>
     state  nil)

(spp past-tense-irregular :reward 5)

;;; When the stem matches the verb and the suffix is not "blank" consider it regular

(p past-tense-regular
   =goal>
    isa     goal
    state   done
   =imaginal>
     isa    past-tense
     verb   =stem
     stem   =stem
     suffix =suffix
   !safe-eval! (not (eq =suffix 'blank))
  ==>
   =goal>
     state  nil)

(spp past-tense-regular :reward 4.2)

;;; when there's no stem and no suffix that's a "give up" result

(p no-past-tense-formed
   =goal>
     isa    goal
     state  done
   =imaginal>
     isa    past-tense
     stem   nil
     suffix nil
  ==>
   =goal>
     state  nil)

(spp no-past-tense-formed :reward 3.9)
)
```
- Strategy 1 : retrieve
	- 根据记忆，回忆过去式
- Strategy2 : analogy
	- 类比记忆中的任意一个单词的过去式，产生过去式
	- 如果  回忆  verb == stem ，则复制suffix, stem保留原样     (production : analogy-success-add-suffix)
	- 如果 回忆   verb != stem  而且 verb不等于word , 则 放弃回忆       (production : analogy-success-giveyp)
	- 如果 回忆   verb != stem  而且 verb等于word , 则复制stem和suffix      (production : analogy-success-match-and-copy)

#### Results
___
<img src="{{site.baseurl}}/assets/figs/post-20-12-06/result1.png" width="300px"><img src="{{site.baseurl}}/assets/figs/post-20-12-06/result2.png" width="300px">
<img src="{{site.baseurl}}/assets/figs/post-20-12-06/result3.png" width="300px"><img src="{{site.baseurl}}/assets/figs/post-20-12-06/result4.png" width="300px">
