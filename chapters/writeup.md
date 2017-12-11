---
layout: chapter
title: Modeling Referring Expression Use in Accessibility Theory
description: "RE and CA"
---

## The Semantic Puzzle

In any given discourse context, the set of entities to which it is possible for a speaker to refer is quite large, including at least all physically present individuals, all those in the epistemic common ground of the interlocutors, and potentially any new referent that a speaker chooses to introduce into that common ground. Compounding this situation, most referring expressions in our lexicon are underspecified in their reference, such that they will often be ambiguous between two or more possible referents in the context, as many of them will share the minimal features for which referring expressions are specified (i.e. their grammatical phi-features, in the case of pronouns). Given these two facts, how does a speaker select which expression to use to signal an intended referent to a listener? Following this, how does a listener, upon hearing an utterance, disambiguate between all the possible referents for which that utterance is literally true, and identify the intended referent of the speaker?

Consider a context, such as a State of the Union address being given by Barack Obama. If you and I are seated next to each other in the audience, and I wish to convey the opinion that the speech contains some very good points or is being delivered in a persuasive style, I might lean over to you and whisper something like “____ is really nailing it tonight.” If we are only considering the literal truth of whether my utterance matches the speaker, all the following options to fill in the blank will be referentially equivalent:

> *The 44th President of the United States of America = The President = Barack Obama = the man standing at the podium = the speaker = that man = he*
 
How do I guarantee that you know I am intending to compliment the speech, and not, for example, the man in charge of catering the event? The simplest solution to this problem would be for the speaker to always choose the maximally restrictive utterance, which will pick out the unique referent in all cases (e.g. a proper name such as *Barack Obama*, or a unique definite description such as *the 44th President of the United States of America*). But such descriptions are not always easily available for less well-known referents, or would require an extremely lengthy utterance (e.g. *the short, bearded man in the corner of the room wearing the plaid jacket and the garish red hat*), which we know from experience that speakers are very disinclined to use unless absolutely necessary, normally after their attempt at a less specified utterance has already failed to signal the listener properly. This reflects a very general tendency in language to opt for more economical forms whenever possible, especially in the context of descriptions and pronouns, taking forms such as Minimize DP constraints (Schlenker 2005, Patel-Grosz & Grosz 2017), and the Avoid Pronoun Principle (Chomsky 1981). This structural economy restrictions impose strong preferences towards minimal pronominal forms, and even null elements such as PRO in the latter case. However, as mentioned above, these economical forms come with much greater ambiguity. Clearly, neither informative specificity nor structural economy dominate our decision calculus, as we do not produce exclusively proper names nor exclusively pronouns, null or otherwise. Instead, we can model the speaker’s decision as an interaction between these inverted cost and informativity parameters.

Returning to the State of the Union example, it seems uncontroversial that the average speaker would be overwhelmingly likely to select the minimal pronoun he in this context, and that the average listener would have no trouble resolving this reference in favor of the president giving the speech, even if there were hundreds of other male-gendered individuals in the room. What licenses this easy inference? Many semantic theories (Heim 1982, Roberts 2004, Ariel 1988) identify the relevant factor with something like salience or accessibility, which roughly correspond to some combination of the linguistic, physical, and psychological prominence of the entity in question. Both due to his physical prominence in the room, his high level of social relevance, and the frequency to which he is referred, Obama would be considered a highly salient and accessible referent and any ambiguous pronoun is likely to be resolved in favor of him. Additionally, given the relative ease of referring to him with a highly economical pronoun, it would be pragmatically rather strange to refer to him with any more structurally complex definite. Some formal semantic theories encode this difference in terms of the presuppositions associated with definite NPs and pronouns. Roberts (2004) is a representative example of this: both pronouns and definite NPs have presuppositions requiring their referent to be weakly familiar (though this can be easily accommodated) and unique in the context, while only pronouns have a salience presupposition. This theory, however, is difficult to implement as a model of predicting speaker behavior. For one, salience is a relatively poorly defined concept, and it provides only a binary way of distinguishing salient from non-salient entities. In a clean-cut case like the State of the Union address we have been discussing, it explains the binary choice between using a pronoun and a structurally larger definite, but when it comes to more complex and gradient states, it will be difficult to make predictions.

