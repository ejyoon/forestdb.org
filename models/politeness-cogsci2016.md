---
layout: model
title: Politeness (CogSci 2016)
model-language: webppl
---
<script src="http://web.stanford.edu/~erindb/webppl-viz/webppl.min.js"></script>  
<link rel="stylesheet" href="http://web.stanford.edu/~erindb/webppl-viz/viz.css">

This is a model of polite language used in Ref:yoonTesslerPolitenessCogSci.


## Polite RSA

~~~~
var states = [1,2,3,4,5]
var weightBins = [0.1,0.3,0.5,0.7,0.9]

var utterances = ["terrible","bad","okay","good","amazing"]

var statePrior = function(){
  return uniformDraw(states)
}

var utterancePrior = function(){
  return uniformDraw(utterances)
}

// model parameters
var alpha = 1.25
var speakerOptimality = 10

// measured in Experiment 1
var literalSemantics = {
// "US": [
  "terrible":[.93,.34,.03,.00,.03],
  "bad":[.88,.86,.03,.07,.03],
  "okay":[0.00,0.26,1.00,.60,.36],
  "good":[.03,.07,.57,.97,.93],
  "amazing":[.03,.00,.07,.64,1.00]
// ],
// "India": [
//  "terrible":[.75,.56,.12,.23,.20],
//  "bad":[.80,.75,.12,.14,.08],
//  "okay":[.09,0.44,0.92,.68,.58],
//  "good":[.06,.16,.73,.97,.91],
//  "amazing":[.11,.16,.36,.80,.97]
//]
}

var meaning = function(words, state){
    return words=="sayNothing" ? true : flip(literalSemantics[words][state-1])
} 

var listener0 = cache(function(utterance) {
  Enumerate(function(){
  var state = statePrior()
  var m = meaning(utterance, state)
  condition(m)
  return state
  })
})


var speaker1 = cache(function(state, speakerGoals) {
  Enumerate(function(){
    var utterance = utterancePrior()

    var L0 = listener0(utterance)

    var epistemicUtility = L0.score([],state)
    var socialUtility = expectation(L0, function(s){return alpha*s})

    var eUtility = speakerGoals.honesty*epistemicUtility 
    var sUtility = speakerGoals.kindness*socialUtility

    var speakerUtility = eUtility+sUtility

    factor(speakerOptimality*speakerUtility)

    return utterance
  })
})

var listener1 = function(exptCondition, queryStatement) {
  Enumerate(function(){

    var utterance = exptCondition.utterance
    var trueState = exptCondition.state
    var knownGoalsWeights = exptCondition.goalWeights

    var state = statePrior()

    // Expt 2. goal weights are known (e.g. "speaker is trying to be nice")
    var speakerGoals = knownGoalsWeights ? knownGoalsWeights : 
      {
        honesty: uniformDraw(weightBins),
        kindness: uniformDraw(weightBins)
      }

    // Expt 3. trueState is known.
    condition(trueState ? trueState == state : true)

    var S1 = speaker1(state, speakerGoals)

    factor(S1.score([],utterance))

    var returnStatement = {
      state: state,
      goals: speakerGoals
    }

    return returnStatement[queryStatement]
  })
}

// Experiment 2 setup
var expt2 = listener1(
{
  utterance: "amazing",
  state: false,
  goalWeights: {
        honesty: 0.3,
        kindness: 0.9
  }
}, "state")

print("How good was the project?")
print("Speaker says amazing, but you know she was trying to be nice.")
print(expt2)

// Experiment 3 setup

var expt3 = listener1(
{
  utterance: "amazing",
  state: 3,
  goalWeights: false
}, "goals")

print("What were the speaker's goals?")
print("Speaker says amazing, but you know the project only deserved a 3 out of 5.")

vizPrint(expt3)

~~~~


References:

- Cite:yoonTesslerPolitenessCogSci

