# Chapter 3

## Conditions and Inference

```javascript
var model = function() {
  var A = flip()
  var B = flip()
  var C = flip()
  condition(A + B + C >= 2) // only make sense if then you INFER
  return {'A': A, 'B': B, 'C' : C}
}

// We only need 8 executions to see all the paths for this example. 
var truedist = Infer(model)
var dist = Infer({method: "enumerate", maxExecutions: 8, strategy : "likelyFirst"}, model)
viz.table(truedist)
viz.table(dist)
```

**Oss**: ```"likelyFirst"``` follows the path that are most likely, so if the probability of true is more likely it will follow those paths first --> if you weight a coin more towards the false, likelyFirst will follow that path only AFTER having followed the truth path. We can "solve" this by taking away weight from the negative path:

```javascript
// Factor can be used to guide the heuristics. 
// Here, we remove some weight to the path where A is false
var model = function() {
  var A = flip()
  if (A == false) {factor(-10)};
  var B = flip()
  var C = flip()
  // But we give back the weight after (otherwise the fraction N(A|not b) will be much smaller because each sample has much lower weight)
  if (A == false) {factor(+10)};
  condition(A + B + C >= 2)
  return {'A': A, 'B': B, 'C' : C}
}
// This time it works. 
var truedist = Infer(model)
var dist = Infer({method: "enumerate", maxExecutions: 6, strategy : "likelyFirst"},model)
viz.table(truedist)
viz.table(dist)
```

## Using observe to condition distributions

```javascript
// observed data
var k = 1 // number of people who support candidate A
var n = 20  // number of people asked

var model = function() {
    // standard model structure, no evidence
    var prior_p = uniform(0, 1);
    // given that each person has probability p, which is the probability that (out of n people), x will vote A?
    var priorPredictive = binomial(prior_p, n); 

    // assume that this is the probability distribution of population proportion who support candidate A
    var p = uniform(0, 1);

    // Observed k people support "A" --> FIXED k, this will condition p
    // Assuming each person's response is independent of each other, with "prior" p
    observe(Binomial({p : p, n: n}), k);

    // predict what the next n will say (with an updated p)
    var posteriorPredictive = binomial(p, n);


    return {
        prior: prior_p, priorPredictive : priorPredictive,
        posterior : p, posteriorPredictive : posteriorPredictive
    };
}

var posterior = Infer(model);

viz.marginals(posterior)
```

## Drawing to image + generative model of linear regression + noise + data

```javascript
//Observed data
var observedData = [{"x":0,"y":0},{"x":1,"y":5.2},{"x":2,"y":9.4},{"x":3,"y":15},{"x":4,"y":20.5},{"x":5,"y":24.3}]
    
///fold
// Create a drawing of size 500,500
var myDraw = Draw(500, 500, true)
    
//Intermediate functions to draw

//Scale a point to the image
var scale = function(p) {
      var newX = 20 + p.x * 75
      var newY = 480 - p.y * 15
      return{"x":newX,"y":newY}
}
    
// Draw a list of data points
var drawData = function(img,obs) {
    var scaleData = map(scale,obs)
    map(function(p) {
      img.circle(p.x, p.y, 3, 'black', 'white')
    }, scaleData)
}
    
// Draw a linear function 
var drawLinFun = function(img,f) { 
      var a = scale({"x":0, "y":f(0)})
      var b = scale({"x":5.5, "y":f(5.5)})
      img.line(a.x,a.y,b.x,b.y,1,0.01,"red")
}
///


// Make a function from a coefficient
var makeFun = function(a) {
    return ( function(x) {
        return(a * x)
      })
}
    
// Compute a Random Line, and observe data
// The prior distribution on a is the uniform one 
// The generative model take into account some noises, expressed by a gaussian. 
var randomLine = function() {
    var a = uniform(0,10)
    var f = makeFun(a)
      
    var obsFn = function(datum){
        observe(Gaussian({mu: f(datum.x), sigma: 2}), datum.y) // why noise on x?
    }
    mapData({data: observedData}, obsFn)
      
    return a      
}

// Infer the distribution on coefficients
var post = Infer(randomLine)
    
// Draw samples from the inferred distribution, to have some visual intuition about the distribution
var randomDraw = function() {
    var a = sample(post)
    drawLinFun(myDraw,makeFun(a))
}

//Visualize the distribution on coefficients
viz(post)
    
// Draw the data 
drawData(myDraw,observedData)
// Draw the samples from the marginal distribution 
repeat(1000,randomDraw)
    
print("The value should be around 5")
```

## Fair / unfair coin based on prior and data

```javascript
// Our degree of belief for the fact that the coin is fair 
var fairPrior = 0.999
// The observed list of data 
var obsData = repeat(5,function() {'h'})

//Make a Coin of a Given Weight
var makeCoin = function(x) {
    return function() {
        (flip(x)) ? 'h' : 't'
    }
}

//Generative Model 
var model = function() {
    //We create the coin acording to our belief 
    var prior = flip(fairPrior)
    var isFair = flip(fairPrior)
    var coin = (isFair) ? makeCoin(0.5) : makeCoin(0.95)
    // A standard way to compare data with a distribution is to use "map"
    mapData({data: obsData}, function(datum) {condition(coin() == datum)})
    // mapData is similar to map, but you inform the inference that all calls are independent
    return {prior: prior, posterior: isFair}
}

var post = Infer(model)
viz.marginals(post)
```

### How to influence your prior with data?

The "obsFn" can be made wither with an "observe" in the middle of the function, or with a "condition". Both cases, when followed by an "Infer" afterwards, will have an influence on the initial distribution (that can also be a CALL to a thunk function) (?)

```javascript
// We can be even more precise in our prior distribution.
// We can try to infer the actual weight of the coin, by believing that trick coin could have any weight > 0.5

//Generative Model 
var model = function() {
    //We create the coin acording to our belief 
    var prior = flip(fairPrior)
    var isFair = flip(fairPrior)
    // The weight is either 0.5, or a trick coin that prefer head 
    var weight = (isFair) ? 0.5 : uniform(0.5,1)


    var dist = Bernoulli({p : weight})
    // Note that we use observe here, with a Bernoulli distribution, this optimizes the inference
    mapData({data: obsData},function(datum) {observe(dist,datum == 'h')})

    // alternative code:
    var coin = makeCoin(weight)
    mapData({data: obsData}, function(datum) {condition(coin() == datum)})

    return {prior: prior, posterior: weight}

}

// We can try SMC as we alernate between observations and samples 
var post = Infer({method: 'SMC', particles : 5000},model)
viz.marginals(post)
```

