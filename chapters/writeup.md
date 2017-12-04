---
layout: chapter
title: References
description: "References"
---

### References

Introduction

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

The posited hierarchy, from high-accessibility markers to low, is:

> Definite NPs > Demonstratives > Pronouns > Null

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

#### Simple Model

Let us first consider a simplified model where there are no lifted threshold variables.

~~~~
// Possible utterances
var utterances = ['null','he', 'she', 'that boy', 'that girl', 'the tall boy',
                 'the short boy', 'the tall girl', 'the short girl']

var utterancePrior = function(){
  var ps = [4,3,3,2,2,1,1,1,1]
  var n = 1
  return categorical(map(function(x){x+n}, ps), utterances)
}

var state0 = [
  {
    gender: "male",
    height: "tall",
  },
  {
    gender: "male",
    height: "short",
  },
  {
    gender: "female",
    height: "tall",
  },
  {
    gender: "female",
    height: "short",
  },
]

// Add an access value to each entity in the above state.
var statePrior = function() {
  return map(function(x){
    var access = uniformDraw([1,2,3])
    return {
      gender: x.gender,
      height: x.height,
      access: access,
    }
  }, state0)
}

// Extract a referent from a state.
var refPrior = function(state) {
  var n = 0
  var ps = map(function(x){return x["access"] + n}, state)
  var ref = categorical(ps, state)
  return ref
}
~~~~

#### Section 3
Now recall the very first model we've seen in this class from Frank and Goodman, 2014.
We have a fully enumerated state and a non-probabilistic interpretation function.
How does their exact model perform on this data?

~~~~
///fold:
// Possible utterances
var utterances = ['null','he', 'she', 'that boy', 'that girl', 'the tall boy',
                 'the short boy', 'the tall girl', 'the short girl']

var utterancePrior = function(){
  var ps = [4,3,3,2,2,1,1,1,1]
  var n = 1
  return categorical(map(function(x){x+n}, ps), utterances)
}

var state0 = [
  {
    gender: "male",
    height: "tall",
  },
  {
    gender: "male",
    height: "short",
  },
  {
    gender: "female",
    height: "tall",
  },
  {
    gender: "female",
    height: "short",
  },
]

// Add an access value to each entity in the above state.
var statePrior = function() {
  return map(function(x){
    var access = uniformDraw([1,2,3])
    return {
      gender: x.gender,
      height: x.height,
      access: access,
    }
  }, state0)
}

// Extract a referent from a state.
var refPrior = function(state) {
  var n = 0
  var ps = map(function(x){return x["access"] + n}, state)
  var ref = categorical(ps, state)
  return ref
}
///

// Literal meaning function.
var literalMeanings = function(utterance, referent){
  return (utterance == 'he' || utterance == 'that boy') ?
    referent.gender == 'male' :
  (utterance == 'she' || utterance == 'that girl') ?
    referent.gender == 'female' :
  utterance == 'the tall boy' ?
    (referent.gender == 'male' && referent.height == 'tall') :
  utterance == 'the short boy' ?
    (referent.gender == 'male' && referent.height == 'short') :
  utterance == 'the tall girl' ?
    (referent.gender == 'female' && referent.height == 'tall') :
  utterance == 'the short girl' ?
    (referent.gender == 'female' && referent.height == 'short') :
  true
}

// L0
// p(ref=ent | utterance=utt, state) = [utt](ent) * p(ref=ent)
var literalListener = function(utterance,state){
  return Infer({model: function(){
    var ref = refPrior(state)
    condition(literalMeanings(utterance,ref))
    return ref
  }})
}

// optimality
var alpha = 1

// S1
// p(utterance=utt | ref=ent, state; l0) 
//     = p(ref=ent | utterance=utt, state; l0) * p(utterance=utt)
var speaker = function(state, referent){
  Infer({model: function(){
    var utterance = utterancePrior()
    factor(alpha * literalListener(utterance,state).score(referent))
    return utterance
  }})
}

// L1
// p(ref=ent | utterance=utt, state; s1)
//     = p(utterance=utt | ref=ent, state; s1) * p(ref=ent)
var pragmaticListener = function(utterance,state){
  Infer({model: function(){
    var referent = refPrior(state);
    observe(speaker(state,referent), utterance)
    return referent
  }})
}

var state = statePrior()

print("State")
var printer = map(function(ent) {
  print(ent)
}, state)

print("\n")
print("S1")
var printer = map(function(ref) {
    print("referring to")
    print(ref)
    viz.table(speaker(state, ref))
}, state)

print("\n")
print("L1")
var printer = map(function(utt) {
  print("when hearing '" + utt + "'")
  viz.table(pragmaticListener(utt, state))
}, utterances)
~~~~

#### Section 4

What happens if we add an S2?

