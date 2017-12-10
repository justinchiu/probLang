---
layout: chapter
title: Referring Expressions and Contextual Accessibility
description: "RE and CA"
---

#### Semantic Theory
The puzzle: given a wide array of possible descriptions for any given referent,
how do speakers choose what to say?
Given an utterance with ambiguous reference, how does a listener disambiguate to select the speaker’s intended referent?
In a specific context (e.g. Obama giving a State of the Union address), all of the following are referentially equivalent,
at least in terms of their literal truth:

> The 44th President of the United States of America = The President = 
Barack Obama = the man standing at the podium = that man = he

Accessibility Theory (Ariel 1988, 2001) predicts that all of these utterances encode different levels of accessibility of the referent.
Specifically, we are more likely to use highly specific, full definite NPs as descriptions
(e.g. The 44th President of the United States of America) when the intended referent is very difficult to access,
and pronominal forms (e.g. he) when the referent is highly accessible.

The posited hierarchy, from low-accessibility markers to high, is:

> Definite NPs < Demonstratives < Pronouns < Null

What constitutes accessibility? A complex array of properties:
* Frequency of reference
* Distance since last reference
* Number of competing referents
* Discourse topic status
* Extralinguistic salience
* Structural and thematic positions

Accessibility provides a natural way to disambiguate reference.
Von Heusinger (2000) argues that accessibility serves as a choice function that assigns to a set one of its elements.
This theory, and other experimental data makes certain predictions:
* Speakers should refer efficiently, using the least specific and informative utterances for the most predictable referents
(Tily & Piantadosi 2009)
* The presence of similar entities to the intended referent in the context creates interference,
lowering predictability and pushing speakers towards more specific utterances (Fukumura et al 2011)
* Entities with the most recent instances of reference should attract the lowest-accessibility-marking utterances (Ariel 1988):

