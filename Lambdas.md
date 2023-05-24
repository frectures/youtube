## Java-Lambdas und Kotlin-Lambdas im Vergleich

Praktisch alle Mainstream-Sprachen haben heutzutage Lambda-Ausdrücke (anonyme Funktionen):
```
  x => x + y    // JavaScript, C#, Scala
  x -> x + y    // Java
{ x -> x + y }  // Kotlin, Groovy
```
Syntaktisch fahren Java-Lambdas und Kotlin-Lambdas ähnliche Schienen,
die Semantik unterscheidet sich aber in zwei interessanten Detail-Fragen:

1. Dürfen umgebende lokale Variablen (das `y` in `x -> x + y`) verändert werden?
2. Wohin springt eine `return`-Anweisungen aus einem Lambda zurück?

Dieser Artikel untersucht beide Fragen und ist auch für komplette Neueinsteiger in Kotlin geeignet.

## 1. Umgebende lokale Variablen

Listing 1a zeigt eine idiomatische JavaScript-Fabrikfunktion `makeCounter` zum Erzeugen von Zählfunktionen.
Man beachte, dass die lokale Variable `next` das Verlassen der Funktion `makeCounter` mittels `return` überlebt
und *nicht* "vom Stack verschwindet", weil das Lambda `() => next++` sie anschließend noch benötigt.

Die Listings 1b und 1c zeigen direkte Portierungsversuche von JavaScript nach Java und Kotlin,
die im Folgenden als Vergleichsgrundlage für Java-Lambdas und Kotlin-Lambdas dienen werden.

```js
function makeCounter() {
    let next = 1;
    return () => next++;
}         /////////////
       ///////
const counter = makeCounter();

console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3

// Listing 1a: makeCounter in JavaScript
```

```java
public class JavaLambdas {
    public static IntSupplier makeCounter() {
        var next = 1;
        return () -> next++;
    }

    public static void main(String[] args) {
        final var counter = makeCounter();

        System.out.println(counter.getAsInt());
        System.out.println(counter.getAsInt());
        System.out.println(counter.getAsInt());
    }
}

// Listing 1b: makeCounter in Java
```
```kotlin
fun makeCounter(): () -> Int {
    var next = 1
    return { next++ }
}

fun main() {
    val counter = makeCounter()

    println(counter())
    println(counter())
    println(counter())
}

// Listing 1c: makeCounter in Kotlin
```

### Syntax

Syntaktisch fällt in Kotlin sofort ein Dutzend Unterschiede zu Java auf:
1. Funktionen können außerhalb von Klassen existieren
2. Funktionen werden mit dem Schlüsselwort `fun` definiert
3. Deklarationen folgen dem Muster `name: Typ` statt `Typ name`
4. Funktionstypen `() -> Int` statt funktionaler Interfaces `IntSupplier`
5. Semikola sind optional
6. Die geschweiften Klammern `{ next++ }` gehören zum Lambda
7. Leere Parameterliste `() ->` im Rückgabetyp, aber nicht im Lambda
8. `main` benötigt keinen Parameter `args: Array<String>`
9. Rückgabetyp `Unit` (entspricht Javas `void`) ist optional
10. `val` statt `final var`
11. `println` statt `System.out.println`
12. `counter()` statt `counter.getAsInt()`

### (Effektiv) unveränderlich

Listing 1b ist das einzig fehlerhafte im Bunde.
Der Java-Compiler lehnt das Programm an der Stelle `next++` mit folgender Begründung ab:

> error: local variables referenced from a lambda expression must be final or effectively final

In Listing 2 fehlt das `++` nach `next` probehalber (somit wird leider immer dieselbe Zahl 1 zurückgeliefert).
Dadurch ist die Variable `next` effektiv unveränderlich, und der Java-Compiler akzeptiert das Programm.

