---
layout: post
title:  "ACT-R Assignment Unit8"
date:   2020-12-16 22:36:14 +0800
categories: 認知科学
tags: ACTR_Assignment ACT-R
---
#### 顔分類タスク
___
  - a simplification of an experiment which was performed by Robert M. Nosofsky
  - **The experiment**
    - trained on learning 10 faces (5 of each categories)
    - varied along:
      - eye height     (EH)
      - eye seperation   (ES)
      - nose length      (NL)
      - mouth height     (MH)
    - testing phase
      - new faces
      - old faces
  - **The model:**
    - Not involve any learning mechanism
    - > training information pre-encoded in declarative memory,  model reset on every trial
    - 流れ：
      - presented with the attributes one at a time
        - name-value
      - collect those attributes into a single chunk (encoded in the imaginal buffer)
      - using the chunk to retrieve a best matching in the DM
      - based on the chunk, make a category choice for the current stimuli
    - Hint
      - first thing to do is to creating stimulus representation from the individual attributes
      - no more than 5 productions
      - use procedual partial matching
      - use dynamic pattern matching
  -  **The Stimulus Attributes**
    - Goal buffer in the initial state
      - CHUNK1-0
        - NAME EH
        - VALUE 0.7
        - STATE ADD-ATTRIBUTE
    - We have to convert the numeric number 0.7 into a symbolic description stored in a slot of the **imaginal** buffer
        - small
        - medium
        - large
    - use procedual partial matching and similarity hook to convert numeric attributes t label attributes

  - **Model Response**
    - Goal state set to CATEGORIZE
      - CHUNK5-0
        - STATE CATEGORIZE
    - Retrieving an example in the DM that is similar to the current stimulus in the imaginal buffer
      - but the slot names were unkown
      - using an indirect request with the chunk from the imaginal buffer :
      - ?  +retrieval> =imaginal

  - **Running the experiment**
    - Test functionality of adding atrributes
      - > (categorize-attribute size -.9)
      - result in a chunk created in the imaginal buffer
        - CHUNK1-0
          - CATEGORY UNKNOWN
          - SIZE             SMALL
      - the attribute name can be any string
    - Test functionality of retrieving using dynamic pattern matching
      - > (categorize-stimulus 1 -1 -1 -1)

  - **Fitting the data**
    - > (categorize-experiment 10)
    - reference result if the model is built correctly
      - CORRELATION:  0.948
      - MEAN DEVIATION:  0.174
    - paramters than can be tuned
      - (sgp :mp 1     # declarative partical matching
      - :ppm 1           #  procedual  partical  matching
      - :egs .25         #  noise for utility
      - :ans .25)        #  noise for activation
    - After paramter tuning:  (n = 500)
      - CORRELATION:  0.948
      - MEAN DEVIATION:  0.142

  - **Generality**
    - To test the generability:
      - change similarity scores
        - > (categorize-experiment 100 4.5)
        - offset  /  shift  = 4.5
      - also change attribute names
        - > (categorize-experiment 100 -3 a b c d)
    - the model should be unaffected by that change and produce the same fit to the data regardless of the number provided for the shift and the name provided for the attributes
    - > Changing the attribute names and the offset should not affect the model’s ability to do the task if it has been written to perform the task generally.

#### Model
___
```lisp
(clear-all)

 (define-model categorize
   (sgp :v nil :act t)
   (sgp :esc t :lf .01 :blc 10 :rt -100 :do-not-harvest imaginal)
   (sgp :sim-hook "size-similarities" :cache-sim-hook-results t)
   (sgp :mp 1 :ppm 1 :egs .25 :ans .25)

   (chunk-type goal state name value)
   (chunk-type example category)

   (add-dm (small isa chunk)
           (medium isa chunk)
           (large isa chunk)
           (unknown isa chunk))

   (define-chunks (categorize) (add-attribute))

   (call-act-r-command "create-example-memories")

   (set-similarities (small medium -.4)
                     (medium large -.4)
                     (small large -.8))


   (set-buffer-chunk 'imaginal (first (define-chunks (category unknown))))

   ;; Declare the goal buffer chunk-type usage which is going to be set from
   ;; outside of the model.

   (declare-buffer-usage goal goal :all)

   (p add-information-small
     =goal>
       isa goal
       state add-attribute
       value small
       name  =n
     =imaginal>
   ==>
     =imaginal>
       category unknown
       =n   small
     =goal>
       state add-attribute
       value nil
       name  nil
     )

   (p add-information-medium
     =goal>
       isa goal
       state add-attribute
       value medium
       name  =n
     =imaginal>
   ==>
     =imaginal>
       category unknown
       =n   medium
     =goal>
       state add-attribute
       value nil
       name  nil
     )

   (p add-information-large
     =goal>
       isa goal
       state add-attribute
       value large
       name  =n
     =imaginal>
   ==>
     =imaginal>
       category unknown
       =n   large
     =goal>
       state add-attribute
       value nil
       name  nil
     )

   (p find-example
     =goal>
       isa goal
       state categorize
     =imaginal>
     ?retrieval>
       state  free
       buffer empty
   ==>
     +retrieval> =imaginal
   )

   (p done
     =goal>
       isa goal
       state categorize
     =imaginal>
     =retrieval>
       category =val
   ==>
     =imaginal>
       category =val
     =goal>
       isa goal
       state nil
   )


  )
  ```
- key notes
	- attending imaginal slots:
  ```lisp
   (p example
		     ...
		     =imaginal>              ; do not query
     ==>
       =imaginal>              ; do not use +!!!
         category unknown
         =n   medium
    ```

#### Results
___
- with default parameter
	- (sgp      :mp 1      :ppm 1         :egs .25         :ans .25)     
	- <img src="{{site.baseurl}}/assets/figs/post-20-12-16/result1.png" width="500px">
	- Test Generality with a shift
	- <img src="{{site.baseurl}}/assets/figs/post-20-12-16/result2.png" width="500px">
	- Test Generality with different attributes name
	- <img src="{{site.baseurl}}/assets/figs/post-20-12-16/result3.png" width="500px">
