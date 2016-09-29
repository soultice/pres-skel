%Disputation
%Florian Pfingstag
%29.09.2016

# Übersicht

## In dieser Präsentation

* Theorie
* Histogram-based approach
* Single-document based approach
* Das scientific dataset
* Resultate
* Fokus auf praktischer Arbeit

## Nicht in dieser Präsentation

* Einführung
* Zu viel Theorie
* Nicht verwendete Algorithmen, Distanzmaße, Ähnlichkeitsmaße

# Dataset & Preprocessing

Das Datenset sowie das Preprocessing kann auf die meisten Algorithmen übertragen
werden. Es ist jedoch nicht zwingend notwendig, dass diese Schritte einheitlich
für alle Algorithmen gelten.

## Dataset

Das für die Evaluation verwendete Datenset ist das scientific dataset, welches
auch in scisumm verwendet wird. Es ist auf github zur Verfügung gestellt:  

    https://github.com/WING-NUS/scisumm-corpus  

Das Set beinhaltet zehn Klassen mit jeweils einer Peer-Zusammenfassung und ist
weiter unterteilt in Trainings-, Test- und Evaluations-Daten.

## Sentence recognition

Da Sätze die kleinste Einheit von Sentence-Clustering darstellen, müssen sie erkannt werden.
Das geschieht in diesem Fall durch das Python Modul SpaCy.  

Es ist in der Lage Sätze schnell und zuverlässig in unterschiedlichen Sprachen
zu erkennen.  
Es kann zusätzlich POS-Tags erstellen sowie syntaktische Ähnlichkeiten und
Named Entities erkennen.

## Clean-up

Zum Erfolg von Sentence Clustering trägt das Preprocessing maßgeblich bei.  

Da einige Algorithmen auch auf Wortebene vergleichen, und Satzabstands/-ähnlichkeits
Maße oft auf Wortebene agieren, ist es notwendig den Korpus im Vorhinein zu
bereinigen:

* Stopwörter werden mittels NLTK-Stopwords entfernt
* Zu kurze Wörter werden entfernt
* Zu kurze Sätze werden nicht mit einbezogen
* Sätze werden nicht mit einbezogen wenn sie zu viele Sonderzeichen enthalten
* Die Sätze werden dann in ihrer originalen Reihenfolge als Python-List abgespeichert

# Histogram-based approach

Der Histogram-basierte Ansatz clustert Sätze mit hilfe eines Histograms.
Es wird zu jedem, schom im Cluster vorhandenen, Satz der Satzabstand berechnet
und daraus das Histogram erstellt.

## Theorie

Der Satzabstand/Die Satzähnlichkeit wird berechnet durch einen simplen BOW
Algorithmus bei dem Wort-Unigramme zweier Sätze verglichen werden:

$$ Sim(S_i,S_j) = \frac{ (2 \cdot |S_i \cap S_j|) }{ (|S_i| + |S_j|) } $$

Hier sind $S_i, S_j$ die zwei Sätze, die verglichen werden sollen und
$|S_i|$ ist die Länge des Satzes i

Für jeden Satz des Input Dokumentes wird nun der Satzabstand zu jedem Cluster
berechnet.  
Der Satz wird zu dem Cluster hinzugefügt, der am 'stärksten'
verbessert wird; falls kein Cluster ausreichend verbessert wird, erstellt
der Algorithmus einen neuen Cluster der den Satz enthält.

Die Histogram-ratio wird für jeden Cluster gespeichert.  

## Histogram Ratio

$$ {HR}_c = \frac{ \sum_{i=T}^{B} h_i} { \sum_{j=1}^{B} h_j} $$

Hier ist T das Threshold-Bin, B das höchste belegte Bin und $h_i,h_j$ die 
Anzahl an Elementen in dem entsprechenden bin.

## Ablauf

<img src="./Images/05_hist_bin.png" class="center" alt="Drawing" style="width: 500px;"/>

# Single-document approach

Anders, als die meisten Multi-Document Summarization Algorithmen, wird
im Single-Document Ansatz zuerst für jedes Dokument eine Zusammenfassung erstellt.
Aus diesen Zusammenfassungen wird dann durch Clustering der Sätze eine 
einzelne Zusammenfassung erstellt.

## Theorie

Die Sätze der einzelnen Dokumente werden mit Wertigkeit versehen, die sich 
aus 4 Features bildet:

$$ DF = w_1 + w_2 + ... + w_n $$ 
$$ SRI = \cases{1 & if following sentence has pronoun
                                           ,\cr 0 & otherwise \cr}$$

CS ist die Anzahl an Matches von Synsets der Query zu einem Satz.  
LF ist gewichtet nach der Position des Satzes im Dokument.

$$ SW = v \cdot DF + w \cdot LF + x \cdot SRI + y \cdot CS$$

Die Sätze werden dann mit dem maximalen Gewicht eines Dokumentes normalisiert.

