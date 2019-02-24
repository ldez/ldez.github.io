---
title: "Kata Print Diamond"
date: 2015-08-22 22:22:38 +0200
categories: [java, kata, tdd]
slug: kata-print-diamond
---

Ce kata définit par [Seb Rose](http://claysnow.co.uk/recycling-tests-in-tdd/) a pour but de montrer une approche visant à recycler les tests afin de ne pas créer de phase tunnel de refactor long et massif.

J'ai donc respecté l'approche TDD pour l'émergence du design et fait des '*baby step*' en créant des tests pour chacune des étapes afin d'avoir une *victoire* à chaque fois. (Il est très important d'avoir rapidement un retour positif pour ce que l'on fait)


## Description

Étant donné une lettre, imprimer un diamant en commençant par "A" avec la lettre fournie pour point le plus large.

Par exemple `print-diamond ‘C’` affiche :

```
  A
 B B
C   C
 B B
  A
```

<!--more-->

## Le déroulé

### Etape 1

A la lecture du kata je pense comprendre que la lettre de départ du Diamant (`A`) est une constante.

Je créé la classe `DiamondTest` et un test pour définir la constante `ROOT_LETTER`.
Cela me permet de créer la classe `Diamond`.

Tests unitaires :

```java
public class DiamondTest {

    @Test
    public void should_have_A_as_root_letter() throws Exception {
        assertEquals(Diamond.ROOT_LETTER, 'A');
    }

}
```

Implémentation :

```java
public class Diamond {

    public static final char ROOT_LETTER = 'A';

}
```


### Etape 2

Je défini un test pour le cas de lettre `A` et implémente la méthode `print`.
La représentation du 'diamant' pour `A` étant une seule lettre sans espace cela est très facile.
Pour l'instant le code de l'implémentation est stupide.

Tests unitaires :

```java
@Test
public void should_print_A_when_widest_letter_is_A() throws Exception {
    final Diamond diamond = new Diamond();

    final String display = diamond.print('A');

    assertEquals("A", display);
}
```

Implémentation :

```java
public class Diamond {

    public static final char ROOT_LETTER = 'A';

    public final String print(char widestLetter) {
      return "A";
    }
}
```


### Etape 3

Le résultat final (afficher le Diamant) me parait trop complexe pour pouvoir définir tout de suite un test unitaire pertinent.

Je décide donc :

- de créer une classe de tests d'intégration avec 2 cas : `A` et `B`.
- de commencer par essayer de créer la suite de caractères attendu, sans les répétitions des lettres, uniquement la suite de lettre qui composera chaque ligne.

Évidemment les tests d'acceptation resterons rouge jusqu'à ce qu'à la résolution complète du kata.

Tests d'acceptation :

```java
public class DiamondAcceptanceTest {

    @Test
    public void should_print_ABBA_when_widest_letter_is_B() throws Exception {
        final Diamond diamond = new Diamond();

        final String display = diamond.print('B');

        String expected = " A \n"
                        + "B B\n"
                        + " A ";
        assertEquals(expected, display);
    }

}
```

Je défini un test pour le cas de lettre `B` avec pour objectif d'écrire `ABA` et je refactore la méthode `print`.
Pour l'instant le code de la méthode est toujours stupide.

Tests unitaires :

```java
@Test
public void should_print_ABA_when_widest_letter_is_B() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('B');

    String expected = "ABA";
    assertEquals(expected, display);
}
```

Implémenation :

```java
public final String print(char widestLetter) {
  if ('B' == widestLetter) {
		return "ABA";
	}
	return "A";
}
```

Tous les tests sont verts sauf les tests d'acceptation.


### Etape 4

Je recommence comme à l'étape précédente mais avec les lettres `C` et `D`.

J'ai ajouté chaque test puis fait l'implémentation relative à celui ci, afin de ne pas être trop répétitif dans cette article j'ai factorisé l'explication les cas des lettres `C` et `D`.

Tests d'acceptation :

```java
@Test
public void should_print_ABCCBA_when_widest_letter_is_C() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('C');

    String expected = "  A  \n"
                    + " B B \n"
                    + "C   C\n"
                    + " B B \n"
                    + "  A  ";
    assertEquals(expected, display);
}

@Test
public void should_print_ABCDDCBA_when_widest_letter_is_D() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('D');

    String expected = "   A   \n"
                    + "  B B  \n"
                    + " C   C \n"
                    + "D     D\n"
                    + " C   C \n"
                    + "  B B  \n"
                    + "   A   ";
    assertEquals(expected, display);
}
```

Tests unitaires :

```java
@Test
public void should_print_ABCBA_when_widest_letter_is_C() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('C');

    String expected = "ABCBA";
    assertEquals(expected, display);
}

@Test
public void should_print_ABCDCBA_when_widest_letter_is_D() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('D');

    String expected = "ABCDCBA";
    assertEquals(expected, display);
}
```

Implémentation :

```java

public final String print(char widestLetter) {
	if ('D' == widestLetter) {
        return "ABCDCBA";
    }
    if ('C' == widestLetter) {
        return "ABCBA";
    }
    if ('B' == widestLetter) {
        return "ABA";
    }
    return "A";
}
```

Tous les tests sont verts sauf les tests d'acceptation.


### Etape 5

Le code de l'implémentation est stupide mais assez complet pour commencer un refactor du code.

Pour cela je vais changer de paradigme :

- au lieu de comparer chaque lettre à `A`, je vais comparer la distance de chaque lettre par rapport à `A`.
- pourquoi ? parce je trouve plus simple de manipuler des chiffres afin de visualiser les duplications possibles.

Implémentation :

```java
public final String print(final char widestLetter) {

    // distance between ROOT_LETTER and the widest Letter
    final int distanceWithRootLetter = Character.compare(widestLetter, ROOT_LETTER);

    if (3 == distanceWithRootLetter) {
        return "ABCDCBA";
    }
    if (2 == distanceWithRootLetter) {
        return "ABCBA";
    }
    if (1 == distanceWithRootLetter) {
        return "ABA";
    }
}
```

Tous les tests sont verts sauf les tests d'acceptation.


### Etape 6

Le code de l'implémentation est toujours stupide, je vais donc factoriser le code pour en sortir un code plus générique.

Pour cela je me dis qu'une suite de lettre peut être vu comme une suite de distance par rapport à `A`.

```javascript
['A', 'B', 'A'] -> [0, 1, 0]
['A', 'B', 'C', 'B', 'A'] -> [0, 1, 2, 1, 0]
['A', 'B', 'C', 'D', 'C', 'B', 'A'] -> [0, 1, 2, 3, 2, 1, 0]
```

L'objectif maintenant est de créer une séquence de chiffre représentant cette suite.

Je connais la distance de la lettre la plus lointaine avec `A` (que j'ai nommé `distanceWithRootLetter` lors de la précédente étape).

Créer une suite de ce style c'est assez simple, il suffit de créer une suite qui va de `-distanceWithRootLetter` à `distanceWithRootLetter` puis d'ajouter `distanceWithRootLetter` à la valeur absolue de chaque valeur de la suite.

Par exemple pour B :

```javascript
['A', 'B', 'A'] -> [0, 1, 0]
```

`distanceWithRootLetter = 1`

```javascript
[
  -distanceWithRootLetter,
  0,
  distanceWithRootLetter
]
= [-1, 0, 1] = [i1, i2, i3]
->
[
  distanceWithRootLetter - |i1|,
  // avec i1 = -distanceWithRootLetter
  // donc maintenant la valeur de cette case dans la ligne est de 0.
  distanceWithRootLetter - |i2|,
  // avec i2 = 0
  // donc maintenant la valeur de cette case dans la ligne est de 1.
  distanceWithRootLetter - |i3|
  // avec i3 = distanceWithRootLetter
  // la valeur de cette case dans la ligne est de 0.
]
= [0, 1 , 0]
```

Et voilà il ne me reste plus qu'à ajouter cette distance à `A` pour obtenir la bonne suite de lettre.

Implémentation :

```java
public final String print(final char widestLetter) {

    // distance between ROOT_LETTER and the widest Letter
    final int distanceWithRootLetter = Character.compare(widestLetter, ROOT_LETTER);

    return IntStream.rangeClosed(-distanceWithRootLetter, distanceWithRootLetter)
            // compute distance
            .map(number -> distanceWithRootLetter - Math.abs(number))
            // convert distance to character
            .mapToObj(distance -> (char) (ROOT_LETTER + distance))
            // convert to String
            .map(String::valueOf)
            // aggregate letters
            .collect(Collectors.joining());
}
```

Tous les tests sont toujours verts sauf les tests d'acceptation.


### Etape 7

Maintenant que je peux générer une suite de lettre correspondante à chacune des lettres attendu sur chaque ligne je vais ajouter des sauts de lignes.

Je recycle donc les tests pour vérifier cela.

Tests unitaires :

```java
@Test
public void should_print_ABA_when_widest_letter_is_B() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('B');

    String expected = "A\nB\nA";
    assertEquals(expected, display);
}

@Test
public void should_print_ABCBA_when_widest_letter_is_C() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('C');

    String expected = "A\nB\nC\nB\nA";
    assertEquals(expected, display);
}

@Test
public void should_print_ABCDCBA_when_widest_letter_is_D() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('D');

    String expected = "A\nB\nC\nD\nC\nB\nA";
    assertEquals(expected, display);
}
```

Cela est rendu très simple par l'utilisation des `Stream` car il suffit de donner le paramètre `\n` à la fonction `joining`.

Implémentation :

```java
public final String print(final char widestLetter) {

    // distance between ROOT_LETTER and the widest Letter
    final int distanceWithRootLetter = Character.compare(widestLetter, ROOT_LETTER);

    return IntStream.rangeClosed(-distanceWithRootLetter, distanceWithRootLetter)
            // compute distance
            .map(number -> distanceWithRootLetter - Math.abs(number))
            // convert distance to character
            .mapToObj(distance -> (char) (ROOT_LETTER + distance))
            // convert to String
            .map(String::valueOf)
            // aggregate letters
            .collect(Collectors.joining("\n"));
}
```

Tous les tests sont verts sauf les tests d'acceptation.


### Etape 8

J'ai obtenu une colonne de caractères correspondant aux caractères attendu sur chaque ligne.

Je dois maintenant doubler chaque caractère sauf les extrémités contenant la lettre `A` une seule fois.

Je recycle donc encore les tests pour vérifier cela.

Tests unitaires :

```java
@Test
public void should_print_ABA_when_widest_letter_is_B() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('B');

    String expected = "A\nBB\nA";
    assertEquals(expected, display);
}

@Test
public void should_print_ABCBA_when_widest_letter_is_C() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('C');

    String expected = "A\nBB\nCC\nBB\nA";
    assertEquals(expected, display);
}

@Test
public void should_print_ABCDCBA_when_widest_letter_is_D() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('D');

    String expected = "A\nBB\nCC\nDD\nCC\nBB\nA";
    assertEquals(expected, display);
}
```

Pour l'implémentation je choisi de représenter ma ligne sous la forme d'un tableau pour ne pas avoir à transformer chacun des caractères en une `String` pour de les agréger.

En plus la méthode `String.valueOf` sait transformer un tableau de caractères en `String`.
Cette décision pourra être revue mais à cette étape c'est la manière la plus simple et la plus rapide.

Implémentation :

```java
public final String print(final char widestLetter) {

    // distance between ROOT_LETTER and the widest Letter
    final int distanceWithRootLetter = Character.compare(widestLetter, ROOT_LETTER);

    return IntStream.rangeClosed(-distanceWithRootLetter, distanceWithRootLetter)
            // compute distance
            .map(number -> distanceWithRootLetter - Math.abs(number))
            // convert distance to character
            .mapToObj(distance -> (char) (ROOT_LETTER + distance))
            // build a line
            .map(letter -> {
                if (ROOT_LETTER == letter) {
                    return new char[] { letter };
                }
                return new char[] { letter, letter };
            })
            // convert to String
            .map(String::valueOf)
            // aggregate letters
            .collect(Collectors.joining("\n"));
}
```

Tous les tests sont verts sauf les tests d'acceptation.


### Etape 9

Maintenant j'ai le bon caractère sur chaque ligne et le bon nombre de fois le caractère.

Je recycle donc encore les tests pour créer des lignes de la bonne taille.

Tests unitaires :

```java
@Test
public void should_print_ABBA_when_widest_letter_is_B() throws Exception {
    final Diamond diamond = new Diamond();

    final String display = diamond.print('B');

    String expected = "A  \n"
                    + "BB \n"
                    + "A  ";
    assertEquals(expected, display);
}

@Test
public void should_print_ABCCBA_when_widest_letter_is_C() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('C');

    String expected = "A    \n"
                    + "BB   \n"
                    + "CC   \n"
                    + "BB   \n"
                    + "A    ";
    assertEquals(expected, display);
}

@Test
public void should_print_ABCDDCBA_when_widest_letter_is_D() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('D');

    String expected = "A      \n"
                    + "BB     \n"
                    + "CC     \n"
                    + "DD     \n"
                    + "CC     \n"
                    + "BB     \n"
                    + "A      ";
    assertEquals(expected, display);
}
```

Je choisi de conserver la représentation de la ligne sous forme de tableau afin de positionner les caractères dans une ligne au lieu de faire une `String` et de faire des agrégations de caractère Espace.

Implémentation :

```java
public final String print(final char widestLetter) {

    // distance between ROOT_LETTER and the widest Letter
    final int distanceWithRootLetter = Character.compare(widestLetter, ROOT_LETTER);

    // square size : total number of rows == length of a line
    final int squareSize = distanceWithRootLetter * 2 + 1;

    return IntStream.rangeClosed(-distanceWithRootLetter, distanceWithRootLetter)
            // compute distance
            .map(number -> distanceWithRootLetter - Math.abs(number))
            // convert distance to character
            .mapToObj(distance -> (char) (ROOT_LETTER + distance))
            // build a line
            .map(letter -> {
                final char[] line = new char[squareSize];
                // Fill for replace 'null' character with space character
                Arrays.fill(line, ' ');

                if (ROOT_LETTER == letter) {
                    line[0] = letter;
                } else {
                    line[0] = letter;
                    line[1] = letter;
                }
                return line;
            })
            // convert to String
            .map(String::valueOf)
            // aggregate letters
            .collect(Collectors.joining("\n"));
}
```

Tous les tests sont verts sauf les tests d'acceptation.


### Etape 10

J'ai le bon caractère sur chaque ligne et le bon nombre de fois le caractère et des lignes de la bonne taille, il ne me reste plus qu'à placer les caractères afin de former le diamant.

Je modifie donc les tests pour vérifier cela.

Tests unitaires :

```java
@Test
public void should_print_ABBA_when_widest_letter_is_B() throws Exception {
    final Diamond diamond = new Diamond();

    final String display = diamond.print('B');

    String expected = " A \n"
                    + "B B\n"
                    + " A ";
    assertEquals(expected, display);
}

@Test
public void should_print_ABCCBA_when_widest_letter_is_C() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('C');

    String expected = "  A  \n"
                    + " B B \n"
                    + "C   C\n"
                    + " B B \n"
                    + "  A  ";
    assertEquals(expected, display);
}

@Test
public void should_print_ABCDDCBA_when_widest_letter_is_D() throws Exception {
    Diamond diamond = new Diamond();

    String display = diamond.print('D');

    String expected = "   A   \n"
                    + "  B B  \n"
                    + " C   C \n"
                    + "D     D\n"
                    + " C   C \n"
                    + "  B B  \n"
                    + "   A   ";
    assertEquals(expected, display);
}
```

Implémentation :

```java
public final String print(final char widestLetter) {

    // distance between ROOT_LETTER and the widest Letter
    final int distanceWithRootLetter = Character.compare(widestLetter, ROOT_LETTER);

    // square size : total number of rows == length of a line
    final int squareSize = distanceWithRootLetter * 2 + 1;

    return IntStream.rangeClosed(-distanceWithRootLetter, distanceWithRootLetter)
            // compute distance
            .map(number -> distanceWithRootLetter - Math.abs(number))
            // convert distance to character
            .mapToObj(distance -> (char) (ROOT_LETTER + distance))
            // build a line
            .map(letter -> {
                final char[] line = new char[squareSize];
                // Fill for replace 'null' character with space character
                Arrays.fill(line, ' ');

                // distance between current letter and root letter
                final int distance = Character.compare(letter, ROOT_LETTER);

                if (ROOT_LETTER == letter) {
                    line[distanceWithRootLetter] = letter;
                } else {
                    line[distanceWithRootLetter - distance] = letter;
                    line[distanceWithRootLetter + distance] = letter;
                }
                return line;
            })
            // convert to String
            .map(String::valueOf)
            // aggregate letters
            .collect(Collectors.joining("\n"));
}
```

Mes tests ressemblent exactement aux tests d'acceptation, je supprime donc ma classe de tests pour ne garder que les tests d'acceptation.

Tous les tests sont verts !!


### Etape 11

Je sens qu'il y a une forme de duplication entre le positionnement du caractère pour le première ligne et le premier caractère d'une ligne du cœur du diamant.

Un peu de factorisation.

Implémentation :

```java
public final String print(final char widestLetter) {

    // distance between ROOT_LETTER and the widest Letter
    final int distanceWithRootLetter = Character.compare(widestLetter, ROOT_LETTER);

    // square size : total number of rows == length of a line
    final int squareSize = distanceWithRootLetter * 2 + 1;

    return IntStream.rangeClosed(-distanceWithRootLetter, distanceWithRootLetter)
            // compute distance
            .map(number -> distanceWithRootLetter - Math.abs(number))
            // convert distance to character
            .mapToObj(distance -> (char) (ROOT_LETTER + distance))
            // build a line
            .map(letter -> {
                final char[] line = new char[squareSize];
                // Fill for replace 'null' character with space character
                Arrays.fill(line, ' ');

                // distance between current letter and root letter
                final int distance = Character.compare(letter, ROOT_LETTER);

                // put the first letter in the line
                line[distanceWithRootLetter - distance] = letter;

                // if is not the first line, put the second letter
                if (ROOT_LETTER != letter) {
                    line[distanceWithRootLetter + distance] = letter;
                }
                return line;
            })
            // convert to String
            .map(String::valueOf)
            // aggregate letters
            .collect(Collectors.joining("\n"));
}
```

Tous les tests sont toujours verts.


## Conclusion

L'erreur classique avec ce kata est de créer 4 tests correspondant à la création finale du diamant pour les lettres `A`, `B`, `C` et `D` puis de rentrer dans une phase de refactor massif.
Jusqu'à la fin du kata tous les tests échouerons, cela induit que le design ne va pas émerger poussé par les tests.

C'est pour cela que j'ai volontairement refactoré/recyclé mes propres tests.

Le fait d'avoir fait des tests puis de les recycler m'a permit :

- d'avoir des objectifs de petite taille (*baby step*).
- de créer du code uniquement pour répondre à un test et donc de faire émerger le design.
- d'être récompensé à chaque étape en ayant des tests qui sont passés du rouge au vert à chaque fois.

Vous pourriez vous demander si nous ne risquons pas de produire des régressions en recyclant des tests comme cela.

Dans ce cas précis nous ne courrons aucun risque :
En plus de faire un commit à chaque étape, et donc de pouvoir revenir facilement sur la précédente version avec des tests verts, chaque étape procède comme l'ajout d'une nouvelle fonctionnalité.

Cela est comparable à l'évolution d'une application : certaines règles métier peuvent changer.

Il n'est pas nécessaire de conserver le contenu des précédents tests car ils ne seront plus jamais valides.
Il faut donc faire évoluer les tests correspondants aux nouvelles règles et les faire évoluer.

Les tests seront donc toujours justes et formeront le filet de sécurité nécessaire à l'ajout de la nouvelle règle ou du refactor dans mon cas.
On comprends à travers cela l'importance de bien nommer ses tests afin de facilement cibler une fonctionnalité.

Un autre point, vous avez aussi pu voir que j'ai changé de paradigme en cours de route afin de faciliter l'apparition de la duplication et de faciliter la lecture.

De cela il faut noter que changer d'angle de vu, lorsque vous êtes bloqué, allant jusqu'à tout réécrire ce que vous veniez de faire, est très positif.
Rester sur une solution dont on ne voit pas comment arriver à la fin est un piège.
Prenez du recul, changer d'activité, faites une pause, faites tout pour renouveler votre perception et changez d'avis !

Augmenter la duplication permet de tuer toute envie d'optimisation trop rapide (YAGNI).

Pour finir, cette solution utilise les outils que Java 8 nous fournit pour faire de la programmation fonctionnelle et permet d'avoir un code plutôt propre et une documentation de bonne qualité.

Vous trouverez le code complet [ldez/kata-print-diamond/2015_08_22](https://github.com/ldez/kata-print-diamond/tree/2015_08_22), regarder les commits pour voir la construction étape par étape.