Accessibility Theory (Ariel 1985, 1991) provides a more cognitively-focused and computationally implementable approach to speaker choice of referring expressions. Each possible referent is assigned some degree of accessibility, a sort of meta-variable composed of a wide array of linguistic, pragmatic, and extralinguistic properties, likely including at least the following (Ariel 1988):
-    Frequency of reference
-    Distance since last reference
-    Number of competing referents
-    Discourse topic status (local or global)
-    Extralinguistic salience (e.g. visual access)
-    Position in the syntactic structure
-    Thematic role
    
Only together do these properties, and some language-specific structural properties, accurately predict a speaker’s choice in referring expression (Ariel 2001). It is argued to be a distinctly non-binary variable, contrasting with prior work (see Givon 1983), but rather operate over a gradient. For the purposes of easy visualization, take one highly relevant factor, the distance between the referring expression and the last utterance of its antecedent. The table below (reproduced from Ariel 1988) shows preferences for pronouns, demonstratives, and definite descriptions in different contexts:

|   | Same sentence  | Previous sentence  |  Same paragraph | Across Paragraph | 
|---|---|---|---|---|
| Pronoun  | 93.2%  | 82.1%  | 47.8%  | 26.7%  |
| Demonstrative  | 3.4%  | 12.8%  | 10.8%  | 14.4%  |
| Definite description | 3.4%  | 5.1%  | 41.4%  | 58.9% |

*Table 1: Effect of antecedent distance on referring expression choice*

This shows the pattern that pronouns are markers of high-accessibility antecedents, demonstratives of mid-accessibility, and definite descriptions of low-accessibility. Ariel takes these to be not accidental properties of these expressions, nor derived behavior from their semantics, but rather to more directly be what they mean: a pronoun's semantic denotation is that it mark a high-accessibility antecedent bearing the relevant phi-features. This allows for a very straightforward and implementable meaning in a computational model.

The notion of accessibility as a kind of choice function, which maps a set of possible entities to one intended referent (von Heusinger 2000), allows us to make some very direct predictions about the behavior of the speaker and listener based on past experimental data. Since in some default sense, highly accessible entities are more predictable referents, speakers should "refer efficiently", and use lower-cost utterances to refer to higher-accessibility referents (Tily & Piantadosi 2009). The presence of similar entities to the intended referent in the context creates interference, lowering predictability and pushing speakers towards more specific utterances (Fukumura et al 2011). Additionally, experimental evidence suggests that speakers choose utterances based on their own evaluation of accessibility rather than that of the listener (Fukumura et al 2012), and so the listener must infer the relevant accessibility thresholds of the speaker. In the next section, we present a computational model that approximates some of these predictions about speaker and listener behavior.

## The Model

Before outlining the details of our model, we review the generalized Rational Speech Act model.

### The Rational Speech Act model

Introduced in Frank and Goodman (2012), the Rational Speech Act (RSA) model is a framework for modeling
simple communication scenarios probabilistically. It posits recursive pragmatic reasoning between a listener and a speaker attempting to be optimally informative for the lowest-cost utterances, i.e. speaking most efficiently. With the intent to communicate some property of the world state, a pragmatic speaker reasons about what conclusions a listener who is only concerned with literal meaning of utterances would draw from any given utterance, and selects the utterance most likely to direct the listener to the correct world state. The literal listener here is a modeling tool, not a representation of any real individual in a discourse; real listeners are modeled in a higher recursive layer, the pragmatic listener, which reasons about which utterance a pragmatic speaker (with access to the literal meanings of the literal listener) would select to communicate any given world state, and returns another distribution over states given the utterance it hears from the pragmatic speaker. Our model requires another recursive speaker layer on top of this to resolve an additional variable; this will be discussed below. Computationally, this takes the form of a stack of listeners and speakers: a listener hears an utterance and returns a distribution of states; a speaker observes a state and returns a distribution over utterances. At the bottom of the stack is a meaning function which evaluates the truthiness of an utterance
given a state, which can be viewed as a joint distribution over utterances and states.

### Setup

We model the state as a list of entities that are of either male or female gender (given that all of these utterances are third person singular, this is the only relevant phi feature that could assist in utterance choice), and for simplicity have only one other
identifying property: their height. For our model we assume that every possible combination of gender and height is enumerated in the state. The default state, then, consists of four entities, for each of these combinations, though as we will see nothing about the rest of the model enforces a maximum of four.

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
~~~~

