---
title: "Promesse de promesse, promis ?"
date: 2015-05-21 21:43:19 +0200
categories: [javascript, promise, promesse]
slug: promesse-de-promesse
---

Lors d'une conversation avec un collègue de travail ([Emmanuel Demey](http://gillespie59.github.io)), une question est venue :

Si une promesse prend comme argument de son `resolve` une promesse, que se passe t'il dans le `then` de la méthode appelante ?

Le resultat est une promesse ou une `String` ?

<!--more-->

## Le code

### Version native :

```javascript
var method01 = function () {
  return new Promise(function (resolve, reject) {
    console.log('start method01');

    setTimeout(function () {
      console.log('resolve method01');
      resolve('yeah');
    }, 5000);
  });
};

var method02 = function () {
  return new Promise(function (resolve, reject) {
    console.log('start method02');

    setTimeout(function () {
      console.log('resolve method02');
        resolve(method01());
    }, 1000);
  });
};

method02().then(function (data) {
    // La question : 'data' est une promesse ou une string ?
    console.log('then method2', data);
});
```

### Version avec `$q` d'AngularJS :

```javascript
var method01 = function () {
  console.log('start method01');

  var deferred = $q.defer();

  setTimeout(function () {
    console.log('resolve method01');
    deferred.resolve('yeah');
  }, 5000);

  return deferred.promise;
};

var method02 = function () {
  console.log('start method02');

  var deferred = $q.defer();

  setTimeout(function () {
    console.log('resolve method02');
    deferred.resolve(method01());
  }, 1000);

  return deferred.promise;
};

method02().then(function (data) {
    // La question : 'data' est une promesse ou une string ?
  console.log('then method2', data);
});
```

## La réponse

Réponse à la question : c'est une `String` !

Résultat :
```
start method02
resolve method02
start method01
resolve method01
then method2 yeah
```

## Pourquoi ?

Pourquoi *'then method2 yeah'* alors que `method02` est résolue avant `method01` ?


Dans la spécification : [point-49](https://promisesaplus.com/#point-49)
> If x is a promise, adopt its state [3.4]:
>
> - If x is pending, promise must remain pending until x is fulfilled or rejected.
> - If/when x is fulfilled, fulfill promise with the same value.
> - If/when x is rejected, reject promise with the same reason.


Donc `method02().then` attend la resolution de la promesse retournée par `method02`.

Hors cette promesse ne peut être résolue (fin de l'appel de la fonction `resolve`) que lorsque la méthode `method01` aura fini son traitement en étant résolue ou rejectée.

Pour être plus clair :
```javascript
resolve(promesse);
// équivalent à
promesse.then(...)
```

## Autres points notables

### Resolve & fonction

Il n'est pas possible de retourner une fonction en tant que promesse : si une fonction est passée dans le `resolve`, elle interprétée comme un thenable `function(resolve, reject) {...}`.

Voir la spécification : [point-56](https://promisesaplus.com/#point-56)

### Funny callback

```javascript
var method01 = function () {
  return new Promise(function (resolve, reject) {
    console.log('start method01');

    setTimeout(function () {
      console.log('resolve method01');
      resolve('yeah');
    }, 5000);
  });
};

method01.then(console.log); // Callback
```