![Alt text](https://github.com/justinchiu/probLang/raw/master/images/ling_table.png)

Following Fukumura (2012), our model assumes that speakers select referents based on their own model of accessibility for a given context,
and listeners must pragmatically infer the speaker’s thresholds for using specific accessibility-marking utterances.

#### Threshold Model

##### Setup

We model the state as a list of entities that are of either male or female gender, and only have one other
identifying property: their height. For our model we assume that
every possible combination of gender and height is enumerated in the state.

The possible utterances are then the pronouns "null", "he", and "she"; the demonstratives
"that girl", "that boy"; and the definite descriptions "the tall girl", "the short girl", etc.

~~~~
// Possible properties.
var genders = ["male", "female"]
var heights = ["short", "tall"]
var accesses = [1,2,3]

// Example entity.
var entity = {
  gender: "female",
  height: "tall",
  access: 3
}

// Example state.
var state = [
  {
    gender: "male",
    height: "short",
    access: 1
  },
  {
    gender: "male",
    height: "tall",
    access: 1
  },
  {
    gender: "female",
    height: "short",
    access: 2
  },
  {
    gender: "female",
    height: "tall",
    access: 3
  }
]

// Possible utterances
var utterances = ['null','he', 'she', 'that boy', 'that girl', 'the tall boy',
                 'the short boy', 'the tall girl', 'the short girl']
~~~~

##### Priors

The utterance prior is engineered as an inverted word count model
so as to avoid specifying a separate utility function.
Shorter utterances, such as the pronouns, receive higher mass than longer utterances.
This is highly analgous to coding or compression schemes which seek to minimize total message length.

For our state prior, since we would like to have a fully enumerated state, we can sample
accessibility levels for each of the possible entities.

~~~~
// Possible utterances
var utterances = ['null','he', 'she', 'that boy', 'that girl', 'the tall boy',
                 'the short boy', 'the tall girl', 'the short girl']
                 
// Essentially an inverted word count model.
var utterancePrior = function(){
  var ps = [4,3,3,2,2,1,1,1,1]
  var n = 1
  return categorical(map(function(x){x+n}, ps), utterances)
}

var statePrior = function(state) {
  return map(function(x){
    var access = uniformDraw([1,2,3])
    return {
      gender: x.gender,
      height: x.height,
      access: access,
    }
  }, state)
}
~~~~

At this point, our setup looks a bit like Frank and Goodman's original RSA model for the blue circle
reference game. We have a state where entities share properties, and thus some referring expressions may
be ambiguous. For example, similar to how we would disambiguate "blue" to refer to the blue circle or square,
how can we resolve the pronoun "he" if the state contains two males of different heights?

To do so we introduce two accessibility thresholds, one each for the demonstrative and pronoun referring expressions.
Intuitively, we assume that either referring expression
is used if an entity's accessibility is greater than or equal to the 
corresponding threshold, with the pronoun taking precedence over the demonstrative.
If the entity's access is lower than both thresholds, we default to the definite description.

This is similar to the Lassiter and Goodman model for gradable adjectives.
Analogous to how a threshold on cost was communicated by the statement "expensive",
a referring expression marks whether an entity’s accessibility exceeds a threshold.
Since we have multiple thresholds, the analogous setup for kettles would be to have one threshold for
"expensive" and one for "very expensive".
The main difference between our model and the Lassiter and Goodman model
is that the comparison class for an object (in the case of 
the gradable adjectives model, this would be the cost of a kettle) would be all possible objects (costs of kettles)
in the adjectives model. 
In our setting the accessibility of an entity must only compete with other existing entities in the state. 

We also introduce a soft meaning function:
~~~~
var softMeanings = function(utterance, referent, theta_dem, theta_pro){
  return utterance == "null" ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'he' && referent.gender == 'male' ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'she' && referent.gender == 'female' ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'that boy' && referent.gender == 'male' ?
    flip(referent.access >= theta_dem ? 0.99 : 0.01) :
  utterance == 'that girl' && referent.gender == 'female' ?
    flip(referent.access >= theta_dem ? 0.99 : 0.01) :
  utterance == 'the tall boy' ?
    (referent.gender == 'male' && referent.height == 'tall') :
  utterance == 'the short boy' ?
    (referent.gender == 'male' && referent.height == 'short') :
  utterance == 'the tall girl' ?
    (referent.gender == 'female' && referent.height == 'tall') :
  utterance == 'the short girl' ?
    (referent.gender == 'female' && referent.height == 'short') :
  false
};
~~~~
We assume that the definite description is true as long as the referent satisfies its property requirements.
We opt to interpret the thresholded utterances probabilistically, since thresholds are not necessarily hard constraints.
Consider the example that the only male entity in the state has not been mentioned for a long time, 
but someone refers to him as "he". This would be awkward for us to resolve, but not impossible.

We now present the threshold model in its entirety.
One interesting aspect is that every stage of the model sees the "state".
This is different from other models we have seen, but is really only a result of variable naming.
Since the referent is only seen the speakers, we are not violating any RSA contracts.

~~~~
///fold:
// Utility Functions

var printList = function(xs) {map(function(x){
  print(x)
}, xs)}

var swapElems = function(i, j, xs) {
  var val = xs[i]
  xs.splice(i, 1, xs[j])
  xs.splice(j, 1, val)
  return xs
}

var randomInt = function(lower, upper) {
  var int = randomInteger(upper-lower+1)
  return int+lower
}

var knuthShuffle = function(xs) {
  var len = xs.length
  var rands = mapN(function(n){return [n, randomInteger(n+1)]}, len)
  var ok = reduce(function(x, acc){
    var i = x[0]
    var j = x[1]
    return swapElems(i, j, acc)
  }, xs, rands)
  return ok
}

// Possible properties.
var genders = ["male", "female"]
var heights = ["short", "tall"]
var accesses = [1,2,3]

// A scoring function for sorting.
var score = function(x) {
  var g = x.gender === "female" ? 100 : 200
  var h = x.height === "short" ? 10 : 20
  var a = x.access
  return g + h + a
}

// A fully enumerated state.
var fullState = [].concat.apply([], map(function(gender){
  [].concat.apply([], map(function(height){
    [].concat.apply([], map(function(access){
      return {
        gender: gender,
        height: height,
        access: access
      }
    }, accesses))
  }, heights))
}, genders))
// Sort using score function.
var fullState = sort(fullState, lt, score)

// A partial state.
var state = [
  {
    gender: "male",
    height: "short",
    access: 1
  },
  {
    gender: "male",
    height: "tall",
    access: 1
  },
  {
    gender: "female",
    height: "short",
    access: 1
  },

  {
    gender: "female",
    height: "tall",
    access: 3
  },
]

// Sample n random entities from the fully enumerated state
// without replacement
var statePrior = function(n, state) {
  var shuffled = knuthShuffle(state)
  return sort(shuffled.slice(0, n), lt, score)
}

// Shadow partial state with fullState.
var state = fullState

// Possible utterances
var utterances = ['null','he', 'she', 'that boy', 'that girl', 'the tall boy',
                 'the short boy', 'the tall girl', 'the short girl']

var utterancePrior = function(){
  var ps = [4,3,3,2,2,1,1,1,1]
  var n = 1
  return categorical(map(function(x){x+n}, ps), utterances)
};

var softMeanings = function(utterance, referent, theta_dem, theta_pro){
  return utterance == "null" ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'he' && referent.gender == 'male' ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'she' && referent.gender == 'female' ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'that boy' && referent.gender == 'male' ?
    flip(referent.access >= theta_dem ? 0.99 : 0.01) :
  utterance == 'that girl' && referent.gender == 'female' ?
    flip(referent.access >= theta_dem ? 0.99 : 0.01) :
  utterance == 'the tall boy' ?
    (referent.gender == 'male' && referent.height == 'tall') :
  utterance == 'the short boy' ?
    (referent.gender == 'male' && referent.height == 'short') :
  utterance == 'the tall girl' ?
    (referent.gender == 'female' && referent.height == 'tall') :
  utterance == 'the short girl' ?
    (referent.gender == 'female' && referent.height == 'short') :
  false
};

///

// Theta prior.
var thetaPrior = function(){
  return uniformDraw([1,2,3])
};

// L0
// p(ref=ent | utterance=utt, state, thetas) = [utt](ent, thetas) * p(ref=ent)
var literalListener = function(utterance,state, theta_dem, theta_pro){
  return Infer({model: function(){
    var ref = uniformDraw(state);
    condition(softMeanings(utterance,ref,theta_dem,theta_pro))

    return ref
  }})
};

// optimality
var alpha = 1

// S1
// p(utterance=utt | ref=ent, state, thetas; l0) 
//     = p(ref=ent | utterance=utt, state, thetas; l0) * p(utterance=utt)
var speaker = function(state, referent, theta_dem, theta_pro){
  Infer({model: function(){
    var utterance = utterancePrior()
    factor(alpha * literalListener(utterance,state,theta_dem,theta_pro).score(referent))
    return utterance
  }})
};

// L1
// p(ref=ent | utterance=utt, state, thetas; s1)
//     = p(utterance=utt | ref=ent, state, thetas; s1) * p(ref=ent) * p(thetas)
var pragmaticListener = function(utterance,state){
  Infer({model: function(){
    var referent = uniformDraw(state);
    var theta_dem = thetaPrior()
    var theta_pro = thetaPrior()
    observe(speaker(state,referent,theta_dem,theta_pro), utterance)
    return referent
  }})
};

// S2
// p(utterance=utt | ref=ent, state; l1)
//     = p(ref=ent | utterance=utt, state, l1) * p(utterance=utt)
var pragmaticSpeaker = function(state, referent) {
  Infer({model: function() {
    var utterance = utterancePrior()
    factor(alpha * pragmaticListener(utterance, state).score(referent))
    return utterance
  }})
}

~~~~

#### A simple case

Let's consider a small state:
~~~~
///fold:
// Utility Functions

var printList = function(xs) {map(function(x){
  print(x)
}, xs)}

var swapElems = function(i, j, xs) {
  var val = xs[i]
  xs.splice(i, 1, xs[j])
  xs.splice(j, 1, val)
  return xs
}

var randomInt = function(lower, upper) {
  var int = randomInteger(upper-lower+1)
  return int+lower
}

var knuthShuffle = function(xs) {
  var len = xs.length
  var rands = mapN(function(n){return [n, randomInteger(n+1)]}, len)
  var ok = reduce(function(x, acc){
    var i = x[0]
    var j = x[1]
    return swapElems(i, j, acc)
  }, xs, rands)
  return ok
}

// Possible properties.
var genders = ["male", "female"]
var heights = ["short", "tall"]
var accesses = [1,2,3]

// A scoring function for sorting.
var score = function(x) {
  var g = x.gender === "female" ? 100 : 200
  var h = x.height === "short" ? 10 : 20
  var a = x.access
  return g + h + a
}

// A fully enumerated state.
var fullState = [].concat.apply([], map(function(gender){
  [].concat.apply([], map(function(height){
    [].concat.apply([], map(function(access){
      return {
        gender: gender,
        height: height,
        access: access
      }
    }, accesses))
  }, heights))
}, genders))
// Sort using score function.
var fullState = sort(fullState, lt, score)

// Sample n random entities from the fully enumerated state
// without replacement
var statePrior = function(n, state) {
  var shuffled = knuthShuffle(state)
  return sort(shuffled.slice(0, n), lt, score)
}

// Shadow partial state with fullState.
var state = fullState

// Possible utterances
var utterances = ['null','he', 'she', 'that boy', 'that girl', 'the tall boy',
                 'the short boy', 'the tall girl', 'the short girl']

var utterancePrior = function(){
  var ps = [4,3,3,2,2,1,1,1,1]
  var n = 1
  return categorical(map(function(x){x+n}, ps), utterances)
};

var softMeanings = function(utterance, referent, theta_dem, theta_pro){
  return utterance == "null" ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'he' && referent.gender == 'male' ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'she' && referent.gender == 'female' ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'that boy' && referent.gender == 'male' ?
    flip(referent.access >= theta_dem ? 0.99 : 0.01) :
  utterance == 'that girl' && referent.gender == 'female' ?
    flip(referent.access >= theta_dem ? 0.99 : 0.01) :
  utterance == 'the tall boy' ?
    (referent.gender == 'male' && referent.height == 'tall') :
  utterance == 'the short boy' ?
    (referent.gender == 'male' && referent.height == 'short') :
  utterance == 'the tall girl' ?
    (referent.gender == 'female' && referent.height == 'tall') :
  utterance == 'the short girl' ?
    (referent.gender == 'female' && referent.height == 'short') :
  false
};

// Theta prior.
var thetaPrior = function(){
  return uniformDraw([1,2,3])
};

// L0
// p(ref=ent | utterance=utt, state) = [utt](ent) * p(ref=ent)
var literalListener = function(utterance,state, theta_dem, theta_pro){
  return Infer({model: function(){
    var ref = uniformDraw(state);
    condition(softMeanings(utterance,ref,theta_dem,theta_pro))

    return ref
  }})
};

// optimality
var alpha = 1

// S1
// p(utterance=utt | ref=ent, state; l0) 
//     = p(ref=ent | utterance=utt, state; l0) * p(utterance=utt)
var speaker = function(state, referent, theta_dem, theta_pro){
  Infer({model: function(){
    var utterance = utterancePrior()
    factor(alpha * literalListener(utterance,state,theta_dem,theta_pro).score(referent))
    return utterance
  }})
};

// L1
// p(ref=ent | utterance=utt, state; s1)
//     = p(utterance=utt | ref=ent, state; s1) * p(ref=ent)
var pragmaticListener = function(utterance,state){
  Infer({model: function(){
    var referent = uniformDraw(state);
    var theta_dem = thetaPrior()
    var theta_pro = thetaPrior()
    observe(speaker(state,referent,theta_dem,theta_pro), utterance)
    return referent
  }})
};

// S2
// p(utterance=utt | ref=ent, state; l1)
//     = p(ref=ent | utterance=utt, state, l1) * p(utterance=utt)
var pragmaticSpeaker = function(state, referent) {
  Infer({model: function() {
    var utterance = utterancePrior()
    factor(alpha * pragmaticListener(utterance, state).score(referent))
    return utterance
  }})
}

///

// A simple state.
var state = [
  {
    gender: "male",
    height: "short",
    access: 1
  },
  {
    gender: "male",
    height: "tall",
    access: 2
  },
  {
    gender: "female",
    height: "short",
    access: 2
  },

  {
    gender: "female",
    height: "tall",
    access: 3
  },
]

print("State")
printList(state)

print("\n")
print("L1")
var printer = map(function(utt) {
  print("when hearing '" + utt + "'")
  viz.table(pragmaticListener(utt, state))
}, utterances)

print("\n")
print("S2")
var printer = map(function(ref) {
    print("referring to")
    print(ref)
    viz.table(pragmaticSpeaker(state, ref))
}, state)
~~~~

Note that utterance distributions of the low access male and low access female have the same ordering over REs,
even though they have different accessibilities. This is due to the
model partitioning the mass of utterances over semantically compatible entities.

Given that the model performs inference over relative accessibilities with respect to
entities that share properties, you might expect that the highest access
entities in each class might have a similar utterance distribution.
However, this is not the case, as the "null" utterance treats the entire state
as its comparison class.


#### Fully enumerated state

How does this model do?

~~~~
///fold:
// Utility Functions

var printList = function(xs) {map(function(x){
  print(x)
}, xs)}

var swapElems = function(i, j, xs) {
  var val = xs[i]
  xs.splice(i, 1, xs[j])
  xs.splice(j, 1, val)
  return xs
}

var randomInt = function(lower, upper) {
  var int = randomInteger(upper-lower+1)
  return int+lower
}

var knuthShuffle = function(xs) {
  var len = xs.length
  var rands = mapN(function(n){return [n, randomInteger(n+1)]}, len)
  var ok = reduce(function(x, acc){
    var i = x[0]
    var j = x[1]
    return swapElems(i, j, acc)
  }, xs, rands)
  return ok
}

// Possible properties.
var genders = ["male", "female"]
var heights = ["short", "tall"]
var accesses = [1,2,3]

// A scoring function for sorting.
var score = function(x) {
  var g = x.gender === "female" ? 100 : 200
  var h = x.height === "short" ? 10 : 20
  var a = x.access
  return g + h + a
}

// A fully enumerated state.
var fullState = [].concat.apply([], map(function(gender){
  [].concat.apply([], map(function(height){
    [].concat.apply([], map(function(access){
      return {
        gender: gender,
        height: height,
        access: access
      }
    }, accesses))
  }, heights))
}, genders))
// Sort using score function.
var fullState = sort(fullState, lt, score)

// Sample n random entities from the fully enumerated state
// without replacement
var statePrior = function(n, state) {
  var shuffled = knuthShuffle(state)
  return sort(shuffled.slice(0, n), lt, score)
}

// Shadow partial state with fullState.
var state = fullState

// Possible utterances
var utterances = ['null','he', 'she', 'that boy', 'that girl', 'the tall boy',
                 'the short boy', 'the tall girl', 'the short girl']

var utterancePrior = function(){
  var ps = [4,3,3,2,2,1,1,1,1]
  var n = 1
  return categorical(map(function(x){x+n}, ps), utterances)
};

var softMeanings = function(utterance, referent, theta_dem, theta_pro){
  return utterance == "null" ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'he' && referent.gender == 'male' ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'she' && referent.gender == 'female' ?
    flip(referent.access >= theta_pro ? 0.99 : 0.01) :
  utterance == 'that boy' && referent.gender == 'male' ?
    flip(referent.access >= theta_dem ? 0.99 : 0.01) :
  utterance == 'that girl' && referent.gender == 'female' ?
    flip(referent.access >= theta_dem ? 0.99 : 0.01) :
  utterance == 'the tall boy' ?
    (referent.gender == 'male' && referent.height == 'tall') :
  utterance == 'the short boy' ?
    (referent.gender == 'male' && referent.height == 'short') :
  utterance == 'the tall girl' ?
    (referent.gender == 'female' && referent.height == 'tall') :
  utterance == 'the short girl' ?
    (referent.gender == 'female' && referent.height == 'short') :
  false
};

// Theta prior.
var thetaPrior = function(){
  return uniformDraw([1,2,3])
};

// L0
// p(ref=ent | utterance=utt, state) = [utt](ent) * p(ref=ent)
var literalListener = function(utterance,state, theta_dem, theta_pro){
  return Infer({model: function(){
    var ref = uniformDraw(state);
    condition(softMeanings(utterance,ref,theta_dem,theta_pro))

    return ref
  }})
};

// optimality
var alpha = 1

// S1
// p(utterance=utt | ref=ent, state; l0) 
//     = p(ref=ent | utterance=utt, state; l0) * p(utterance=utt)
var speaker = function(state, referent, theta_dem, theta_pro){
  Infer({model: function(){
    var utterance = utterancePrior()
    factor(alpha * literalListener(utterance,state,theta_dem,theta_pro).score(referent))
    return utterance
  }})
};

// L1
// p(ref=ent | utterance=utt, state; s1)
//     = p(utterance=utt | ref=ent, state; s1) * p(ref=ent)
var pragmaticListener = function(utterance,state){
  Infer({model: function(){
    var referent = uniformDraw(state);
    var theta_dem = thetaPrior()
    var theta_pro = thetaPrior()
    observe(speaker(state,referent,theta_dem,theta_pro), utterance)
    return referent
  }})
};

// S2
// p(utterance=utt | ref=ent, state; l1)
//     = p(ref=ent | utterance=utt, state, l1) * p(utterance=utt)
var pragmaticSpeaker = function(state, referent) {
  Infer({model: function() {
    var utterance = utterancePrior()
    factor(alpha * pragmaticListener(utterance, state).score(referent))
    return utterance
  }})
}

///

print("\n")
print("S2")
var printer = map(function(ref) {
    print("referring to")
    print(ref)
    viz.table(pragmaticSpeaker(state, ref))
}, state)
~~~~

![Alt text](https://github.com/justinchiu/probLang/raw/master/images/ling_table2.png)

#### Future Work
Future work should pursue:
* Other properties of the demonstrative, e.g. introduction
* Differences between definite NPs, i.e. finer-grained cost
* Breaking down the accessibility variable and taking into account non-contextual properties, like structural position and thematic role (Fukumura 2009)
* Dynamically updating the accessibility of referents, e.g. definite NPs as accessibility functions (von Heusinger 2000)