The possible utterances are then the pronouns "null", "he", and "she"; the demonstratives
"that girl", "that boy"; and the definite descriptions "the tall girl", "the short girl", etc.

~~~~
// Possible utterances
var utterances = ['null','he', 'she', 'that boy', 'that girl', 'the tall boy',
                 'the short boy', 'the tall girl', 'the short girl']
~~~~

### Priors
Rather than specifying a separate utility function that penalizes longer utterances,
the utterance prior's distribution is an inverted word count model.
Shorter utterances, such as the pronouns, receive higher mass than longer utterances.
This is highly analgous to coding or compression schemes which seek to minimize total message length.

For our state prior, since we would like to observe the speaker's behavior a fully enumerated state, we can sample
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

Below is our meaning function, the joint distribution over states and utterances:
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

Note: in all subsequent visualizations, using viz.table might
be easier to read than viz.hist. 

### A Simple Case

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
  viz.hist(pragmaticListener(utt, state))
}, utterances)

print("\n")
print("S2")
var printer = map(function(ref) {
    print("referring to")
    print(ref)
    viz.hist(pragmaticSpeaker(state, ref))
}, state)
~~~~

We see that the pronoun is preferred for the higher-access cases, and the definite description, which is both higher cost and informativity, is preferred in the lower-access cases. Note that the utterance distributions of the low access male and low access female have the same ordering over REs, even though they have different accessibilities. This is due to the model partitioning the mass of utterances over semantically compatible entities. Thus, even though the tall male entity here is technically 'mid' accessibility, at 2, it behaves as a high accessibility entity because it is high relative to its competitor.

Given that the model performs inference over relative accessibilities with respect to
entities that share properties, you might expect that the highest access
entities in each class to have identical utterance distributions.
However, this is not the case, as we see "null" given greater probability mass for the higher-access female than for the higher-access male. This is because the "null" utterance treats the entire state as its comparison class, since it abstracts over gender and height features, and so the tall male, which is treated as high-accessibility for the purposes of gender-marked expressions, is treated instead as mid-accessibility for expressions that could select the overall higher-accessibility female entity.


### Fully Enumerated State

We now examine the model's output on a fully enumerated state
with the cartesian product of all genders, heights, and accessibilities. Given the expansive number of possible referents in most real discourse situations, taking into account that possible referents include not only those physically present but also those in some degree of shared epistemic background between the listener and speaker, this is likely an even better model of real behavior.
We expect to see results similar to Table 1 in Section 1.

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

// Sample n random entities from the fully enumerated state
// without replacement
var statePrior = function(n, state) {
  var shuffled = knuthShuffle(state)
  return sort(shuffled.slice(0, n), lt, score)
}

///

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

var state = fullState

///fold:

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

print("State")
printList(state)
print("\n")

print("S2")
var printer = map(function(ref) {
    print("referring to")
    print(ref)
    viz.hist(pragmaticSpeaker(state, ref))
}, state)
~~~~

The simplified nature of our state model and the relatively close steps in our accessibility variable lead to a high actual discrepancy between the model distribution and the data distribution, in terms of quantitative patterns. Namely, the corpus data shows much sharper preferences for the most preferred expression, while our model returns overall much smoother and more gradient preferences between expressions with smaller differences. However, the relative ordering of utterances for each entity is exactly as predicted by the corpus data. Table 1 is reproduced below for ease of comparison:

|   | Same sentence  | Previous sentence  |  Same paragraph | Across Paragraph | 
|---|---|---|---|---|
| Pronoun  | 93.2%  | 82.1%  | 47.8%  | 26.7%  |
| Demonstrative  | 3.4%  | 12.8%  | 10.8%  | 14.4%  |
| Definite description | 3.4%  | 5.1%  | 41.4%  | 58.9% |

Without reference to the empirical data, it might be seen as a fault of our model that the demonstrative is never the most preferred expression, even in its supposedly preferred mid-accessbility contexts. However, this is exactly the pattern borne out in the data. This is likely a reflection of the demonstrative failing to win out in either informativeness or cost, as with the existing model, it encodes no additional information beyond what the strictly lower-cost pronoun does. This is discussed more below.

