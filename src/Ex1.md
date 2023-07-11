# Ex1

## Functional Programming

### Simple Example

```javascript
var g = function(a,b) {
  var f = function(a,b) {
    if (b > 0) {
      return (a + b)
    }
    else {
      return (a - b)
    }
  }
  return f(a,b)
}
```

Returns a + |b|, to make it simpler:

```javascript
var g = function(a,b) {
    var f = function(x) {
        (x>0) ? return(x) : return(-x)
    }
    return (a+f(b))
}
```

### Newton's Method

```javascript
var applyN = function(x,f,n) {
  if (n == 0) {
    return x
  }
  else {
    return applyN(f(x), f, n-1)
  }
}

var incr = function(x) {
  return x+1 
}

applyN(5,incr,10) // Should be equal to 15.
```

Copy/Paste the function applyN you defined above. And use it to iterate once, twice, 5 times and 100 times the function f(x) = (x / 2) + 1/x starting from 1. Compare this result with the square root of 2. What can you deduce ?

```javascript
var applyN = function(x,f,n) {
  (n == 0) ? x : applyN(f(x), f, n-1)
}

var sqrt_2_gen = function(x) {
  return (x / 2) + 1/x
}

display(Math.sqrt(2))
display([applyN(1,sqrt_2_gen,1), applyN(1,sqrt_2_gen,2), applyN(1,sqrt_2_gen,5), applyN(1,sqrt_2_gen,100) ])
```

In the general case, if you want to find the square root of R, you can use the function f(x) = (x / 2) + (R / 2x). Verify that this works with the square root of 3.

```javascript
var applyN = function(x,f,n) {
  (n == 0) ? x : applyN(f(x), f, n-1)
}

var func_gen_sqrt_R = function(r) {
  return function(x) { (x/2)+(r/(2*x)) }
}

display(Math.sqrt(3))
display([applyN(1,func_gen_sqrt_R(3),1), applyN(1,func_gen_sqrt_R(3),2), applyN(1,func_gen_sqrt_R(3),5), applyN(1,func_gen_sqrt_R(3),100) ])
```

## Generative Models

### Simple Examples

Show they have the same MARGINAL DISTRIBUTION (probability of being true at the end --> P(b) = P(b|A) * P(A) pointwise product + marginalization )

```javascript
flip() ? flip(0.6) : flip(0.2)
flip(flip() ? 0.6 : 0.2)
flip(.4)
```

Sample from the program 1000 times and plot the results

```javascript
viz(repeat(1000, function () {flip() ? flip(0.6) : flip(0.2)}))
```

### Persistent Randomness

```javascript
var foo = flip() // give A value to foo, then display it 3 times
display([foo, foo, foo])

var foo = function() { return flip() } // function foo calls the flip function every time
display([foo(), foo(), foo()])

var foo = mem(function() { return flip() }) // adding mem fixes the value (given the same input value, in this case it's empty)
display([foo(), foo(), foo()])

var foo = mem(function(x) { return flip() }) // adding an argument lets us choose which calls will have the same value
display([foo(0), foo(0), foo(1)])
```

### Medical Diagnosis

```javascript
var dist = Infer({method: "forward", samples: 1000}, function() { // infer = joint distribution (or marginal if it's only one return)
  var allergies = flip(0.3)
  var cold = flip(0.2)

  var sneeze = cold || allergies
  var fever = cold
  return { sneeze, fever }
})

viz(dist)

viz.table(dist)


// add the possibility of seeing the joint for each person --> need to use mem to keep the symptoms CONSISTENT (the two calls to "cold" for the same person need to give the same result)
var model = function() {
  var allergies = mem(function(person) { return flip(.3) }) 
  var cold = mem(function(person) { return flip(.2) })

  var sneeze = function(person) { return cold(person) || allergies(person) }
  var fever = function(person) {return cold(person)}

  return [sneeze('bob'),fever('bob')]
}

var dist = Infer(model)

viz(dist)
viz.table(dist)
```