Anschließend können die $n$ am stärksten gewichteten Sätze extrahiert werden,
ihre originale Position im Dokument wird dabei beibehalten.

Die daraus entstandenen Sätze werden dann mittels den folgenden Satzabstandsmaßen
geclustert:

## Theorie

**Syntaktische Ähnlichkeit**:

Um die syntaktische Ähnlichkeit zu berechnen werden die zwei zu vergleichenden
Sätze in Vektoren umgewandelt, welche dann ermöglichen die syntaktische Ähnlichkeit
anhand der Werte der Vektoren zu berechnen.

1. The red car drives faster than the green car
2. The green car drives faster than the red car

$$ v0 (1): [1,2,3,4,5,6,7,8,9] $$
$$ vr (2): [1,8,3,4,5,6,1,2,3] $$

Die syntaktische Ähnlichkeit kann dann als Kosinus-Koeffizient der beiden
Vektoren berechnet werden.

## Theorie

**Semantische Ähnlichkeit**:

Die semantische Ähnlichkeit wird berechnet aus mehreren Faktoren:

* Der kürzeste Pfad zwischen zwei Worten aus den zu vergleichenden Sätzen,
basierend auf WordNet Hierarchie

* Die Tiefe des ersten gemeinsamen Parents von beiden Wörtern in einem WordNet
Graphen

Beide können dann vereint werden durch die Funktion:

$$ S_w(w_i,w_j) = \frac{f(d)}{f(d)+f(l)} $$

Der information content, der benötigt wird um die semantische Ähnlichkeit zweier
Sätze zu berechnen, stellt sich wie folgt dar:

$$ I(w) = -\frac{\log p (w)}{(log N+1)} $$


## Ablauf

![](./Images/single_doc.svg)

# Postprocessing

Um die Lesbarkeit des Textes wiederherzustellen, müssen die Schritte des
Preprocessing im Postprocessing wieder umgekehrt werden. Ausserdem
müssen die Cluster nach ihrer Wichtigkeit geordnet werden und die wichtigsten
Sätze im Anschluss ausgewählt werden. Die finale Ausgabe ist dann in der Form
eines Extrakts in Plain-Text dargestellt.

## Rekonstruktion

Die Rekonstruktion der Sätze ist relativ simpel.  
Da sie vor dem Preprocessing
in einer separaten Liste gespeichert werden, kann einfach auf ihren Index zugegriffen 
werden und somit ist der Satz wiederhergestellt.

## Cluster-Ordering

Da die Cluster in einer fixen Reihenfolge gespeichert werden, die nicht unbedingt
der Reihenfolge der Sätze in einem Dokument entspricht, ist es notwendig, sie nach
ihrer Relevanz zu ordnen.  
Dafür stehen unterschiedliche Methoden zur Verfügung:  


* Ordnen nach Größe der Cluster

* Log-count sorting


$$ W(C) = \displaystyle\sum_{w \in C} \log (1+\text{count}(w)) \;\;\;\;\;\;c > th $$

Hier ist $W(C)$ das Gewicht eines Clusters, count($w$) ist die Anzahl an Vorkommen, die
das wort $w$ im input besitzt.

## Sentence Selection

Um die Sätze mit höchstem Informationsgehalt in die Zusammenfassung aufzunehmen,
und die mit geringerem Informationsgehalt zu verwerfen, 
ist es notwendig den Informationsgehalt zu messen.   
Dazu werden die folgenden Methoden vorgeschlagen:

* Arbitrary - Zufallsauswahl
* Longest candidate - der längste Satz jedes Clusters
* Centroid similarity - Ähnlichkeit des Satzes zum Centroid $log(1+\text{term frequency})$
* Local-global importance - Informationsgehalt jedes Satzes im Verhältnis
zum Cluster und allen Dokumenten

# Zusammenfassend

## Evaluationsergebnisse

<table class="center">
<thead>
<tr>
<th>Document</th>
<th>ROUGE-1-R HR</th>
<th>ROUGE-1-R SD</th>
</tr>
</thead>

<tbody>
<tr>
<td> NO1 </td>
<td> 0.196 </td>
<td> 0.366 </td>
</tr>

<tr>
<td> H89 </td>
<td> 0.354 </td>
<td> 0.388 </td>
</tr>

<tr>
<td> J00 </td>
<td> 0.203 </td>
<td> 0.236 </td>
</tr>

<tr>
<td> J98 </td>
<td> 0.363 </td>
<td> 0.321 </td>
</tr>

<tr>
<td> X96 </td>
<td> 0.446 </td>
<td> 0.250 </td>
</tr>

<tr>
<td> P98 </td>
<td> 0.255 </td>
<td> 0.277 </td>
</tr>

<tr>
<td> Average </td>
<td> 0.297 </td>
<td> 0.306 </td>
</tr>

</tr>
</tbody>
</table>

# Quellen


