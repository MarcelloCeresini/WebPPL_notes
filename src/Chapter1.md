# Basis of WebPPL

## Basic Functions and their meaning

```javascript
// Abstract Equality, attempts to type cast    
2 == "2"

// Strict Equality    
2 === "2"

// Some Boolean Operators    
!(true && (true || false))

// Conditionals
if (5 > 1) {1} else {2}
// You can use the ternary operator to gain space and time  
(5 > 1) ? 1 : 2

// You can only define function using this syntax
var f = function(a,b,x) {
    return (a*x + b)
}
f(3,5,4)

// Not imperative but ou can still do this:
var a = [1,2,3,4]
a.pop() 
a.push(1)
print(a)
```

### Strings methods

```javascript
// Slice from beginning (included) to end (excluded). Also useful to create a copy of an array for example
str.slice(1,3)

//Replace one occurence, you can also use regular expressions and replaceAll
str.replace("l","r")

// Some other useful functions includes operations on the alhabet
str.toUpperCase()

// You can use the standard string concatenation as a method instead of using + 
str.concat(" World")
```

### Tail recursion

```javascript
var factorial = function(n) {
    var factorialTR = function(n,r) {
        (n == 0) ? r : factorialTR(n-1,n * r) 
    }
    factorialTR(n,1)
}
```

### Higher Order Functions

```javascript
print("Some Examples of Uses of Higher-Order Operators")

// Compute the square from 0 to 10
print(map(function(x) {x * x},                              _.range(0,11)))

// Compute the logs from 0 to 10 using mapN
print(mapN(function(x) {Math.log(x)},                       11))

//Compute the sums of the two list
print(map2(function(x,y) {x + y},                           [10,15,5,3,6],          [4,7,9,0,13]))

//Compute the factorial by taking the product of the list of numbers from 1 to 5
print(reduce(function(x,r) {x * r},                         1,                      _.range(1,6)))

//Filter a list to keep only the even numbers
print(filter(function(x) {return (x % 2 == 0)},             _.range(0,11)))
```

### Samples

Poll sample

```javascript
// observed data
var k = 1 // number of people who support candidate A
var n = 20  // number of people asked

var model = function() {

    // true population proportion who support candidate A
    var p = uniform(0, 1);

    // Observed k people support "A"
    // Assuming each person's response is independent of each other
    observe(Binomial({p : p, n: n}), k);

    // predict what the next n will say
    var posteriorPredictive = binomial(p, n);

    // recreate model structure, without observe
    var prior_p = uniform(0, 1);
    var priorPredictive = binomial(prior_p, n);

    return {
        prior: prior_p, priorPredictive : priorPredictive,
        posterior : p, posteriorPredictive : posteriorPredictive
    };
}

var posterior = Infer(model);

viz.marginals(posterior)
```

Linear Regression Sample

```javascript
//Observed data
var observedData = [{"x":0,"y":0},{"x":1,"y":5.2},{"x":2,"y":9.4},{"x":3,"y":15},{"x":4,"y":20.5},{"x":5,"y":24.3}]
    
    
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
    
// Make a function from a coefficient
var makeFun = function(a) {
    return ( function(x) {
        return(a * x)
      })
}
    
//Compute a Random Line, and observe data
var randomLine = function() {
    var a = uniform(0,10)
    var f = makeFun(a)
      
    var obsFn = function(datum){
        observe(Gaussian({mu: f(datum.x), sigma: 2}), datum.y)
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
    
drawData(myDraw,observedData)
repeat(1000,randomDraw)
    
print("The value should be around 5")
```
