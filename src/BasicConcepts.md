# Basic Concepts

```javascript
// Ternary operator
(condition) ? iftrue : iffalse

// variable definitio
var a = 2

// function definition
var f = function(a,b,x) {
    return (a*x + b)
}

// tail end recursion
var factorial = function(n) {
    var factorialTR = function(n,r) {
        (n == 0) ? r : factorialTR(n-1,n * r) 
    }
    factorialTR(n,1)
}

// thunk functions sampled
flip(0.9)
// thunk function not sampled
var WeightedCoin = function() {
    return flip(0.9)
}
// Create a function that returns a thunk function !!!
var makeCoin = function(p) {
    return function() {
        flip(p)
    }
}

// repeat GIVES YOU A LIST! 
repeat(n_repetitions, thunk_func)
// ex:
repeat(5, function() {"h"}) // this gives you an array of "h" 5 times

//viz results
viz(repeat(n, f))


//make a distribution using the Bernoulli constructor:
var b = Bernoulli({p: 0.5})

//sample from it with the sample operator:
print( sample(b) )

//compute the log-probability of sampling true:
print( b.score(true) )

//visualize the distribution:
viz(b)
viz.table(Binomial({n : 50, p : 0.5}))

// Distributions
// The categorical distribution takes an array and its associated (unnormalized) probabilities. 
viz(Categorical({ ps:[6,7,3,4] , vs: ["blue","orange","green","yellow"] }))
// Omitting ps gives the uniform distribution over vs
viz(Categorical({ vs: ["blue","orange","green","yellow"] }))
// The discrete distribution over integers according to a probability array 
viz(Discrete({ps : [1,2,3,4,5,6]}))
// Normal
viz(Gaussian({mu : 0, sigma : 1}))
// Uniform Distribution on integers between 0 and n-1 
viz(RandomInteger({n : 11}))
// Continuous Uniform Distribution on reals between a and b 
viz(Uniform({a : 10, b : 20}))

// Infer can transform a thunk function into a distribution
var d = Infer(makeCoin)

// memoisation randomly generates but then remembers
var eyeColor = mem(function (person) {
    return uniformDraw(['blue', 'green', 'brown'])
});

[eyeColor('bob'), eyeColor('alice'), eyeColor('bob')] // same input = sampe output
```

## Methods for viz

```javascript
// if the model outputs more than one distribution (both continuous and discrete), ex.
return {
        prior: prior_p, priorPredictive : priorPredictive,
        posterior : p, posteriorPredictive : posteriorPredictive
    };

// you need to use viz.marginals(Infer(model))
viz.marginals(Infer(model))

```

## Map data


```javascript

var makeFun = function(a) {
    return ( function(x) {
        return(a * x)
      })
}

var randomLine = function() {
    var a = uniform(0,10) // prior on a --> will be influenced by observation of the data
    var func_external = makeFun(a) // returns a function on x
      
    var obsFn = function(datum){
        observe(Gaussian({mu: func_external(datum.x), sigma: 2}), datum.y)
    }
    mapData({data: observedData}, obsFn)
    
    // or, for monodimensional distributions
    // ***
        var dist = Bernoulli({p : weight})
        // Note that we use observe here, with a Bernoulli distribution, this optimizes the inference
        mapData({data: obsData},function(datum) {observe(dist,datum == 'h')})

        // alternative code:
        var coin = makeCoin(weight)
        mapData({data: obsData}, function(datum) {condition(coin() == datum)})
    // ***

    return a      
}

// Infer the distribution on coefficients
var post = Infer(randomLine) // when you infer, the "mapData" and the "observe" inside "obsFn" will actually change a

// "a" will be influenced --> now we have a marginal on a given the observedData
//Visualize the distribution on coefficients
viz(post)
```

## Different Types of Infer

```javascript
var posterior = Infer(model)
var posterior = Infer({method: 'SMC', particles : 5000}, model) // sequential monte carlo: resamples at each observation/conditioning --> works well if the "model" has sampling (ex: coin toss) and conditioning (ex: condition(coin() == datum)) alternated
var posterior = Infer({method: 'rejection', samples : 100}, model)
var posterior = Infer({method: 'SMC', particles : 5000}, model)
var posterior = Infer({method: 'SMC', particles : 5000}, model)
```