Interessanterweise färben Entwicklungsumgebungen die beiden Vorkommen von `next` im Quelltext unterschiedlich ein.
Der Grund dafür ist bei anonymen inneren Klassen einfacher nachvollziehbar als bei Lambdas.
Dort verrät nämlich ein Blick in den Bytecode (hier zwecks Lesbarkeit wieder manuell nach Java zurücktransformiert),
dass `return next` gar nicht die lokale Variable `next` auf dem Stack verwendet,
sondern eine Kopie namens `val$next` auf dem Heap, die vom Konstruktor initialisiert wurde.

```java
public static IntSupplier makeCounter2a() {
    var next = 1;
    return () -> next;
}


public static IntSupplier makeCounter2b() {
    var next = 1;
    return new IntSupplier() {
        @Override
        public int getAsInt() {
            return next;
        }  ////////////
    };
}


class JavaLambdas$1 implements IntSupplier {

    final synthetic int val$next;

    JavaLambdas$1(int parameter) {
        this.val$next = parameter;
    }

    @Override
    public int getAsInt() {
        return this.val$next;
    }  /////////////////////
}

// Listing 2: Anonyme innere Klassen
```

Lambdas mit Zugriff auf umgebende lokale Variablen werden in Java unter der Haube also
durch *Kopieren* der umgebenden lokalen Variablen in Objekt-Felder realisiert.
Genau daher rührt übrigens die Sprachregel, dass anonyme innere Klassen und Lambdas
nur auf (effektiv) unveränderliche umgebende lokale Variablen zugreifen dürfen –
ohne diese Einschränkung würden Veränderungen der scheinbar selben Variable
innerhalb und außerhalb eines Lambdas *verschiedene* Variablen betreffen!

### AtomicInteger

Das `return () -> next;` ohne `++` hat leider nicht den gewünschten Zähleffekt.
Im ursprünglichen Listing 1b schlagen Entwicklungsumgebungen an der Stelle `next++` vor,
den primitiven Typ `int` durch den Referenztyp `AtomicInteger` zu ersetzen.
Dann wird nur eine Referenz auf das `AtomicInteger`-Objekt kopiert und *nicht* der enthaltene `int`.
Listing 3 zeigt diesen Ansatz mit zwei anschließenden Vereinfachungen.

```java
public static IntSupplier makeCounter3a() {
    var next = new AtomicInteger(1);
    return () -> next.getAndIncrement();
}


public static IntSupplier makeCounter3b() {
    var next = new AtomicInteger(1);
    return next::getAndIncrement;
}


public static IntSupplier makeCounter3c() {
    return new AtomicInteger(1)::getAndIncrement;
}

// Listing 3: AtomicInteger
```

### Array oder IntRef

Für den Fall, dass die Thread-Sicherheit von `AtomicInteger` unerwünscht ist, zeigt Listing 4 zwei Alternativen.
Ein gängiger Hack [Wiki] ist die Verwendung eines Arrays mit nur einem Element.
Etwas sauberer ist die Verwendung eines eigenen Typs `IntRef`, der nur aus einem öffentlichen `int`-Feld besteht.
In beiden Fällen wird auch wieder nur eine Objekt-Referenz kopiert und *nicht* der enthaltene `int`.

```java
public static IntSupplier makeCounter4a() {
    var next = new int[1];
    next[0] = 1;
    return () -> next[0]++;
}


public static IntSupplier makeCounter4b() {
    var next = new IntRef();
    next.element = 1;
    return () -> next.element++;
}

static class IntRef {
    public int element;
}

// Listing 4: int[] oder IntRef
```

### Kotlin

Im Gegensatz zu Java akzeptiert Kotlin den Ausdruck `next++` im ursprünglichen Listing 1c.
Wie kann das sein?
Listing 5 zeigt eine manuelle Rücktransformation des Bytecodes von `makeCounter` und `main`.

