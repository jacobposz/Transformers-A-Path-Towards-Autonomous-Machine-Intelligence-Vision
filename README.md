# A Path Towards Autonomous Machine Intelligence

## Overview
### JEPA
In this paper, LeCun proposes how he thinks we should reach machine intelligence.

LeCun's proposal is centered around JEPA. As stated by LeCun, "JEPA models learn high-level representations that capture the dependencies between two data points, such as two segments of video that follow each other. JEPA replaces contrastive learning with “regularized” techniques that can extract high-level latent features from the input and discard irrelevant information."

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

#### Mode-2 perception-action episode
![image](https://user-images.githubusercontent.com/89123268/197664794-04755acb-a672-4d89-a40d-c140d5303072.png)

- again, we have an input (x) and put it through the encoder (Enc(x)) -> however, now, we are going to roll out the world module across different time steps
- the actor (a[0]) is going to take the state it recieves from  the encoder and propse an action (this is the same actor as before) -> we can then use this and put it back into the world module along with the latent prediction
- the predictor (Pred(s,a)) takes whatever comes out of the encoder; in other words, it takes a latent state of the world and it predicts the next latent state of the world (hence, this is why LeCun calls this "non-generative")...it doesn't predict the world, but instead predicts the latent state of the world which enables it to focus on whats truely important for the task
- we can now give the actor the representation; if it proposes an action, we can use the world module to predict the next state 
- from the next state, we can ask the actor for an action -> the actor gives us an action and we can predict the next state
- let's assume that episodes are always the same length (C(s[T])) and you won't get any reward until the very end -> we can compute the reward which is fine (we could already do this before), however, once the entire loop is finished, if all of these things are differentiable, we can say that the action sequence will give us a reward -> can we make this reward bigger? -> since evrything's differentiable, we can certainly use back propogation and gradient descent to ask how the action needs to change in order to make C(s[T]) increase
- we can modify/optimize all of these actions at inference time using gradient descent -> gradient descent is used here to improve the initial action sequence to a more optimal set of actions -> we can improve the actions, a, using gradient descent, through all of the modules until we have completely optimized the action sequence -> which means, the very first action, a[0], is hopefully a much better action than the action first proposed by the naive actor -> we can then take this action and feed it back to the world as an action

*** this is essentially the module thinking about the future and figuring out through forward looking, what it need to do/change to improve the outcome
*** you can include the costs, C, which you can have after every step... thes don't have to be optimization; they can also be search, evolutionary search, tree search, etc. -> essentially, anything that actually tries to improve the action sequence at inference time

#### Training a reactive module from the result of Mode-2 reasoning
![image](https://user-images.githubusercontent.com/89123268/197669875-7bcd3678-2f48-439f-b735-27e56dca2df8.png)

- actions, a, are the ones we have come up with through this optimization process
- you can ask the actor, A, (or take the output from the initial actor) and then try to make these things as close as possible
- everything's differentiable, so you can train the actor to essentially match the better actions because if you have a good world model, you can improve the low level actor, A, and at some point, the initial action sequence that it proposes will already be close to optimal

### Self-Supervised Learning (SSL)
![image](https://user-images.githubusercontent.com/89123268/197671011-21bc1626-a787-4680-9c28-6e841951909c.png)

- you have a piece of data (entire block) and you mask out the right hand side (y) and then you use what you do know (x) and you try to predict the thing you don't know
- you don't want to predict the thing you don't know but instead, create an energy function -> an energy function tells you how well x and y fit together
- you want to train a system that sees the data space in this format, which is going to be an energy landscape
- imagine the block as a video sequence with a bunch of frames -> if you have an energy landscape, you're trying to relate the start of a video sequence to the end of a video sequence
- the system that you train should assign a very low energy to all of the video sequences that are realistic
- we don't need to predict y from x directly because there could be multiple video sequences following the same beginning -> if we were to just predict y, we would probably train the system to identify only one correct continuation -> however, if we train the energy function, the energy function could assign a low value to any possible continuation as long as it assigns a high value everywhere else
- LeCun stresses that an energy function is something you minimize at inference time while the training loss is something that you minimize at training time

### Latent-Variable Energy-Based Model (LVEBM)
![image](https://user-images.githubusercontent.com/89123268/197673225-3f3edf6a-bf39-4d55-8fe4-05902a096a0d.png)

- same formula as before; we have an x and a y and an energy function, Ew that tells us how well those two are compatible with each other which is Fw
- however, as we've seen, there could be many y that are possible for a given x
- just by looking at x, we can't tell whcih of the y's is compatible -> thats why we introduce a latent variable, z -> z captures all of the information about y that isnt directly in x... for example, if a car has the option of turning left or rightm this would be represented by z
- if we have an x and a y, in order to compute that energy that tells us how well the two are compatible, we need to minimize over z
- there is always a hidden variable in energy-based models -> the hidden variable captures everything that isnt captured in x about y -> we minimize over that latent variable to get the actual energy which means, we're looking for the value of the latent variable that makes x and y most compatible
- if we already know that x and y are compatible with one another, then minimizing over z (if we have a good energy function) could actually tell us something about the latent structure of the world -> so, we could infer z or if we have this model trained then if we have an x, we could actually sample some z values in order to produce different possibilities of y -> this gives us a lot of freedom to handle uncertainty in the world or unobserved structure in the world

### Architectures and their capacity for collapse
![image](https://user-images.githubusercontent.com/89123268/197675195-c54360d8-3b9f-415c-b303-d92e374308e5.png)

#### a) Prediction / regression - No Collapse
- D represents the energy/compatibility function
- what if we have a deterministic encoder that gives us the latent representation of x
- then, we use a predictor module in order to predict y
- we'll predict y directly then compare it with the true y -> we have a loss in between them
- this cannot collapse becasue we need to predict the actual y

#### b) Generative latent-variable Architecture - Can Collapse
- again, we compute the representation for x, but now we introduce z that can vary over a certain domain, which gives us a domain that we can control for the output of the predictor Pred
- if we now try to predict y from z and x, we can set z equal to y and we'd be good -> so, this can collapse

#### c) Auto-Encoder - Can Collapse
- this is the same as the first architecture, except only y go into the encoder, instead of both x and y
- after going through the encoder, y gets a latent representation, goes through a decoder that gives you back an estimation of oneself -> so, this can collapse

#### d) Joint Embedding Architecture - Can Collapse
- we have an encoder for x and an encoder for y (these could be the same but don't have to be) -> this will give us 2 latent representations -> then, we use an energy function to compute how well these 2 latent representations fit together, possibly with the help of a latent variable
- if the encoders always output constant vectors and the constant vectors are the same for both x and y, we'll always be good -> however, this can collapse if they are different

*** so, how do we design the loss to prevent collapse? -> there are two approaches: constrastive methods and regularized methods

### Constrative Methods Versus Regularized Methods
![image](https://user-images.githubusercontent.com/89123268/197679370-f6278ad9-4a4b-421a-87e9-3151550593c0.png)

#### Contrastive Methods



#### Regularized Methods


## Question 1

## Question 2

## Architecture Overview (pseudocode)

## Critical Analysis

## Resource LInks

## Video

## References
