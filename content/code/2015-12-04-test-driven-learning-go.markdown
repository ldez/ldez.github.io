---
title: "Test Driven Learning - Go"
date: 2015-12-04 22:13:46 +0100
categories: [GoLang, Test, TDL]
slug: test-driven-learning-go
---

Go est un langage qui buzz beaucoup depuis quelques temps en grande partie à cause de [Docker](https://www.docker.com), j'ai donc eut l'envie de m'initier à ce langage.

Je me suis demandé comment commencer : faire des tutoriaux, souvent cela se résume à des copier-coller et finit par une connaissance très partielle.

Du coup, je me suis dit pourquoi ne pas essayer de faire le kata FizzBuzz pour apprendre.

C'est un kata très simple que je connais bien, je l'ai fait avec différentes contraintes en Java alors pourquoi ne pas le faire en Go ?

C'est parti !

<!--more-->

## Point de départ

Le site officiel de Go : https://golang.org.

Pour commencer, j'ai installé Go.

C'est plutôt simple, tout se trouve à cette adresse suivante : https://golang.org/dl.

Après il me fallait un minimum de documentation sur le langage.

N'étant pas un grand fan de StackOverflow, j'aime les sites contenant la documentation et les samples officiels.

J'ai trouvé 4 points d'entrées qui m'ont semblé pertinent :

- https://golang.org/doc
- https://golang.org/doc/effective_go.html
- https://golang.org/doc/code.html
- https://golang.org/ref/spec
- https://github.com/golang/go/wiki


## La structure

La question de base pour commencer correctement avec un langage c'est la structure d'un projet. Pour Go, il a été facile de trouver cela sur le site officiel : https://golang.org/doc/code.html

```
<GOPATH>
├── bin/
│   ├── hello                          # command executable
│   └── outyet                         # command executable
├── pkg/
│   └── linux_amd64/
│       └── github.com/golang/example/
│           └── stringutil.a           # package object
└── src/
    ├── github.com/golang/example/
    │   └── .git/                      # Git repository metadata
    ├── hello/
    │   └── hello.go                   # command source
    ├── outyet/
    │   ├── main.go                    # command source
    │   └── main_test.go               # test source
    └── stringutil/
        ├── reverse.go                 # package source
        └── reverse_test.go            # test source
```

Normalement lors de l'installation la variable d'environnement `GOROOT`, pointant vers les binaires de Go, a du être crée.

**IMPORTANT:** En plus de `GOROOT`, il faut définir la variable d'environnement `GOPATH` qui doit pointer vers la racine du workspace du projet (cf structure ci-dessus).
Plus d'informations : https://golang.org/doc/code.html#GOPATH


J'ai créé uniquement la structure suivante :

```
gohome
├── bin/
├── pkg/
└── src/
    └── fizzbuzz/
```

## Les tests

Après cela il me fallait comprendre comment fonctionnait les tests avec Go.

Je ne voulais pas trop rentrer dans le langage tant que je savais pas comment faire un test.

Une petite fouille sur le site de Go et le dépôt GitHub officiels et hop :

- https://github.com/golang/go/wiki/LearnTesting
- https://blog.golang.org/examples
- https://github.com/golang-samples/testing
- https://golang.org/pkg/testing

Un point important dans Go est le fonctionnement de sa bibliothèque de tests :

*Citation, https://golang.org/doc/faq#assertions*
> Why does Go not have assertions?
>
> Go doesn't provide assertions.
> They are undeniably convenient, but our experience has been that programmers use them as a crutch to avoid thinking about proper error handling and reporting.
> Proper error handling means that servers continue operation after non-fatal errors instead of crashing.
> Proper error reporting means that errors are direct and to the point, saving the programmer from interpreting a large crash trace.
> Precise errors are particularly important when the programmer seeing the errors is not familiar with the code.
>
> We understand that this is a point of contention.
> There are many things in the Go language and libraries that differ from modern practices, simply because we feel it's sometimes worth trying a different approach.

Il n'y a donc pas d'assertions de base dans Go mais juste une structure permettant _d'enregistrer les erreurs_.

J'ai trouvé cela dans la documentation de Go :

*https://golang.org/doc/code.html#Testing*
```go
package stringutil

import "testing" // <1>

func TestReverse(t *testing.T) { // <2>

  for _, c := range []struct {
    in, want string
  }{
    {"Hello, world", "dlrow, olleH"},
    {"Hello, 世界", "界世, olleH"},
    {"", ""},
  } {
    got := Reverse(c.in)
    if got != c.want {
      t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want) // <3>
    }
  }
}
```
- (1) importation de la bibliothèque de tests
- (2) paramètre de la méthode pour accéder au framework de tests
- (3) émission d'une erreur (échec du test)

Dans la documentation, il est aussi dit que les fichiers de tests doivent se terminer par `_test.go` et que chaque méthode de test doit avoir la forme `TestXXX`.

### Couverture de code

Pour compléter les tests, j'avais aussi besoin de connaître la couverture du code par les tests.

Après quelques recherches sur le blog de Go, j'ai trouvé cet article : [The cover story](https://blog.golang.org/cover)

Pour voir la couverture :

```bash
go test -cover
```

Il est possible de créer un rapport avec les statistiques de couverture du code (coverage profile) :

```bash
go test -coverprofile=coverage.out
```

Le flag `-coverprofile` définit automatiquement le flag `-cover` afin d'activer l'analyse de la couverture.

Il est possible jouer sur les données du rapport grâce à l'option `-covermode`.

```
-covermode <option>
```

- `set`: est-ce chaque instruction est exécutée ? (par défaut)
- `count`: combien de fois chaque instruction est exécutée ?
- `atomic`: comme `count`, mais compte précisément dans les programmes parallélisés

Pour visualiser le rapport généré pour chaque fonction :

```bash
go tool cover -func=coverage.out
```

Pour visualiser le rapport sous une forme plus visuelle, il est possible produire une page HTML.

```bash
go tool cover -html=coverage.out
```

Cette commande génère un fichier HTML dans le dossier temporaire de l'utilisateur et l'ouvre dans le navigateur par défaut.

Pour voir l'aide de `tool cover` :

```bash
go tool cover
```

Voilà je me sens prêt à commencer.

Je ne connais pas encore le langage mais je sais comment écrire et éxéxuter un test !


## IDE

Pour pouvoir écrire du code, un bon outil c'est important.

Donc je suis parti à la recherche d'un IDE pour Go.

Premier test avec mes IDE habituels, car oui j'utilise plusieurs IDE.

Donc j'ouvre Brackets, IntelliJ et Eclipse à la recherche du support langage.

Je ne trouve rien de très pertinent.

Je suppose que VIM doit avoir un support mais n'étant pas un grand habitué de cet éditeur je décide de continuer à chercher.

Je me retrouve sur le site officiel puis le GitHub et je trouve cette page :

- https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins

Je décide donc de tester [LiteIDE](https://github.com/visualfc/liteide).
Super autocomplétion des bibliothèques, le style de l'IDE est moche mais bon s'il va bien c'est OK.
Mais *fail* car le support du clavier est destiné uniquement au clavier QWERTY.

Donc c'est reparti pour tester un nouvel IDE.

J'avais déjà installé [Atom](https://atom.io) donc je regarde les plugins disponible et surprise :

- [language-go](https://atom.io/packages/language-go)
- [go-plus](https://atom.io/packages/go-plus) : `apm install go-plus`

`language-go` permet le support de la coloration syntaxique de Go. Fournit par défaut avec Atom.

`go-plus` permet d'avoir de l'auto-complétion, le formateur officiel, du linting et donne un retour visuel des erreurs de syntaxe. Un super plugin !

`go-plus` nécessite l'installation de [Gocode](https://github.com/nsf/gocode) pour l'autocompletion.

J'ai donc arrêté mon choix sur Atom.


## Le langage

Maintenant, il faut mettre les mains dans le cambouis !

Les commentaires sont assez classiques : `// mon commentaire` et `/* mon commentaire */`

Après quelques lectures de la documentation et de quelques samples, je comprends que Go est un langage fonctionnel où les fonctions sont simplement exposée par le package.

J'ai décidé de créer un premier fichier `fizzbuzz.go`.

A l'aide de la documentation j'ai obtenu ceci :

*fizzbuzz.go*
```go
package kata

import "fmt"

// FizzBuzz main function
func FizzBuzz() {
  fmt.Println("Kata FizzBuzz")
}
```

C'est un fichier simple qui affiche `Kata FizzBuzz` dans la console.

Je tente :

```bash
go run fizzbuzz.go
```

Mais cela ne marche pas :

```
go run: cannot run non-main package
```

Ok j'ai fait n'importe quoi en voulant lancer ce fichier.
Pour que cela marche j'aurais du définir le package de mon fichier comme étant `main` et créer une fonction nommé `main` :

*fizzbuz.go*
```go
package main

import "fmt"

// FizzBuzz main function
func main() {
  fmt.Println("Kata FizzBuzz")
}
```

Mais mon premier objectif n'était pas de faire tourner une ligne de commande mais des tests afin de pouvoir faire le kata.

J'ai donc gardé le premier contenu du fichier `fizzbuzz.go` et créé un nouveau fichier `fizzbuzz_test.go`.

Pour cela, il me faut déclarer des variables, je regarde les sources en exemples dans le GitHub officiel.

Je comprends que les variables sont déclarées sous la forme :

```go
var foo string
```

Et qu'il est possible, lors d'une déclaration avec assignation, d'utiliser une notation réduite :

```go
foo := "bar"
```

- pas besoin de `var`
- pas besoin de définir le type

J'écris donc mon premier test.

*fizzbuz_test.go*
```go
package kata

import "testing"

func Test_Should_TODO(t *testing.T) {

  FizzBuzz()

  actual := "0"
  expected := "1"

  if actual != expected {
    t.Errorf("Expected %s but was %s", expected, actual)
  }
}
```

J'ai simplement appelé la fonction `FizzBuzz` dans le test  et lancé le test :

```bash
go test
```

La sortie de console était plutôt positive :

- mon message `Kata FizzBuzz` s'affichait bien
- mon test échouait ce qui était mon objectif.

```
Kata FizzBuzz
--- FAIL: Test_Should_TODO (0.00s)
        fizzbuzz_test.go:13: Expected 1 but was 0
FAIL
exit status 1
FAIL    /mysources/kata/fizz-buzz/go        0.084s
```

J'ai une idée très partielle du langage mais c'est parti pour le kata.


## Le kata

Mon objectif est bien sur de faire ce kata en TDD avec des *baby steps*.


### Description du kata

> Afficher les chiffres de 1 to 100.
>
> Pour les multiples de trois afficher `Fizz`.
>
> Pour les multiples de cinq afficher `Buzz`.
>
> Pour les multiples de trois et de cinq afficher `FizzBuzz`.


### Étape 1 - Afficher un chiffre

Je connais bien ce kata donc je commence par écrire un test simple pour écrire un chiffre dans la console.

*fizzbuz_test.go*
```go
package kata

import "testing"

func Test_should_print_string_representation_when_passing_an_integer(t *testing.T) {

    rt := Display(1)

    expected := "1"
    if rt != expected {
        t.Errorf("Must display %s but display %s", expected, rt)
    }
}
```

Voilà le test est écrit mais il est rouge : la fonction `Display` n'existe pas.

Pour créer cette méthode je dois pouvoir passer un paramètre à une fonction et convertir un `int` en `string`.

Encore un retour à la documentation.

La déclaration d'un paramètre d'une fonction se fait en écrivant le nom du paramètre puis son type.
Il possible de définir le type de retour en ajoutant le type après la déclaration de la méthode.

```go
func Foo(bar int) string {
  // ...
}
```

Le deuxième point est plus compliqué car je ne vois pas comment convertir un `int` en `string`.

Je ne trouve rien rapidement dans la documentation donc Google est mon ami.

Ce n'est pas très intuitif car il faut importer le package `strconv` et utiliser la méthode `strconv.Itoa()`.

Je crée la méthode `Display` dans `fizzbuz.go`.

*fizzbuz.go*
```go
package kata

import "fmt"
import (
    "fmt"
    "strconv"
)

// FizzBuzz main function
func FizzBuzz() {
    fmt.Println("Kata FizzBuzz")
}

// Display number
func Display(number int) string {
    return strconv.Itoa(number)
}
```

Je lance les tests (`go test`) et c'est bon mon test est vert, victoire !


### Étape 2 - Multiple de 3

Bon maintenant, je vais devoir ajouter un nouveau test pour afficher `Fizz` lorsque le chiffre est `3`.

Écrire le test est assez simple.

*fizzbuz_test.go*
```go
// Pour les multiples de trois afficher "Fizz".
func Test_should_print_Fizz_when_passing_3(t *testing.T) {

    rt := Display(3)

    expected := "Fizz"
    if rt != expected {
        t.Errorf("Must display %s but display %s", expected, rt)
    }
}
```

Je lance les tests (`go test`) et ce nouveau test est rouge.

Je crée un code stupide mais qui fait passer le test.

Pour cela j'ai besoin d'écrire un `if`, je l'ai déjà fait plus haut mais en copiant du code.

Je comprends en écrivant cette partie que les parenthèses sont optionnelles. Le formateur de Go les supprime automatiquement.

*fizzbuz.go*
```go
// Display number
func Display(number int) string {

    if number == 3
        return "Fizz"
    return strconv.Itoa(number)
}
```

Les accolades pour les `if` sont optionnelles, pour l'instant je n'en met pas (je ne suis pas fan du code inline ou sans accolades, je trouve que cela manque de lisibilité et augmente le risque d'erreurs lors des évolutions).

Je relance les tests (`go test`) et ce test est vert, encore une victoire !


### Étape 3 - Multiple de 3

Afin de faire une implémentation plus correcte j'ajoute un test pour le chiffre `6`.

*fizzbuz_test.go*
```go
// Pour les multiples de trois afficher "Fizz".
func Test_should_print_Fizz_when_passing_6(t *testing.T) {

    rt := Display(6)

    expected := "Fizz"
    if rt != expected {
        t.Errorf("Must display %s but display %s", expected, rt)
    }
}
```

Je lance les tests (`go test`) et ce nouveau test est rouge.

Maintenant je fais un refactor de l'implémentation afin de traiter ce nouveau cas.

*fizzbuz.go*
```go
// Display number
func Display(number int) string {

    if number % 3 == 0
        return "Fizz"
    return strconv.Itoa(number)
}
```

Je relance les tests (`go test`) et tous les tests sont verts, encore une victoire !


### Étape 4 - Multiple de 5

J'ajoute un nouveau test pour afficher `Buzz` lorsque le chiffre est `5`.

*fizzbuz_test.go*
```go
// Pour les multiples de cinq afficher "Buzz".
func Test_should_print_Buzz_when_passing_5(t *testing.T) {

    rt := Display(5)

    expected := "Buzz"
    if rt != expected {
        t.Errorf("Must display %s but display %s", expected, rt)
    }
}
```

Je lance les tests (`go test`) et ce nouveau test est rouge.

Suivi de l'implémentation très simpliste :

*fizzbuz.go*
```go
// Display number
func Display(number int) string {

    if number % 3 == 0
        return "Fizz"
    if number == 5
        return "Buzz"
    return strconv.Itoa(number)
}
```

Je relance les tests (`go test`) et tous les tests sont verts, encore une victoire !


### Étape 5 - Multiple de 5

Même travail que pour les étapes 3 et 4.

*fizzbuz_test.go*
```go
// Pour les multiples de cinq afficher "Buzz".
func Test_should_print_Buzz_when_passing_20(t *testing.T) {

    rt := Display(20)

    expected := "Buzz"
    if rt != expected {
        t.Errorf("Must display %s but display %s", expected, rt)
    }
}
```

Je lance les tests (`go test`) et ce nouveau test est rouge.

*fizzbuz.go*
```go
// Display number
func Display(number int) string {

    if number % 3 == 0
        return "Fizz"
    if number % 5 == 0
        return "Buzz"
    return strconv.Itoa(number)
}
```

Je relance les tests (`go test`) et tous les tests sont verts, encore une victoire !


### Étape 6 - Multiple de 3 et 5

Ajout d'un nouveau test pour les multiples de trois et de cinq qui doivent se transformer en "FizzBuzz".

*fizzbuz_test.go*
```go
// Pour les multiples de trois et de cinq afficher "FizzBuzz".
func Test_should_print_FizzBuzz_when_passing_15(t *testing.T) {

    rt := Display(15)

    expected := "FizzBuzz"
    if rt != expected {
        t.Errorf("Must display %s but return is %s", expected, rt)
    }
}
```

Première phase : aller au vert le plus rapidement.

*fizzbuz.go*
```go
// Display number
func Display(number int) string {

    if number % 3 == 0 && number % 5 == 0
        return "FizzBuzz"
    if number % 3 == 0
        return "Fizz"
    if number % 5 == 0
        return "Buzz"
    return strconv.Itoa(number)
}
```

Les tests sont verts mais il y a beaucoup de duplication dans le code donc je dois factoriser le code en créant une méthode `IsMultipleOf`.

*fizzbuz.go*
```go
// IsMultipleOf divisor for number
func IsMultipleOf(divisor int, number int) bool {
    return number%divisor == 0
}
```

Ce qui me permet d'écrire la méthode `Display` ainsi :

*fizzbuz.go*
```go
// Display number
func Display(number int) string {

    if IsMultipleOf(3, number) && IsMultipleOf(5, number) {
        return "FizzBuzz"
    }
    if IsMultipleOf(3, number) {
        return "Fizz"
    }
    if IsMultipleOf(5, number) {
        return "Buzz"
    }
    return strconv.Itoa(number)
}
```

Les tests sont verts mais je ne suis pas très fan de la répétition donc :

*fizzbuz.go*
```go
// Display number
func Display(number int) string {

    fizz := IsMultipleOf(3, number)
    buzz := IsMultipleOf(5, number)

    if fizz && buzz {
        return "FizzBuzz"
    }
    if fizz {
        return "Fizz"
    }
    if buzz {
        return "Buzz"
    }

    return strconv.Itoa(number)
}
```

Les tests sont verts, avant de passer à l'étape suivante j'ajoute un test pour vérifier que la règle fonctionne bien :

*fizzbuz_test.go*
```go
// Pour les multiples de trois et de cinq afficher "FizzBuzz".
func Test_should_print_FizzBuzz_when_passing_30(t *testing.T) {

    rt := Display(30)

    expected := "FizzBuzz"
    if rt != expected {
        t.Errorf("Must display %s but return is %s", expected, rt)
    }
}
```

Les tests sont verts, c'est parti pour la dernière étape.


### Etape 7 - de 1 à 100

Il ne me reste plus qu'à traiter une séquence de 1 à 100.

Pour cela j'ajoute un test :

*fizzbuz_test.go*
```go
// Afficher les chiffres de 1 to 100.
func Test_should_display_a_valid_result_when_diplay_numbers_between_1_and_100(t *testing.T) {

    rt := FizzBuzz(100)

    expected := "1\n2\nFizz\n4\nBuzz\nFizz\n7\n8\nFizz\nBuzz\n11\nFizz\n13\n14\nFizzBuzz\n16\n17\nFizz\n19\nBuzz\nFizz\n22\n23\nFizz\nBuzz\n26\nFizz\n28\n29\nFizzBuzz\n31\n32\nFizz\n34\nBuzz\nFizz\n37\n38\nFizz\nBuzz\n41\nFizz\n43\n44\nFizzBuzz\n46\n47\nFizz\n49\nBuzz\nFizz\n52\n53\nFizz\nBuzz\n56\nFizz\n58\n59\nFizzBuzz\n61\n62\nFizz\n64\nBuzz\nFizz\n67\n68\nFizz\nBuzz\n71\nFizz\n73\n74\nFizzBuzz\n76\n77\nFizz\n79\nBuzz\nFizz\n82\n83\nFizz\nBuzz\n86\nFizz\n88\n89\nFizzBuzz\n91\n92\nFizz\n94\nBuzz\nFizz\n97\n98\nFizz\n"

    if rt != expected {
        t.Errorf("Must display %s but return is %s", expected, rt)
    }
}
```

Le test est rouge, le code ne compile pas, il faut que je modifie la méthode `FizzBuzz`.

J'ai besoin de faire un `for`, je regarde la documentation, c'est comme en Java et je ne vois pas de générateur de 'range'.

*fizzbuz.go*
```go
// FizzBuzz main function
func FizzBuzz(max int) string {
    fmt.Println("Kata FizzBuzz")

    var result string

    for i := 1; i < max; i++ {
        result += Display(i) + "\n"
    }
    return result
}
```

Tout les tests sont verts, c'est la victoire ultime !


## Et après ?

J'ai fini le kata, cela m'a pris 3h, j'ai appris beaucoup de choses sur le langage, surtout sur la philosophie derrière le langage.

Pour aller plus loin, je vais devoir faire un kata plus complexe ou débuter une application (qui sera une forme de kata).

L'exploration des bibliothèques va aussi être un point important :

- explorer les bibliothèques externes de tests comme [GoConvey](http://goconvey.co) ou [GoCheck](https://labix.org/gocheck) ou [Testify](https://github.com/stretchr/testify) ou [GoCov](https://github.com/axw/gocov)
- avoir une meilleure vision d'ensemble des bibliothèques internes de Go
- trouver les bonnes bibliothèques externes pour les cas commun

Je vais aussi regarder des applications existantes, des articles, et des présentations, etc. afin d'approfondir ma compréhension.

Je vais sûrement faire _un tour de Go_ : https://tour.golang.org/welcome/

Vous trouverez le code de résultant de cette article ici : https://github.com/ldez/tdl-go