```java
// class version 52.0 (52)
// compiled from: KotlinLambdas.kt
public final class KotlinLambdasKt {

    @org.jetbrains.annotations.NotNull
    public static final kotlin.jvm.functions.Function0<Integer> makeCounter() {
        var next = new kotlin.jvm.internal.IntRef();
        next.element = 1;
        return new KotlinLambdasKt$makeCounter$1(next);
    }

    public static synthetic void main(String[] args) {
        main();
    }

    public static final void main() {
        // ...
    }
}

// Listing 5: Kotlin dekompiliert
```

Hier fällt ein halbes Dutzend interessanter Punkte auf:

1. `class version 52.0 (52)` ist Java 8, d.h. Kotlin läuft auf allen Systemen, die Java 8 unterstützen.
   Deshalb ist Kotlin unter Android-Entwicklern besonders beliebt.
2. Klassenlose Funktionen in `Dateiname.kt` werden zu statischen Methoden einer Klasse `DateinameKt`
3. Jeder `Typ` in Kotlin-Quelltexten schließt `null` normalerweise aus.
   Falls `null` erwünscht ist, nimmt man `Typ?` statt `Typ`.
4. Funktionstypen `() -> Int` sind Interfaces `kotlin.jvm.functions.Function0<Integer>`,
   analog zu `java.util.function.Supplier<Integer>`
5. `next` ist gar kein `int`, sondern ein `kotlin.jvm.internal.IntRef`
6. Wenn man `main` ohne Parameter `args` definiert, synthetisiert der Compiler eine delegierende Überladung

Punkt 5 lüftet das Geheimnis, warum Kotlin den Ausdruck `next++` akzeptiert:
Der Compiler verpackt umgebende lokale `int`-Variablen, die später verändert werden, automatisch in hauseigene `IntRef`-Objekte.
Scheinbare Zugriffe auf den `int`-Wert von `next` greifen dann in Wirklichkeit auf `next.element` zu.
Sowohl außerhalb als auch innerhalb des Lambdas ist `next.element` *dieselbe* `int`-Variable.
Diesbezüglich sind Veränderungen umgebender lokaler Variablen in Kotlin also unproblematisch und deshalb erlaubt.

## 2. Rücksprung aus Lambdas

```kt
fun main() {
    val digits = IntRange(0, 9)
    digits.forEach({ a ->
        digits.forEach({ b ->
            if (a * b == 42) {
                println("Kotlin: $a * $b == ${a * b}")
                return
            }
        })
    })
}

// Listing 6a: Rücksprung aus Java-Lambda
```

```java
public static void main(String[] args) {
    IntStream.rangeClosed(0, 9).forEach(a -> {
        IntStream.rangeClosed(0, 9).forEach(b -> {
            if (a * b == 42) {
                System.out.println(" Java : " + a + " * " + b + " == " + a * b);
                return;
            }
        });
    });
}

// Listing 6b: Rücksprung aus Kotlin-Lambda
```

Die Listings 6a und 6b zeigen scheinbar dasselbe Programm in Kotlin und Java.
Zwei verschachtelte `forEach`-Aufrufe suchen zwei Ziffern, deren Produkt 42 ist.
Bei einem passenden Pärchen schreibt das Programm die Rechnung auf die Konsole und springt zurück.
Die Programme schreiben folgendes auf die Konsole:

```
Kotlin: 6 * 7 == 42

 Java : 6 * 7 == 42
 Java : 7 * 6 == 42
```

Wieso schreibt Kotlin nur 1 Zeile auf die Konsole, aber Java schreibt 2 Zeilen?
Weil das `return` in Kotlin komplett aus der `main`-Funktion zurückspringt,
aber das `return;` in Java nur aus dem inneren Lambda.
Tatsächlich graut die Entwicklungsumgebung das `return;` am Ende des Java-Lambdas aus:

> `return` is unnecessary as the last statement in a `void` method

