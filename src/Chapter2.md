# Chapter 1 notes

## Generative phisical model

This code creates shapes with dimensions and positions, and physics.animate makes them fall

```javascript
var dim = function() { return uniform(5, 20) }
var staticDim = function() { return uniform(10, 50) }
var shape = function() { return flip() ? 'circle' : 'rect' }
var xpos = function() { return uniform(100, worldWidth - 100) }
var ypos = function() { return uniform(100, worldHeight - 100) }

var ground = {shape: 'rect',
              static: true,
              dims: [worldWidth, 10],
              x: worldWidth/2,
              y: worldHeight}

var falling = function() {
  return {
    shape: shape(),
    static: false,
    dims: [dim(), dim()],
    x: xpos(),
    y: 0}
};

var fixed = function() {
  return {
    shape: shape(),
    static: true,
    dims: [staticDim(), staticDim()],
    x: xpos(),
    y: ypos()}
}

var fallingWorld = [ground, falling(), falling(), falling(), fixed(), fixed()]
physics.animate(1000, fallingWorld);
```

## Generative Model for a Bayesan Network

```javascript
// priors
var lungCancer = flip(0.01)
var TB = flip(0.005)
var stomachFlu = flip(0.1)
var cold = flip(0.2)
var other = flip(0.1)

// conditional probabilities with leaky-or ("other" is a leak node)
var cough = 
    (cold && flip(0.5)) ||
    (lungCancer && flip(0.3)) ||
    (TB && flip(0.7)) ||
    (other && flip(0.01))

var fever = 
    (cold && flip(0.3)) ||
    (stomachFlu && flip(0.5)) ||
    (TB && flip(0.1)) ||
    (other && flip(0.01))

var chestPain = 
    (lungCancer && flip(0.5)) ||
    (TB && flip(0.5)) ||
    (other && flip(0.01))

var shortnessOfBreath = 
    (lungCancer && flip(0.5)) ||
    (TB && flip(0.2)) ||
    (other && flip(0.01))

var symptoms = {
  cough: cough,
  fever: fever,
  chestPain: chestPain,
  shortnessOfBreath: shortnessOfBreath
}

// by calling symptoms, we sample the model TOPOLOGICALLY from parents to children
symptoms
```

## Risk Game (still generative model)

IMPORTANT: if a function f RETURNS a pair {a, b}, you can extrapolate it as if it was a dict --> f.a and f.b

```javascript
// Modelization of a battle in the Risk Game 
var dice = function() {
  return (sample(RandomInteger({n:6})) + 1)
}

// Compute the number of remaining attackers and defenders for a given dice throw
var compare = function (nba,la,nbd,ld,n) {
  if (n == 0) {
    return {nba,nbd}
  }
  else {
    return (la.pop() > ld.pop()) ? compare(nba,la,nbd + 1,ld,n-1) : compare(nba + 1, la, nbd, ld ,n-1)
    // if la[i] > lb[i] add one death to b
  }
}

// Define a single round of the risk game, with x attackers and y defenders
var round = function(x,y) {
  var att = sort(repeat(x,dice))    // x dice rolls
  var def = sort(repeat(y,dice))    // y dice rolls
  var take = Math.min(x,y)          // take the min number between attackers and defenders
  var c = compare(0,att,0,def,take) // start with 0 deaths each and the lists of dices. n is the number of comparisons
  return c
}

// A full battle, which is a sucession of rounds. 
var war = function(x,y) {
  if (x == 0 || y == 0) {return {x,y}}
  else {
    var nbatt = Math.min(3,x) // at least 3 attackers
    var nbdef = Math.min(2,y) // at least 2 defenders
    var c = round(nbatt,nbdef) // do a round
    return war(x - c.nba,y - c.nbd) // recursive, decrease the number of tanks wrt the deaths
  }
}

var nbatt = 40 
var nbdef = 40

// Get the number of remaining defenders 
var distfun = function() {
  var res = war(nbatt,nbdef)
  return res.y // if a function f RETURNS a pair {a, b}, you can extrapolate it as if it was a dict --> f.a and f.b
}

var dist = Infer(distfun)

viz(dist)
```