~~~~
///fold:
// Possible utterances
var utterances = ['null','he', 'she', 'that boy', 'that girl', 'the tall boy',
                 'the short boy', 'the tall girl', 'the short girl']

var utterancePrior = function(){
  var ps = [4,3,3,2,2,1,1,1,1]
  var n = 1
  return categorical(map(function(x){x+n}, ps), utterances)
}

var state0 = [
  {
    gender: "male",
    height: "tall",
  },
  {
    gender: "male",
    height: "short",
  },
  {
    gender: "female",
    height: "tall",
  },
  {
    gender: "female",
    height: "short",
  },
]

// Add an access value to each entity in the above state.
var statePrior = function() {
  return map(function(x){
    var access = uniformDraw([1,2,3])
    return {
      gender: x.gender,
      height: x.height,
      access: access,
    }
  }, state0)
}

// Extract a referent from a state.
var refPrior = function(state) {
  var n = 0
  var ps = map(function(x){return x["access"] + n}, state)
  var ref = categorical(ps, state)
  return ref
}

// Literal meaning function.
var literalMeanings = function(utterance, referent){
  return (utterance == 'he' || utterance == 'that boy') ?
    referent.gender == 'male' :
  (utterance == 'she' || utterance == 'that girl') ?
    referent.gender == 'female' :
  utterance == 'the tall boy' ?
    (referent.gender == 'male' && referent.height == 'tall') :
  utterance == 'the short boy' ?
    (referent.gender == 'male' && referent.height == 'short') :
  utterance == 'the tall girl' ?
    (referent.gender == 'female' && referent.height == 'tall') :
  utterance == 'the short girl' ?
    (referent.gender == 'female' && referent.height == 'short') :
  true
}

// L0
// p(ref=ent | utterance=utt, state) = [utt](ent) * p(ref=ent)
var literalListener = function(utterance,state){
  return Infer({model: function(){
    var ref = refPrior(state)
    condition(literalMeanings(utterance,ref))
    return ref
  }})
}

// optimality
var alpha = 1

// S1
// p(utterance=utt | ref=ent, state; l0) 
//     = p(ref=ent | utterance=utt, state; l0) * p(utterance=utt)
var speaker = function(state, referent){
  Infer({model: function(){
    var utterance = utterancePrior()
    factor(alpha * literalListener(utterance,state).score(referent))
    return utterance
  }})
}

// L1
// p(ref=ent | utterance=utt, state; s1)
//     = p(utterance=utt | ref=ent, state; s1) * p(ref=ent)
var pragmaticListener = function(utterance,state){
  Infer({model: function(){
    var referent = refPrior(state);
    observe(speaker(state,referent), utterance)
    return referent
  }})
}

var state = statePrior()

///
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

print("\n")
print("S1")
var printer = map(function(ref) {
  print("referring to")
  print(ref)
  viz.table(speaker(state, ref))
}, state)

print("\n")
print("S2")
var printer = map(function(ref) {
  print("referring to")
  print(ref)
  viz.table(pragmaticSpeaker(state, ref))
}, state)
~~~~

#### Section 5

Interestingly enough, the utterance predictions are almost exactly the same.

#### Section 6: Threshold Model

Since there is no improvement with an additional S2, we instead consider a reformulation 
of the model. 

We introduce two accessibility thresholds. One each for the demonstrative and pronoun referring expressions.
Intuitively, we assume that either referring expression
is used if an entity's latent access variable is greater than or equal to the 
corresponding threshold, with the pronoun RE taking precedence over the demonstrative.
If the entity's access is lower than both thresholds, we default to the definite description.

This is indeed similar to the Tessler and Goodman model of generics in spirit.
A referring expression is “endorsed” if an entity’s access exceeds a threshold.
However, the main difference is that there are multiple thresholds which have an ordering according to preference. 
Rather than just endorsing a referring expression in isolation,
the speaker must pick the expression that has the highest preference as well as appropriate threshold according to the entity’s accessibility.

We introduce a softer meaning function in order to prevent all paths from having zero mass
during inference. This comes up, for example, in the following situation:

~~~~
var utterance = "he"
var state = [
  {
    gender: "female",
    height: "short",
    access : 1
  }
]
~~~~

#### Section 7:

The solution is simple, albeit slightly hacky:

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

#### Section 8:

We now present the threshold model in its entirety.

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

~~~~

#### Section 9: Fully enumerated state

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

#### Section 10: Examining a special case

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

#### Future Work
Future work should pursue:
* Other properties of the demonstrative, e.g. introduction
* Differences between definite NPs, i.e. finer-grained cost
* Breaking down the accessibility variable and taking into account non-contextual properties, like structural position and thematic role (Fukumura 2009)
* Dynamically updating the accessibility of referents, e.g. definite NPs as accessibility functions (von Heusinger 2000)

