---
title: "Javascript - Héritage & assign/extend"
date: 2015-04-22 02:14:44 +0200
categories: [javascript, héritage]
slug: javascript-inheritence-assign-extend
---

Une petite reflexion autour de l'héritage en Javascript ES5.

## Assign/Extend

### Example 1-1

```javascript
// one
var one = { val: 'A', fn: function(){ return this.val; } };
this.one = one;

// two
var two = Object.assign({}, one);
this.two = two;
```

<!--more-->

![example1-1](/images/example1-1.png)

- Notes :
  - Le prototype de `one` est `Object.prototype`.
  - Le prototype de `two` est `Object.prototype`.
  - Les attributs et les fonctions de `one` sont disponible dans `two`.
- **Conclusion : pas de liaison directe entre les objets -> héritage ??**


### Example 1-2

```javascript
// one
var one = new Date();
this.one = one;

// two
var two = Object.assign({}, one);
this.two = two;

// three
var three = Object.assign({}, Date.prototype);
this.three = three;
```

![example1-2](/images/example1-2.png)

- Notes :
  - Le prototype de `one` est `Date.prototype`.
  - Le prototype de `two` est `Object.prototype`.
    - Donc l'objet `two` ne peut appeller aucunes des méthodes de `Date`.
  - Le prototype de `three` est `Object.prototype`.
    - Le prototype de `Date` est un cas particulier car ces méthodes ne sont pas énumérables.
- **Conclusion : pas d'héritage**


### Example 1-3

```javascript
// one
function One(){
  this.val = 'A';
}
One.prototype.fn = function(){
	return this.val;
}
var one = new One()
this.one = one;

// two
var two = Object.assign({}, one);
this.two = two;
```

![example1-3](/images/example1-3.png)

- Notes :
  - Le prototype de `one` est `One.prototype`. On peut aussi observer que le contructeur de `One` a pour prototype `Function.prototype`.
  - Le prototype de `two` est `Object.prototype`.
    - Donc l'objet `two` ne peut appeller aucunes méthodes de `One`
    - L'objet `two` possède une propriété `val` qui est une copie de `one.val` au moment de l'assignation.
- **Conclusion : état étrange, copie partielle -> héritage ??**


### Exemple 1-4

```javascript
// one
function One(){
  this.val = 'A';
}
One.prototype.fn = function(){
	return this.val;
}
var one = new One()
this.one = one;

// two
var two = Object.assign({}, One.prototype);
this.two = two;
```

![example1-4](/images/example1-4.png)

- Notes :
  - Le prototype de `one` est `One.prototype`. On peut aussi observer que le contructeur de `One` a pour prototype `Function.prototype`.
  - Le prototype de `two` est `Object.prototype`.
    - L'objet `two` possède pas la propriété `val`.
    - L'objet `two` ne possède une fonction `fn` qui est une copie de `One.prototype.fn`.
    - L'objet `two` ne possède pas le construteur `contructor` de l'objet `One`.
- **Conclusion : état étrange, copie partielle -> héritage ??**


## Prototype

### Example 2-1

```javascript
// one
var one = { val: 'A', fn: function(){ return this.val; } };
this.one = one;

// two
var two = {};
two.prototype = one;
this.two = two;
```

![example2-1](/images/example2-1.png)

- Notes :
  - Le prototype de `one` est `Object.prototype`.
  - Le prototype de `two` est `Object.prototype`.
    - Cette manière est stupide car `prototype` est considéré comme un attribut non pas comme l'accesseur du `prototype`.
    - Logiquement aucun attribut ni aucune fonction de `one` n'est donc disponible sur `two`.
- **Conclusion : Stupide -> WRONG WAY**


### Example 2-2

```javascript
// one
var one = { val: 'A', fn: function(){ return this.val; } };
this.one = one;

// two
function Two(){}
Two.prototype = one;
this.two = new Two();

one.val = 'N';
```

![example2-2](/images/example2-2.png)

- Notes :
  - Le prototype de `one` est `Object.prototype`.
  - Le prototype de `two` est `this.two.<prototype>`, c'est un prototype anonyme.
    - Tous les attributs et les fonctions de `one` sont accessible par `Two`.
    - Le `prototype` de `Two` est directement lié à l'instance `one` donc toutes modifications des attributs de `one` sont aussi appliqués à `Two`. (ex: `two.val === 'N'` à la fin du script)
- **Conclusion : liaison directe -> pas d'héritage, c'est une délégation directe avec une closure**


### Example 2-3

```javascript
// one
function One(){
  this.val = 'A';
}
One.prototype.fn = function(){
	return this.val;
}
var one = new One()
this.one = one;

// two
function Two(){
	One.call(this);
}
Two.prototype = Object.create(One.prototype);
Two.prototype.constructor = Two;
var two = new Two();
this.two = two;
```

![example2-3](/images/example2-3.png)

- Notes :
  - Le prototype de `one` est `One.prototype`.
  - Le prototype de `two` est `One.prototype`.
    - Tous les attributs et les fonctions de `One` ont été héritées par `Two`.
- **Conclusion : Héritage -> GOOD WAY**


## Conlusion

L'héritage classique (exemple 2-3) est un standard en OOP mais est-ce que l'héritage par concaténation (exemples 1-x) peut-il être réellement appellé héritage ?
J'aurais tendance à dire que non à cause du coté _aléatoire_ de cette technique.

Quelques articles intéressants :

- [Common Misconceptions About Inheritance in JavaScript](https://medium.com/javascript-scene/common-misconceptions-about-inheritance-in-javascript-d5d9bab29b0a)
- [Why Prototypal Inheritance Matters](http://aaditmshah.github.io/why-prototypal-inheritance-matters/)
