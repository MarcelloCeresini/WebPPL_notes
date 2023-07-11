# Ex3, Hierarchical Models

## A Simple bag model

```javascript
var colors = ["blue","red","green"]
var data = ["green","green","green","green","green","green","blue"]

var makeBag = function() {
    // We define the probability array as an array of numbers sampled uniformly
    // for each color we sample a probability
    var ps = repeat(colors.length,function() {return uniform(0,1)})
    return Categorical({ps, vs : colors})
}

var prior = function() {
    return repeat(colors.length,function() {return uniform(0,1)})
}

var priorPredictive = function() { // 
    return sample(makeBag()) // sample of the distribution built from the prior
}

viz(Infer(prior)) // what is the probability of each possible array of 3 values to appear (uniform distrib over the cube [0,1]^3)
viz(Infer(priorPredictive)) // what is the probability of blue, red and green to be the result of the model (uniform distrib over the discrete values RGB)
  
var posterior = function() {
    var ps = repeat(colors.length,function() {return uniform(0,1)}) // priors of the single colors
    var distribution = Categorical({ps, vs : colors}) // create a DISTRIBUTION obj with those probab
    mapData({data: data}, function(datum) {observe(distribution, datum)}) // observe with data
    return ps // return the initial priors conditioned (posterior)
}

var predictivePosterior = function() {
    var ps = repeat(colors.length,function() {return uniform(0,1)})
    var distribution = Categorical({ps, vs : colors}) 
    mapData({data: data}, function(datum) {observe(distribution, datum)}) 
    return sample(distribution) // sample from the distribution built from the prior (conditioned)
}

viz(Infer(posterior))
viz(Infer(posteriorPredictive)) // MUCH more informative (in high dimensions + continuous priors but discrete distribution)
```

MORE BAGS

```javascript
var colors = ["blue","red","green"]

var data = [{bag: 1, color: "green"},{bag: 1, color: "green"},{bag: 1, color: "green"},{bag: 1, color: "green"},{bag: 1, color: "green"},{bag: 1, color: "green"}, {bag: 1, color: "blue"},
{bag: 2, color: "blue"},{bag: 2, color: "blue"},{bag: 2, color: "blue"},{bag: 2, color: "blue"},{bag: 2, color: "red"},
{bag: 3, color: "red"},{bag: 3, color: "red"},{bag: 3, color: "red"},{bag: 3, color: "red"},{bag: 3, color: "red"},{bag: 3, color: "red"},{bag: 3, color: "red"}]

// 
var makeBag = mem(function(n) {
  var ps = dirichlet(Vector([1,1,1])) // more optimized (you only need 2 of the 3 priors)
  // When you increase the value of a parameter by 1, it is as if you had one more sample from this color.
  return Categorical({ps, vs : colors})
})

var model = function() {
  mapData({data: data}, function(datum) {
    observe(makeBag(datum.bag), datum.color)
  })
  return {1: sample(makeBag(1)), 2: sample(makeBag(2)), 3: sample(makeBag(3))} // three different posteriorPredictive (distribution of SAMPLES of the discrete distribution, while posterior/prior are the distributions on the PARAMETERS OF THE DISTRIBUTIONS!)
}

viz.marginals(Infer(model))
```

```javascript
var prototype = T.mul(A,colors.length) // T.mul(VECTOR, NUMBER) --> Vector = Vector([1,2,3])
```

## Sampling from curve

```javascript
var model = function() {
  var x = gaussian(xmu, xsigma);
  var y = gaussian(ymu, ysigma);
  condition(onCurve(x, y));
  return {x: x, y: y};
  // if you want to use rejection sampling, you can see what is the probab of being on curve with p
  // var p = onCurve(x, y)
  //return p
};

var post = Infer({method: 'rejection', samples: 1000}, model);
// different infers with MCMC (verbose shows you the acceptance rate, lag increases the number of steps between accepted samples in MCMC)
var post = Infer({method: 'MCMC', samples: 10000}, model);
var post = Infer({method: 'MCMC', samples: 100000}, model);
var post = Infer({method: 'MCMC', samples: 1000, verbose: true}, model);
var post = Infer({method: 'MCMC', samples: 10000, lag: 10}, model);
var post = Infer({method: 'MCMC', samples: 1000, lag: 100}, model);

// better with MCMC to use a single sampling ("single state change")
var model = function() {
  var a = diagCovGaussian({mu: Vector([xmu, ymu]),sigma: Vector([xsigma, ysigma])});
  var x = a.data[0]
  var y = a.data[1]
  condition(onCurve(x, y));
  return {x: x, y: y};
};
```
