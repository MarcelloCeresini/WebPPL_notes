# Exam 2

## First Exercise

```javascript
var data = ["w", "w", "b"]

var model = function() {
  var prior = uniform(0, 1)
  var distPrior = Categorical({ps:[prior, 1-prior], vs:["w", "b"]})
  var priorPredictive = sample(distPrior)
  
  var posterior = uniform(0, 1)
  var distPosterior = Categorical({ps:[posterior, 1-posterior], vs:["w", "b"]})
  var posteriorPredictive = sample(distPosterior)
  mapData({data:data}, function(datum) {observe(distPosterior, datum)})
  
  return {prior: prior, priorPredictive: priorPredictive,
          posterior: posterior, posteriorPredictive: posteriorPredictive}
  
}

viz.marginals(Infer(model))
```

## Second Exercise

### First Question

```javascript
var model = function() {
  var wakeUpLate = mem(function(person) {flip(0.1)})
  var trainIsLate = mem(function() {flip(0.01)})
  var LateInClass = function(person) {wakeUpLate(person) || trainIsLate()}
  
  condition(LateInClass("A"))
  
  return wakeUpLate("A")
}

viz(Infer(model))
```

### Second Question

```javascript
var model = function() {
  var wakeUpLate = mem(function(person) {flip(0.1)})
  var trainIsLate = mem(function() {flip(0.01)})
  var LateInClass = function(person) {wakeUpLate(person) || trainIsLate()}
  
  condition(LateInClass("A"))
  condition(LateInClass("B"))
  condition(LateInClass("C"))
  
  return trainIsLate()
}

viz(Infer(model))
```
