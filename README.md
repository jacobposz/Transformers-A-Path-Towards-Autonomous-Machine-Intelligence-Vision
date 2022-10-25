# A Path Towards Autonomous Machine Intelligence

## Overview
### JEPA
In this paper, LeCun proposes how he thinks we should reach machine intelligence.

LeCun's proposal is centered around JEPA; a new architecture that LeCun proposes. As states by LeCun, "JEPA models learn high-level representations that capture the dependencies between two data points, such as two segments of video that follow each other. JEPA replaces contrastive learning with “regularized” techniques that can extract high-level latent features from the input and discard irrelevant information."

### Main contributions
As stated in the paper, the main contributions of this paper include:
1) an overall cognitive architecture in which all modules are differntiable and many of them are trainable
2) JEPA and Hierarchical JEPA: a non-generative architecture for predictive worl models that learn a hierarchy of representations
3) a non cntrastive self-supervised learning paradigm that produces represenations that are simultaneously informative and predictable
4) A way to use H-JEPA as the basis of predictive world models for hierarchical planning
under uncertainty
 
***H denotes hierarchical arrangement of the jepa architecture

### Entire proposed architecture
![image](https://user-images.githubusercontent.com/89123268/197662144-059230a4-1e00-4099-baca-19c705bcd818.png)

- The world module is the center piece of the model that predicts the state of the world forward in time
- The actor module is going to interact with the world model and is what will ultimately be what does the action; the actor can also act inside of the world module in essentially a simulated reality to plan forward (what would happen if I were to do something?) or it could interact with the world model to find the best action
- the short-term memory is going to be used to train the world model and the  the "critic" (things that happen in the world that will be stored in the short-term memory)
- perception module takes whatever the world gives it and makes it available as a representation or as a perception; this is essentially the entry point to the systems that we have now. This is the closest module we have to something that's actually working which is our current deep learning systems; they're very good at perception.
- the configurator is the "master module" that configures all the other modules depending on what situation they're in

### World Module
#### Mode-1 perception-action episode
![image](https://user-images.githubusercontent.com/89123268/197663582-2a003ca0-d0b4-4c2b-b8d3-8e83c6f2fa29.png)

- Mode 1 is reactive; you simply go from perception of the world to action without much thought
- we start with the world and get an observation (x) then put this through the encoder (Enc(x)) (which is the perception module) which will give us a latent representation
- different paths emerge, although only one is important: the path that goes to the actor (A(s)) -> the actor then sends back an action to the world
- the other path leads to the cost module which tells us the cost of something (this can be good or bad, intrinsic motivation or external reward, etc.) -> we can compute it, however, in this basic loop, the actor has been trained already to just act on a perception 
- at inference time, the actor doesn't need to look at the cost anymore in order to act








## Question 1

## Question 2

## Architecture Overview (pseudocode)

## Critical Analysis

## Resource LInks

## Video

## References
