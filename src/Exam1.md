# Exam 1

## First Exercise

```javascript
var model = function(){
  var learnLesson = mem(function(person) {flip(.8)})
  var examDifficult = mem(function() {flip(.4)})
  var success = function(person) {
    learnLesson(person) ? flip(0.9) : (examDifficult() ? flip(0.3) : flip(0.6))
  }
  
  var ExamA = success("A")
  var ExamB = success("B")
  var ExamC = success("C")
  condition(ExamA == true)
  condition(ExamB == true)
  condition(ExamC == true)
  
  return learnLesson("A")
}

viz.table(Infer(model))
```

## Second Exercise

```javascript
var fairDice = function() {
  var ps = [1, 1, 1, 1, 1, 1]
  var vs = [1, 2, 3, 4, 5, 6]
  return Categorical({ps:ps, vs:vs})
}

var trickDice = function() {
  var ps = [1, 1, 1, 1, 1, 5]
  var vs = [1, 2, 3, 4, 5, 6]
  return Categorical({ps:ps, vs:vs})
}

var data = [6, 6, 1, 6]

var model = function() {
  var isFair = flip()
  var dice = (isFair) ? fairDice() : trickDice()
  
  mapData({data: data}, function(datum) {observe(dice, datum)})
  return isFair
}

viz(Infer(model))
```
