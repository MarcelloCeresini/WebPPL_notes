# Ex2

main header content

## Fair Coins and Biased Coins

```javascript
var model = function() {
  return flip()// Your code here
}

var logProb = Infer({method:'enumerate'}, model).score(true); // score is the log prob
Math.exp(logProb); // need to exponentiate it

var model = function() {
  var fair = function() {flip()}
  var biased = function() {flip(0.9)}
  var isFair = flip()
  var myCoin = (isFair) ? fair : biased
  
  var a = myCoin()
  var b = myCoin()
  var c = myCoin()
  condition(a==true)
  condition(b==true)
  return c

  // or also 
  // condition(c==true)
  return isFair
}

viz.table(Infer({method:'enumerate'}, model));

```

## Conditioning and Intervention

```javascript
var lungCancer = flip(0.01);
var cold = flip(0.2);
var cough = (
  (cold && flip(0.5)) ||
  (lungCancer && flip(0.3))
);
cough;
```

Now instead: example in which intervention and conditioning give different results

```javascript
var model1 = function(){
  var lungCancer = flip(0.1);
  var cold = flip(0.2);
  var cough = true
  return cold
}

var model2 = function(){
  var lungCancer = flip(0.1);
  var cold = flip(0.2);
  var cough = (
    (cold && flip(0.5)) ||
    (lungCancer && flip(0.3))
  );
  condition(cough == true)
  return cold
}

viz.table(Infer(model1))
viz.table(Infer(model2))
```

## Computing Marginals

Find the marginal distribution of a model --> simply use bayes to transform conditional distrib into joint, then multiply the priors

## Extending the Smiles Model

```javascript
var extendedSmilesModel = function() {
  var nice = mem(function(person) { flip(.7) });
  var wantSomething = function(person) { (nice(person)) ? flip(.2) : flip(.5)}
  
  var smiles = function(person, wants) {
    return wants ? flip(.8) : ((nice(person)) ? flip(.8) : flip(.5));
  }
  
  var wantsAlice = wantSomething("alice")
  // condition(wantsAlice == true)
  // condition(wantsAlice == false)
  // condition(nice('Alice') == true)
  // condition(nice('Alice') == false)
  return smiles('alice', wantsAlice);                                            
}
```

Instead if he doesn't smile 5 days and today smiles?

```javascript
var extendedSmilesModel = function() {
  var nice = mem(function(person) { flip(.7) });
  var wantsFun = function(person) {
    return nice(person) ? flip(0.2) : flip(0.5) 
  }
  var smiles = function(person,wants) {
    return wants ? flip(0.8) : (nice(person) ? flip(.8) : flip(.5))
  }
  // Note that we repeat the same condition, because this function call is random 
  // Thus, each call reinforce our belief that Bob isn't nice. 
  condition(!(smiles('Bob',wantsFun('Bob'))))
  condition(!(smiles('Bob',wantsFun('Bob'))))
  condition(!(smiles('Bob',wantsFun('Bob'))))
  condition(!(smiles('Bob',wantsFun('Bob'))))
  condition(!(smiles('Bob',wantsFun('Bob'))))
  var wantsBob = wantsFun('Bob')
  condition(smiles('Bob',wantsBob))
  return wantsBob
}


viz.table(Infer({method: "enumerate"}, extendedSmilesModel));
```

## Priors and Predictives

```javascript
// observed data
var k = 1; // number of successes
var n = 20;  // number of attempts
var priorDist = Uniform({a: 1, b: 1});

var model = function() {
   var p = sample(priorDist);

   // Observed k number of successes, assuming a binomial
   observe(Binomial({p : p, n: n}), k);

   // sample from binomial with updated p
   var posteriorPredictive = binomial(p, n);

   // sample fresh p (for visualization)
   var prior_p = sample(priorDist);
   // sample from binomial with fresh p (for visualization)
   var priorPredictive = binomial(prior_p, n);

   return {
       prior: prior_p, priorPredictive : priorPredictive,
       posterior : p, posteriorPredictive : posteriorPredictive
   };
}

var opts = {method: "MCMC", samples: 2500, lag: 50};
var posterior = Infer(opts, model);

viz.marginals(posterior);

viz(Beta({a: 1 + 1, b: 1 + 20 - 1}))
```

If the prior is Beta(A,B) and you observe k sucesses and (n-k) failures, then the posterior distribution is Beta(A + k, B + n - k).
