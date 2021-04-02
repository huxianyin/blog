---
layout: post
title:  "ACT-R Tutorial Unit2"
date:   2020-09-21 16:37:13 +0800
categories: 認知科学
tags: ACTR_Tutorial ACT-R
---
#### Perception and Motor Actions
<img src="{{site.baseurl}}/assets/figs/post-20-09-21/banner.png" width="500px">

#### Imaginal Module
---
- All requests to the imaginal module through the imaginal buffer are requests to create a new chunck.
- The Amount of time it takes to create a chunk   =   <span style="color:red">0.2</span> sec
    ```lisp
      (p encode-letter
         ...
         ?imaginal>          ; //query imaginal to verify if it is available
           state free
       ==>
         ...
         +imaginal>             ;// send request to imaginal to create a new chunk
           isa array
           letter =letter
	```
#### Vision Module
---
- can parse text, lines, button features from the displays created.
- tow buffer :
 - **visual-location**
 ```lisp
     (p find-location
       ...
     ==>
       +visual-location>  
         :attended nil  ;// request for a location which the model has not attended
	   :attended t    ;// request for a location which has been attended previously
         :attended new    ;// the model has not attended to the location and the object has also recently appeared in the visual scene
       ...
     )
  ```
 - **visual**
  ```lisp
     (p attend-letter
       =goal>
         ...
       =visual-location>   ;// if there is a chunk in the visual-location buffer
       ?visual>            ;// query visual buffer if it is available for a new request
         state free
     ==>
      +visual>     ;// move attention to the location specified in visual-location
         cmd         move-attention
         screen-pos  =visual-location
       =goal>
         ...
  ```
- for visual-location buffer of vision module, there is no time involved in handling the request (in such case, no need to query), reflects the assumption that there is a perceptual system operating in parallel within the vision module.
- the state of vision module for the visual-location buffer is <span style="color:red">always free</span>.

####  Motor Module
---
