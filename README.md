# A Path Towards Autonomous Machine Intelligence

## Overview
### JEPA
In this paper, Yann LeCun proposes how he thinks we should reach machine intelligence.

LeCun's proposal is centered around JEPA. As stated by LeCun, "JEPA models learn high-level representations that capture the dependencies between two data points, such as two segments of video that follow each other. JEPA replaces contrastive learning with “regularized” techniques that can extract high-level latent features from the input and discard irrelevant information."

### Main contributions
As stated in the paper, the main contributions of this paper include:
1) An overall cognitive architecture in which all modules are differntiable and many of them are trainable
2) JEPA and Hierarchical JEPA: a non-generative architecture for predictive worl models that learn a hierarchy of representations
3) A non cntrastive self-supervised learning paradigm that produces represenations that are simultaneously informative and predictable
4) A way to use H-JEPA as the basis of predictive world models for hierarchical planning
under uncertainty

### Entire proposed architecture
![image](https://user-images.githubusercontent.com/89123268/197662144-059230a4-1e00-4099-baca-19c705bcd818.png)

- The world module is the center piece of the model that predicts the state of the world forward in time
- The actor module interacts with the world model and is what performs the action; the actor can also act inside of the world module in essentially a simulated reality to plan forward or it could interact with the world model to find the best action
- the short-term memory is going to be used to train the world model and the  the "critic" (things that happen in the world that will be stored in the short-term memory)
- The perception module takes whatever the world gives it and makes it available as a representation or as a perception
- The configurator is the "master module" that configures all the other modules depending on what situation they're in

### World Module
#### Mode-1 perception-action episode
![image](https://user-images.githubusercontent.com/89123268/197663582-2a003ca0-d0b4-4c2b-b8d3-8e83c6f2fa29.png)

- Mode 1 is reactive; you simply go from perception of the world to action without much thought
- We start with the world and get an observation x then put it through the encoder Enc (which is also the perception module) which will give us a latent representation
- Different paths emerge, although only one is important: the path that goes to the actor A -> the actor then sends back an action to the world
- The other path leads to the cost module, C, which tells us the cost of something -> we can compute it, however, in this basic loop, the actor has already been trained to act on a perception 
- At inference time, the actor doesn't need to look at the cost anymore in order to act

#### Mode-2 perception-action episode
![image](https://user-images.githubusercontent.com/89123268/197664794-04755acb-a672-4d89-a40d-c140d5303072.png)

- Again, we have an input x and put it through the encoder Enc -> however, now, we are going to roll out the world module across different time steps
- The actor a is going to take the state it recieves from the encoder and propse an action (this is the same actor as before) -> we can then use this and put it back into the world module along with the latent prediction
- The predictor Pred takes whatever comes out of the encoder; in other words, it takes a latent state of the world and it predicts the next latent state of the world (hence, this is why LeCun calls this "non-generative")
- We can now give the actor the representation; if it proposes an action, we can use the world module to predict the next state 
- From the next state, we can ask the actor for an action -> the actor gives us an action and we can predict the next state
- We can optimize all of these actions at inference time using gradient descent -> that is, we can improve the actions, a, using gradient descent, through all of the modules until we have completely optimized the action sequence -> which means, the very first action, a[0], is hopefully a much better action than the action first proposed by the naive actor -> we can then take this action and feed it back to the world as an action

**you can include the costs, C, which you can have after every step... thees don't have to be optimization; they can also be search, evolutionary search, tree search, etc. -> essentially, anything that actually tries to improve the action sequence at inference time**

#### Training a reactive module from the result of Mode-2 reasoning
![image](https://user-images.githubusercontent.com/89123268/197669875-7bcd3678-2f48-439f-b735-27e56dca2df8.png)

- Actions, a, are what we have come up with through this optimization process
- Everything's differentiable, so you can train the actor to essentially match the best actions because if you have a good world model, you can improve the low level actor, A(s[0]), and at some point, the initial action sequence will already be close to optimal

### Self-Supervised Learning (SSL)
![image](https://user-images.githubusercontent.com/89123268/197671011-21bc1626-a787-4680-9c28-6e841951909c.png)

- You have a piece of data (which is x + y) and you mask out the right hand side (y) and then you use what you do know (x) and you try to predict the thing you don't know
- You don't want to predict the thing you don't know but instead, create an energy function -> an energy function tells you how well x and y fit together
- You want to train a system that sees the data space in this format, which is going to be an energy landscape
- we don't need to predict y from x directly but instead train the energy function -> the energy function could assign a low value to any possible continuation as long as it assigns a high value everywhere else

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

**so, how do we design the loss to prevent collapse? -> there are two approaches: constrastive methods and regularized methods**

### Constrative Methods Versus Regularized Methods
![image](https://user-images.githubusercontent.com/89123268/197679370-f6278ad9-4a4b-421a-87e9-3151550593c0.png)

#### Contrastive Methods
- many self supervised image training procedures are contrastive -> they'll have an image -> they are going to make 2 variations of that image ->  then, they'll take a third image from the database and will make a variation of that -> then, they use embedding models to embed all of the variations -> this will give a data point somewhere in high dimensional space
- you then try to pull the 2 variations from the same image together and push the variations from different images apart
- contrastive training relies on you coming up with these negative samples
- this quickly runs into problems (curse of dimensionality) so this whole approach of finding training examples/negative examples around a training example to do the contrastive training gets less and less tenable the higher the dimensions
- therefore LeCun advertises for something different: regularized methods


#### Regularized Methods
- have other means of restricting space that is a low energy region
- you enforce/encourage the system to keep the region where the energy is low very small -> this is done through regularization

### The JEPA Architecture


## Question 1

## Question 2

## Architecture Overview (pseudocode)

## Critical Analysis

## Resource LInks

## Video

## References