Java kann immer nur "einen Methodenaufruf weit" zurückspringen.
Zur Laufzeit unterliegt Kotlin derselben Einschränkung.
Der Compiler generiert für `forEach` aber gar keine Methodenaufrufe,
sondern kopiert stattdessen den Methodenrumpf an die Aufrufstellen,
weil `forEach` als `inline` deklariert ist, siehe Listing 7.
Dadurch verhalten sich `forEach` und `for` hier komplett identisch,
insbesondere bzgl. `return`-Semantik.

Falls man analog zu Java nur aus dem inneren Lambda zurückspringen will,
schreibt man `return@forEach` statt `return`
(was als letzte Anweisung in einem Kotlin-Lambda aber genau so überflüssig ist wie in Java).

```kt
inline fun <T> Iterable<T>.forEach(action: (T) -> Unit) {
    for (element in this) {
        action(element)
    }
}

inline fun repeat(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) {
        action(index)
    }    
}

// Listing 7: inline-Funktionen
```

### Neue Kontrollstrukturen

Wenn das letzte Argument eines Funktionsaufrufs ein Lambda ist,
kann man dieses letzte Lambda aus der Argumentliste herausschieben.
Werden die runden Klammern dadurch leer, sind sie optional, siehe Listing 8.

```kt
digits.forEach({ a -> ... })
              //              schiebe Lambda aus runden Klammern heraus
digits.forEach(){ a -> ... }
              //              entferne leeres rundes Klammerpärchen
digits.forEach { a -> ... }

repeat(10, { a -> ... })
repeat(10) { a -> ... }

// Listing 8: Lambda-Argument-Syntax
```

Mit diesem Syntax-Trick sehen Funktionsaufrufe wie neue Kontrollstrukturen aus:
bei `forEach` und `repeat` wirken die Lambda-Argumente dank geschweifter Klammern syntaktisch wie Schleifenrümpfe.

Zwei weitere Beispiele sind die Funktionen `assert` und `synchronized` aus Kotlins Standardbibliothek,
siehe Listing 9. Im Gegensatz zu Java sind `assert` und `synchronized` nämlich *keine* Schlüsselwörter in Kotlin!

```kt
synchronized(einLock) {
    assert(eineBedingung) {
        berechneTeureFehlernachricht()
    }
}

// Listing 9: synchronized und assert
```

## Fazit

Im Gegensatz zu Java-Lambdas können Kotlin-Lambdas umgebende lokale Variablen verändern.
Funktionale Idiome aus JavaScript und anderen Programmiersprachen sind dadurch reibungslos nach Kotlin portierbar.

Aufrufe von inline-Funktionen mit Lambda-Argumenten wirken syntaktisch wie neue Kontrollstrukturen.
Das ist ein idealer Nährboden für Domänenspezifische Sprachen, zum Beispiel "HTML in Kotlin". [Ktor]
Außerdem spart Kotlin dank inline-Funktionen Schlüsselwörter im Sprachkern ein.

Lange vor Java 8 bemühten sich Gilad Bracha, Neal Gafter, James Gosling und Peter von der Ahé
übrigens, "Lambda-Kontrollstrukturen" auch in Java zu etablieren. [Neal] [Tube]
Letzten Endes konnten sich ihre Ideen aber nicht gegen simplere Lambda-Designs durchsetzen.

## Links

[Wiki] https://wiki.c2.com/?ClosuresThatWorkAroundFinalLimitation<br>
[Ktor] https://ktor.io/docs/html-dsl.html<br>
[Neal] http://gafter.blogspot.com/2006/08/closures-for-java.html<br>
[Tube] https://www.youtube.com/watch?v=yUmWQHzN5ZU&t=489s

## Autor

Fredrik arbeitet mit Schwerpunkt Nachwuchsgewinnung und -förderung bei der WPS.
Seit 2015 entwickelt er zwei Lernumgebungen ("Karel" und "Skorbut") mit Kotlin.
Seit 2018 schult er Java-Programmierer in Kotlin.