### Alternative Implementation

The above model was made to stay in keeping with the listener-first trend with RSA.
However, this led to including softer semantics in the meaning function, since during inference
the S1 would encounter situations where an utterance cannot describe any referents due to 
theta thresholds. We present an alternative implementation that forgoes an L0 for an S0,
performing inference as S0 -> L1 -> S1.

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

///

var meanings = function(utterance, referent, theta_dem, theta_pro){
  return utterance == "null" ?
    referent.access >= theta_pro :
  utterance == 'he' && referent.gender == 'male' ?
    referent.access >= theta_pro :
  utterance == 'she' && referent.gender == 'female' ?
    referent.access >= theta_pro :
  utterance == 'that boy' && referent.gender == 'male' ?
    referent.access >= theta_dem :
  utterance == 'that girl' && referent.gender == 'female' ?
    referent.access >= theta_dem :
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
  return uniformDraw(accesses)
};

// optimality
var alpha = 1

// S0
// p(utterance=utt | ref=ent, state) 
//     = 1([utt](referent) | thetas) * p(utterance=utt)
var speaker = function(state, referent, theta_dem, theta_pro){
  Infer({model: function(){
    var utterance = utterancePrior()
    condition(meanings(utterance,referent,theta_dem,theta_pro))
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

// S1
// p(utterance=utt | ref=ent, state; l1)
//     = p(ref=ent | utterance=utt, state, l1) * p(utterance=utt)
var pragmaticSpeaker = function(state, referent) {
  Infer({model: function() {
    var utterance = utterancePrior()
    factor(alpha * pragmaticListener(utterance, state).score(referent))
    return utterance
  }})
}

///fold:

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

///

print("State")
printList(state)

print("\n")
print("L1")
var printer = map(function(utt) {
  print("when hearing '" + utt + "'")
  viz.hist(pragmaticListener(utt, state))
}, utterances)

print("\n")
print("S2")
var printer = map(function(ref) {
    print("referring to")
    print(ref)
    viz.hist(pragmaticSpeaker(state, ref))
}, state)
~~~~

The results are essentially the same. The small difference in the posterior distributions
are a result of the lack of influence from the literal listener's prior. Since the
literal listener had a uniformly uninformative prior, this could be seen as using a 
higher temperature in the Gibbs distributions.

## Limitations & Future Work


While in its current state, the model captures the qualitative behavior of the set of corpus data motivating Accessibility Theory, and many of our intuitions about referring expressions, there are three shortcomings of this approach. The first issue to consider is whether this kind of corpus data is what we want to be modeling in the first place. While Ariel's (1988) data is compiled from a mix of fiction and non-fiction sources, the overall size is relatively small, with four texts contributing.  While our model's results are certainly not at odds with more robust experimental work (e.g. Tily & Piantadosi 2000), it should also be tested against much larger and more varied corpora. There is a secondary concern here as to whether corpus data is the gold standard against which a model should be measured in the first place, and so ideally a model would account for the behavior of referring expressions in both experimental and large-scale corpus contexts.

Second, our model only captures the *qualitative* patterns in the existing corpus data. While we do get the correct relative rankings of preference between different types of referring expressions, the precise probabilities are not well captured. In particular, the preferences for pronouns in high-accessibility conditions and definite descriptions in low-accessibility conditions are much more drastic than our model predicts; while our model does assign a clear preference, the difference between the first and second choice expression is usually in the realm of 10-20% probability, while in the corpus data, it is often closer to 60-70%. Given that this preference is powerful in both directions, it cannot be easily resolved in our model just by assigning a higher prior probability (that is, a lower structural cost) to either pronouns or definite descriptions.

Third, the model's current structure flattens over a good deal of complexity in the state. Accessibility is treated as a non-composite variable that, while gradient, is generated by a raw accessibility prior rather than derived from independent, concrete properties of an entity. An ideal model would directly ascribe to entities only more observable properties, such as its frequency of reference, extralinguistic presence, and discourse topic status, and the speaker would compute accessibility for the purposes of selecting a referring expression from those primitives. This was not implemented in the current model largely because no formalized way to integrate these variables has been proposed within Accessibility Theory; while the important contribution of all of these variables has been acknowledged, future experimental work will be necessary to ascertain exactly how speakers incorporate these different variables into their model, and which are privileged. Some work has suggested that structural factors such as syntactic position have a greater effect than predictability or likelihood of reference (Fukumura et al 2010); more will be necessary to resolve this.

Another fruitful area for future work would be the expansion of this model into a dynamic, iterated form in which the accessibility of each entity is updated at choice points in the discourse, more accurately modeling the progression of a conversation or text. To do this, breaking down the accessibility variable into its component parts would be necessary, as then it would be simple to add to its independent variables, like how many times the entity has been referred to. Different types of reference should affect the accessibiliy of the referent in different ways. For example, using a pronoun should increment the accessibility of its antecedent slightly by increasing its frequency and recency of reference, but have no other distinct effect. However, some work has argued that using a definite description has a more dramatic effect on the accessibility of the antecedent, such that definite descriptions act as accessibility functions, while pronouns merely serve as placeholders for additional reference (von Heusinger 2000). In a similar sense, the current model only identifies demonstratives as middle-cost expressions conditioned on a certain accessibility threshold. Most work on demonstratives acknowledges that they have additional features and functions, such as introducing a new referent into the context, or externalizing speaker intention about the referent in some way (Kaplan 1989). While it is an attractive feature of our current model that the encoded semantics of each referring expression is rather minimal, it seems inevitable that some additional detail will have to be provided in order to capture the unique behavior of certain expressions. If this behavior can be derived pragmatically from a more complex set of priors, all the better.


## References

Ariel, Mira. 1985. *The discourse functions of given information.* Theoretical Linguistics 12: 2/3. (pp. 99-113).

-- 1988. *Referring and accessibility*. Journal of Linguistics 24: 1. (pp. 65-87).

-- 1991. *The function of accessibility in a theory of grammar.* Journal of Pragmatics Volume 16, Issue 5, November 1991, Pages 443-463.

-- 2001. *Accessibility theory: an overview*. In Text Representation: Linguistic and Psycholinguistic Aspects, ed. Sanders and Spooren. (pp. 29-87).

Chomsky, Noam. 1981. *Lectures on government and binding*. Dordrecht: Foris.

Frank, Michael C. and Goodman, Noah D. 2012. *Predicting Pragmatic Reasoning in Language Games* Science Volume 336, pp. 998-998.

Fukumura, Kumiko, and Roger P.G. van Gompel. 2010. *Choosing anaphoric expressions: Do people take into account likelihood of reference?* Journal of Memory and Language 62. (pp. 52–66).

Fukumura, Kumiko, Roger P.G. van Gompel, Trevor Harley, Martin J. Pickering. 2011. *How does similarity-based interference affect the choice of referring expression?* Journal of Memory and Language 65 (pp. 331–344)

Fukumura, Kumiko, and Roger P.G. van Gompel. 2012. *Producing Pronouns and Definite Noun Phrases: Do Speakers Use the Addressee’s Discourse Model?* Cognitive Science 36 (pp. 1289–1311).

Heim, Irene. 1982. *The Semantics of Definite and Indefinite Noun Phrases*. Ph.D. thesis, University of Massachusetts, Amherst, PhD.

Kaplan, David. 1989. “Demonstratives,” in Almog, Perry, and Wettstein 1989, pp. 481–563.

Patel-Grosz, Pritty and Patrick G. Grosz. 2017. *Revisiting pronominal typology*. Linguistic Inquiry 48:2.

Roberts, Craige. 2004. *Pronouns as definites*. In Marga Reimer and Anne Bezuidenhout (eds.), Descriptions and Beyond, 503–543. Oxford: Oxford University Press.

Schlenker, Philippe. 2005. *Minimize Restrictors! (Notes on Definite Descriptions, Condition C and Epithets).* In Proceedings of SuB 9, (385–416). 

Tily, Harry and Steven Piantadosi. 2009. *Refer efficiently: Use less informative expressions for more predictable meanings.* In Proceedings of the workshop on the production of referring expressions: Bridging the gap between computational and empirical approaches to reference.

Von Heusinger, Klaus. 2000. *Anaphora, Antecedents, and Accessibility*. Theoretical Linguistics 26:1-2. (pp. 75-93)

