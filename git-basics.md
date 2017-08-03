# DVCS
## VCS allgemein
VCS sind dafür da, die Entwicklung von Software zu sichern und verfolgbar/kontrollierbar zu machen.

Es gibt einige VCS auf dem Markt:

* git (OSS)
* Subversion (OSS)
* Mercurial (OSS)
* Microsoft TFS
* IBM ClearCase
* darcs (OSS)
* BitKeeper
* …

Historisch:

* SCCS
* RCS

## verteilte/dezentralisierte Systeme
Verteilt: Es werden keine Annahmen gemacht, was in welcher Reihenfolge in der Umwelt geschieht.
Dezentralisiert: Keine „eine Quelle der Wahrheit”

Erhöhte Komplexität, sowohl technisch als auch gedanklich

Warum? Unabhängige Arbeit der Beteiligten ⇒ skalierbar auf viele Entwickler, kein Single Point of Failure



# Datenmodell
Zum Verständnis vieler git-Aktionen ist ein Überblick über das Datenmodell nützlich (wie auch bei vielen anderen Programmen, [aber bei git in besonderem Maße](https://xkcd.com/1597/)).
Die ersten paar Konzepte treffen auf alle VCS zu.

## Arbeitsbereich/-baum
Der Arbeitsbereich (engl. working tree) ist das benutzersichtbare Verzeichnis, in dem der aktuell betrachtete Sourcecode o. ä. liegt.
Der Nutzer hat volle Verwaltung des Arbeitsbereichs und kann notfalls alles dort kaputt machen, da die Daten vom VCS gesichert sind.

## Repository
Das Repository ist die Sammlung an Daten, die vom VCS verwaltet werden.

### Staging area
Ab hier wird es git-spezifisch. Die Staging Area ist ein virtueller (AFAIK) Verzeichnisbaum der für die Erstellung des nächsten Commits benutzt wird.
Die tatsächliche Darstellung der Staging Area ist unerheblich.

### Objekte (append-only)
Das Herz von git sind die Objekte. Dabei handelt es sich um einen Satz Datenblöcke, die über ihren Hash adressiert werden können.
Diese Eigenschaft bedingt, dass keine zyklischen Verweise existieren können, weil sich bei einer Änderung des Inhalts i. d. R. auch der Hash ändert.
Dies ist äußerst relevant für die Funktionalität vieler Algorithmen von git.
Im Folgenden eine Aufführung der wichtigsten Objekttypen.

#### Blob
Ein Blob ist ein nicht weiter von git verstandener Block aus Bytes.
Blob-Objekte werden z. B. zur Darstellung des Inhalts(!!) von Dateien eingesetzt.

#### Tree
Ein Tree-Objekt dient v. A. zur Speicherung des Inhalts eines Verzeichnisses und besteht aus einer Liste von beliebigen, aber eindeutigen Namen für den Hash (Verweis!) eines Objekts.

#### Commit
Ein Commit stellt einen Stand in der Entwicklung des Projekts dar.
Er beinhaltet:

* einen Verweis auf das Tree-Objekt, das den Arbeitsbereich zu diesem Stand darstellt
* eine Beschreibung der zuletzt durchgeführten Änderungen am Projekt
* beliebig viele (∈ ℕ₀) Vorgänger-Commits (Mercurial erlaubt z. B. nur 0, 1 und 2 und hat für Fälle 0/1 eine Extrawurst-Darstellung ☺) [nein, für diese Zeile musste ich keine Zeichentabelle aufmachen ☺]
* Zusätzliche Metadaten (Autor, Reviewer, Committer, Datum, …)

## Referenzen
Da sich die Inhalte, die zu einem Hash gehören, nicht ändern können, muss es, damit das Repository nützlich sein kann, Teile geben, die nich mit ihrem Hash identifiziert werden.
Außerdem ist es für die menschlichen Nutzer von git hilfreich, für die Operationen nicht immer SHA1-Hashes zu merken/kopieren.
Dies sind die sog. Referenzen.

### lokal
Die meistgenutzten Referenzen sind die, die sich bei der normalen Benutzung des Repositorys ändern. Diese nennt man lokal, da sie in den Verfügungsbereich des lokalen Repositorys fallen.

#### HEAD
Die wichtigste (aber nicht notwendigerweise bekannteste ☹) Referenz ist HEAD.
Sie verweist auf einen Vorgänger-Commit des aktuellen Arbeitsbereichs.

#### Branches
Branches sind verschiedene getrennt verlaufende Entwicklungszweige des Projekts.
Das Konzept des Branches ist für die Arbeit mit git äußerst wichtig.
Technisch gesehen ist ein Branch einfach nur ein Verweis auf den neuesten Stand eines Branches.
Diese simple Definition eines Zweigs (im englischen Marketing von git *lightweight branches* genannt) ist insbesondere im Vergleich zu Mercurial ein definierender Unterschied.[IMO ist gits Modell besser]

HEAD verweist, außer in einem sog. *detached HEAD mode*, auf den es im gegebenen Fall lautstark hinweisen wird, immer indirekt über einen Branch auf den aktuellen Stand.

### remote
Neben den lokalen Branches gibt es auch solche, die durch den Abgleich mit fremden Kopien des Repositorys angelegt werden.
Die Erkenntnis, dass die parallele, aber getrennte Arbeit eines Kollegen am Projekt nichts anderes als ein getrennter Branch ist, ist ein Fundament der dezentralisierten Arbeit von git. (siehe <https://youtu.be/4XpnKHJAok8?t=14m50s>)
Durch die Trennung von lokalen und fremden Branches werden zum einen die Änderungen eines anderen Mitarbeiters nicht direkt dem Nutzer aufgedrängt (Hallo Mercurial!), und zum Anderen werden bei dezentralisierter Arbeit Namenskonflikte vermieden.

### Tags
Zu guter Letzt gibt es Namen, die innerhalb eines Projekts möglichst eindeutig und unveränderlich vergeben werden sollen.
Diese Referenzen nennen sich in git – so wie in den meisten anderen VCS – Tags (zu deutsch: (Versions-)Marken).



# Aktionen
`git` unterstützt eine große Fülle an Befehlen. Im Folgenden wird eine Auswahl daraus aufgeführt, die jeder git-Nutzer kennen sollte. Grundlegende Kenntnisse der Kommandozeile werden vorausgesetzt.

## git init
Zur Erstellung eines neuen Repositorys im aktuellen Verzeichnis kann `git init` eingesetzt werden.
Der Befehl kann auch ein abgesehen vom Repository leeres Verzeichnis erstellen, indem dem Befehl ein Name mitgegeben wird.

## Pre-commit
Bevor Daten in das Repository geschrieben werden, können/sollten/müssen einige Befehle benutzt werden.

### git add
Standardmäßig befinden sich keine Dateien in der Staging Area. Dies kann mit `git add` geändert werden.
Auch bei bereits in der Staging Area befindlichen Dateien kann der Befehl benutzt werden. Hier wird die Datei in der Staging Area auf den neuen Stand gebracht.

In manchen Workflows ist es notwendig, mehrere Änderungen gleichzeitig zu machen, die aber gerne getrennt übernommen werden.
Wenn mehrere solcher Änderungen in einer Datei liegen, kann das interaktive `git add --patch` bzw. `git add -p` hilfreich sein, mit dem die Änderungen getrennt aufgenommen werden können.

Alle Änderungen des Arbeitsbereichs können automatisch mit dem Kommandozeilenargument `--all` bzw. `-A` übernommen werden.

TIPP: `git add` kann, wie die meisten `git`-Befehle, die mit Dateinamen arbeiten, mehrere Pfade entgegennehmen. Hierbei bezieht sich, auch bei den meisten anderen Befehlen, der Pfad eines Verzeichnisses auf alle darin enthaltenen Dateien.

### git status
`git status` zeigt den aktuellen Stand von HEAD, ggf. im Vergleich zu gewissen remote-Branches, der Staging Area und des Arbeitsbereichs im Vergleich zum Repository an und zeigt, wie bestimmte Änderungen rückgängig gemacht werden können.
`git status` wird keinen Schreibzugriff verursachen und ist somit im Zweifelsfall sicher aufzurufen.

TIPP: `git status` ist ein nützlicher Befehl. Um Tipparbeit zu sparen, kann ein Alias angelegt werden, um den Aufruf z. B. auf `git st` zu kürzen. Dies kann mithilfe von `git config --global alias.st status` bewerkstelligt werden.

### git checkout
`git checkout` übernimmt die Inhalte eines Branches oder Commits in den Arbeitsbereich und setzt HEAD entsprechend.
Lokale Branches werden mit ihrem Namen ausgecheckt, während fremde Branches `remotename/branchname` benannt sind.

Es ist natürlich auch möglich, alte Commits (Commits, die nicht notwendigerweise der aktuelle Stand eines Branches sind) auszuchecken.
In diesem Fall wird ein sog. *detached HEAD state* eingeleitet, was von für git-Neulinge teilweise verstörenden Warnungen begleitet wird. Mehr dazu im nächsten Abschnitt.
Auf alte Commits kann z. B. mithilfe des Tagnamens verwiesen werden. Wenn der Commit keine Referenz besitzt, kann immer noch auf mindestens 2 Weisen auf ihn verwiesen werden:

* Mithilfe seines Hashes. Auch eindeutige Präfixe sind benutzbar.
* Durch einen benannten Nachfolger. Der Vorgänger-Commit des Standard-Branches `master` kann mithilfe von `master^1` benannt werden. Dies funktioniert mit allen Referenzen (HEAD, Branches, Tags)

TIPP: Alle diese Arten, auf Commits zu verweisen, können in allen `git`-Befehlen, die mit Commits arbeiten, verwendet werden.

TIPP: Dieser Befehl wird auch häufig genutzt – es ist u. U. zu empfehlen, ein Alias `co` (analog zu Mercurial oder Subversion) zu vergeben (`git config --global alias.co checkout`)

## git commit
Wenn der richtige Branch ausgecheckt wurde, die Änderungen gemacht und in die Staging Area ausgewählt wurden, können sie mithilfe von `git commit` in den permanenten Teil des Repositoriums übernommen werden.
Dieser Befehl schreibt den in der Staging Area befindlichen Stand in die Objektdatenbank und aktualisiert den in HEAD beschriebenen Branch.
Wenn ein *detached HEAD state* vorliegt, d. h. HEAD verweist direkt auf einen Commit, wird nur der Verweis in HEAD entsprechend aktualisiert.
Das committen/einchecken in einem *detached HEAD state* birgt das Risiko, Commits zu erzeugen, die später nicht mehr bzw. schwer auffindbar sind.

Standardmäßig öffnet git beim Committen einen Texteditor, um die Beschreibung des Commits einzugeben.
Dies kann aber unterbunden werden, indem das Kommandozeilenargument `--message=` bzw. `-m` gefolgt von der Beschreibung übergeben wird.
Dabei sind allerdings die Stilrichtlinien des Projekts zu beachten, die manchmal mehrzeilige Beschreibungen fordern und diese Möglichkeit somit unpraktisch machen.

`git` wird kein Commit mit leerer Beschreibung anlegen und abbrechen wenn es einen leeren Text dazu erhält (z. B. durch nicht-Ändern/-Speichern im Editor oder mit `-m ""`).
Es wird außerdem keinen Commit ohne Committer erstellen. Wenn keine Standardwerte dafür eingestellt sind, wird es zeigen, wie dies zu tun ist.

Der Author ist normalerweise dieselbe Person wie der Committer, es kann jedoch ein Kommandozeilenargument `--author="Name <email@example.com>"` übergeben werden, um Abweichendes anzugeben.

Die Staging Area wird umgangen, wenn `git commit` Argumente wie `git add` erhält, also Pfade, `--all`, dessen Kurzform hier `-a`(Achtung, Groß-/Kleinschreibung!) ist, oder `--patch`/`-p`.

TIPP: `-a -m "blah"` kann zu `-am "blah"` zusammengefasst werden.

TIPP: Analog zu Mercurial oder Subversion kann für `commit` ein Alias `ci` eingerichtet werden (`git config --global alias.ci commit`)

## git reset
Menschen machen unausweichlich Fehler, und bei git können die meisten davon mit `git reset` ausgebügelt werden.
Der Befehl besitzt 3 verschiedene Stufen: `--soft`, `--mixed`(der Standard) und `--hard`.

Wenn die gesamten Änderungen seit dem letzten Commit umsonst waren, können alle von git verwalteten Dateien mit `git reset --hard` auf ihren sauberen Stand zurückgesetzt werden.
Einzelne Dateien können mit `git reset --hard HEAD <pfad>` zurückgesetzt werden.

Wenn versehentlich Änderungen, die für spätere Commits vorgesehen sind, in die Staging Area übernommen wurden, können sie mit `git reset HEAD <pfad>` daraus entfernt werden. Die Dateien im Arbeitsbereich bleiben unberührt.

Wenn Commits erstellt wurden, die unerwünschte Änderungen beinhalten, können diese mithilfe von `git reset --soft <commit>` behoben werden. Hier wird weder Staging Area noch Arbeitsbereich verändert, aber der aktuelle Branch auf Commit gesetzt. Die Implikationen solcher Aktionen bei der Zusammenarbeit werden unten näher beschrieben.

Beim derartigen Zurückrollen von Branches/Commits ist häufig die Circonflex-Notation hilfreich, z. B. setzt `git reset --soft HEAD^2` den aktuellen Branch um 2 Commits zurück. `git reset --soft` setzt jedoch keinesfalls voraus, dass der angegebene Commit ein Vorgänger der ursprünglichen Branch-Spitze ist. Somit kann der Befehl auch genutzt werden, um ‚verlorene’ Commits zurückzubekommen.

Solche ‚verlorenen’ Commits können durch Befehle, die die Geschichte eines Branches neu schreiben, beispielsweise `git reset --soft` selber, `git branch -d` oder `git rebase` (siehe unten) entstehen. git löscht Commits, auf die weder durch Referenzen noch transitiv durch erreichbare Commits verwiesen wird, jedoch nicht sofort, sondern erst bei der nächsten Garbage Collection. Davor können die Commits entweder aus dem Konsolen-Scrollback oder durch den Befehl `git reflog` wiedergefunden werden.

## git branch
Standardmäßig existiert nur ein lokaler Zweig, `master`. Dieser Name hat allerdings keine besondere technische Bedeutung.
Zusätzliche lokale Branches werden in git naheliegenderweise mit dem Befehl `git branch <name>` erstellt.
Dies hat jedoch das häufig unerwünschte Verhalten, dass HEAD nicht automatisch auf diesen Branch gewechselt wird, sondern auf dem alten Branch verbleibt.

Wenn das gewünscht ist, kann `git checkout -b <name>` benutzt werden.
`git checkout -b <name> [<commit>]` ist, neben `git reset --soft`, außerdem ein üblicher Weg, verlorene Commits, die noch nicht von der GC gelöscht wurden, wiederherzustellen.

## git diff
Es besteht häufig das Bedürfnis, nachzuschauen, welche genauen Änderungen zu committen sind. Dazu dient `git diff`.

TIPP: Teilweise so häufig, dass es sich lohnen kann, selbst das kurze `git diff` mit einem Alias auf `git d` abzukürzen. (`git config --global alias.d diff`)

Der Befehl erstellt einen Patch der gesamten Änderungen des Arbeitsbereichs gegenüber der Staging Area.

Wenn nur die Menge der Änderungen pro Datei angezeigt werden sollen, kann `--stat` übergeben werden.

`git diff` kann auch Patches zwischen beliebigen anderen Commits, Tree-Objekten oder Blobs erstellen, die übergeben werden (siehe `git checkout`). Wenn nur ein Commit/Tree/Blob übergeben wird, wird mit dem Arbeitsbereich verglichen; wenn `--cached` bzw. `--staged` übergeben wird, wird mit der Staging Area verglichen.

## git log
Für die Beobachtung, Nachverfolgung und Kontrolle der Entwicklung ist eine Auflistung der Änderungen wichtig.
Der entsprechende Befehl, `git log` ist einer der mächtigsten im Repertoire der Versionsverwaltung.

Der Befehl geht chronologisch rückwärts durch die Vorgänger des aktuellen Commits und zeigt sie auf vielfältige Weise an.

### Historien-Vereinfachung (Filter)
Insbesondere in Großprojekten wie dem Linux-Kernel, für dessen Entwicklung git erstellt wurde, ist die rohe Menge der Commits zu groß, um den Überblick zu behalten.
Daher kann `git log` intuitive und je nach Art auch sehr effiziente Filter einsetzen, um die angezeigte Historie zu vereinfachen.

#### Filter nach Pfad
Die Historie kann auf Änderungen an einem oder mehreren Pfaden (Dateien oder ganze Verzeichnisse) beschränkt werden, indem sie einfach dem Befehl als Argument übergeben werden.

Im Gegensatz zu anderen VCS wie Mercurial oder Subversion speichert git nicht die Historie einzelner Dateien, sondern nur des Projekts als Ganzem. Dies vereinfacht das Datenmodell ungemein und entspricht laut Macher Linus Torvalds eher der Perspektive eines Maintainers ([“I have 22000 files to track. I don't care about one of them. I might care about a subcollection of them containing maybe 1000 files …”](https://youtu.be/4XpnKHJAok8?t=45m)). Da aber der Hash eines ungeänderten Verzeichnisses gleich bleibt, können ungenutzte Commits ohnehin schnell verworfen werden.

#### Filter nach Nutzer
Mit `--author=<regex>` kann die Historie auf die Commits des entsprechenden Autors beschränkt werden.

#### Pickaxe
Die Pickaxe ist ein mächtiger Filter, der alle Änderungen, an denen gegebene Muster beteiligt sind, selektiert werden können.
`-S <string>` wählt alle Commits, in denen der gegebene Text entfernt oder hinzugefügt wurde, während mit `-G <regex>` alle Commits gewählt werden, bei denen Zeilen, in denen der Regex matcht, geändert werden.

Dies ist nützlich, um alle Änderungen, die mit einer bestimmten Funktion/Datenstruktur/Methode/Klasse zusammenhängen, zu finden.

### Limiting
Die Suche kann auf bestimmte Abschnitte der Historie beschränkt werden, z. B. zeitlich mit `--since`/`--after` und `--until`/`--before`

### Anzeige
Die Darstellung der Commitliste kann ebenfalls auf vielfältige Weise angepasst werden.

#### Diff/Diffstat
Die Verteilung der Änderungen der Commits kann im Log dargestellt werden, indem `--stat` übergeben wird.
Auch die Änderungen selber können mithilfe von `--patch`/`-p` angezeigt werden.

Viele der Optionen von `git diff` werden auch hier unterstützt. Tatsächlich sind viele der Filter, wie die Pickaxe, ursprünglich Optionen für `git diff`, die hier die Sonderfunktion, gleichzeitig auch Commits zu filtern, besitzen.

#### Format
Auch das Format der Metadaten kann eingestellt werden.

So wird mit `--oneline` die Historie als kompakte Liste von Hash-Präfixen mit der ersten Zeile der zugehörigen Commit-Beschreibung angezeigt.

#### --graph
Mit der Option `--graph` werden die Abhängigkeiten zwischen den angezeigten Commits mit Linien visualisiert.
Somit kann z. B. der Zusammenhang/Nicht-Zusammenhang von Änderungen besser abgeschätzt werden.

## git merge
> Branches are completely useless unless you merge them!
— [Linus Torvalds, Macher von git, 2007](https://youtu.be/4XpnKHJAok8?t=52m9s)

Das Zusammenführen von Zweigen ist essentiell in git. Es ist hin und wieder nützlich für die persönliche Arbeit, und *unerlässlich* für die Zusammenarbeit (mehr dazu unten).
git besitzt komplexe Maschinerie rund um das Zusammenführen von Änderungen, das meiste davon ist allerdings für den Nutzer i. d. R. uninteressant.

Der zentrale Befehl für diese Aktion ist wenig überraschend `git merge`.
Der Befehl nimmt eine Liste von Commits (im Gegensatz zu Mercurial kann git mehrere Branches in einem Schritt zusammenführen), deren Historie nach Änderungen seit dem Trennungspunkt der beiden Historien durchsucht wird. Diese Änderungen werden in den Arbeitsbereich eingefügt und es wird ein Commit eingeleitet.

Wenn kein Commit angegeben ist, wird, wenn eingestellt, mit dem Upstream-Branch gemerget.

Wenn ein Commit der Vorgänger eines Anderen beim Zusammenführen ist, kann, wenn keine uncommitteten Änderungen vorliegen, einfach der aktuelle Branch-Verweis aktualisiert werden, ohne einen neuen Commit erstellen zu müssen.
Diese Situation nennt sich ‘fast forward’, da die Änderungen einfach ‚vorgespult’ werden können. Das Anlegen eines Commit auch in diesem Fall kann mit `--no-ff` erzwungen werden, während die Erstellung eines Commits in einer nicht-fast-forward-Situation mit `--ff-only` unterbunden wird.

### Konflikte
Beim Zusammenführen treten Konflikte auf, wenn die gleiche Stelle/Zeile im Text in mehreren beteiligten Commit-Pfaden geändert wurde.
In diesem Fall wird vor dem Commit abgebrochen (das gleiche Verhalten kann auch mit `--no-commit` beigeführt werden, wenn man z. B. die Änderungen vor dem Commit inspizieren möchte).
Dem Nutzer werden beide Seiten der Änderung angeboten, sodass eine sinnvolle Lösung vom Nutzer gewählt werden kann.
Anschließend kann auf normalem Wege der Inhalt committed werden.

### git blame
Bei der Bearbeitung von Code stellt sich regelmäßig die Frage, ob und wenn ja welche Designentscheidung zu gewissem Code geführt hat, um entweder neuen Code sinnvoll zu gestalten oder den alten zu verbessern/reparieren. Diese Problemstellung wird in vielen VCS mit einem Befehl `annotate`, der die Zeilen einer Datei der frühesten Revision, in der sie enthalten sind, zuordnet. git besitzt diesen Befehl ebenfalls, als weitestgehendes Synonym des eigentlichen `git blame` mit anderem Ausgabeformat.

Die Ausgabe dieses Befehls profitiert am meisten von Integration in den Editor der Entwicklungsumgebung, da hier gewissermaßen die Ausgabe von git mit dem Inhalt der Datei verflochten werden muss.

### git bisect
git war das erste VCS, das das Finden von problematischen Änderungen (Regressionen, Performance-Einbrüche, Einführung von Sicherheitslücken, …) mithilfe von Binärsuche erlaubte. Der Name `git bisect` wurde seitdem in anderen Systemen wie Mercurial zusammen mit dem Feature übernommen.

Da bei `bisect` eine Binärsuche durchgeführt wird, müssen nur O(log n) Tests durchgeführt werden, um den problematischen Commit zu finden.

### Daggy fixes
`bisect` geht Hand-in-Hand mit einer Methode, wie durch reines Branching und Merging Probleme in mehreren Versionen behoben werden können ohne dass die Änderungen auf mehrere Branches getrennt aufgetragen werden müssen.

Bei diesem Vorgehen wird nach dem Finden des Fehlers, bspw. mit `bisect` (i. d. R. *detached HEAD*) ein neuer Branch an dem problematischen Commit angefangen (`git checkout -b <branchname> HEAD`), auf den der Fix committed wird. Dieser kleine Bugfix-Branch kann nun in alle neueren Versionen ge`merge`t werden, ohne dass andere Änderungen mit übernommen werden oder dass der Fix transplantiert werden muss (siehe unten).

## Zusammenarbeit
Als dezentralisierte Versionskontrolle bietet git flexible Möglichkeiten zur Zusammenarbeit von Nutzern.

### git clone
Der initiale Schritt bei der Zusammenarbeit ist der Zugriff auf das Repository.
In dezentralisierten VCS wird dazu i. d. R. das gesamte Repository kopiert.
Diese Aktion nennt sich klonen und wird mit `git clone` ausgeführt.
Es ist möglich ein unvollständiges Repository mit ausschließlich den neuesten Commits/Trees/Blobs zu erstellen. Solch ein Repository nennt sich *shallow clone* und kann mit der Option `--depth=<tiefe>` erstellt werden. Diese Möglichkeit wird allerdings selten genutzt.

Der Zugriff kann auf verschiedene Art und Weise geschehen:

* über das Dateisystem: lokale Klone können erstellt werden und Netzwerkdateisysteme genutzt. Wenn Hardlinking zwischen Quelle und Ziel möglich ist, braucht der Objektspeicher nicht einmal zusätzlichen Speicher!
* über HTTP(S)
* über Secure Shell (SSH)
* über das git-Protokoll: dieses Protokoll wurde für effizientes Lesen großer Datenmengen vom SSH-Protokoll abgeleitet

Das Klonen legt automatisch eine benannte Gegenstelle (‘remote’) namens `origin` im Repository an, die im Folgenden benutzt werden kann. Zusätzliche Gegenstellen können mit `git remote` verwaltet werden.

### git fetch/pull
Wenn ein Repository bereits existiert, können neue Commits mit `git fetch <remote>` abgerufen werden. Dies aktualisiert auch die Remote-Branches.

Die Änderungen können anschließend bei Bedarf mithilfe von Merges mit den entsprechenden Remote-Branches übernommen werden.

Beides auf einmal wird von `git pull <remote> [<branch>]` durchgeführt. Dieser Befehl akzeptiert zusätzlich zu den Optionen von `fetch` auch die entprechenden Optionen für `merge`.

### git push
Wenn Schreibzugriff (via Dateisystem/HTTPS/SSH) besteht, können mithilfe `git push [<remote> [<branch>]]` Änderungen und Branches auf einen fremden Rechner übertragen werden.

Dies ist nur möglich, wenn dem Repository kein Arbeitsbereich zugeordnet ist.

Es kann außerdem möglich, dass durch nebenläufige Pushes anderer Repositorien der entfernte Branch nicht mehr der Vorgänger des neuen ist.
Hier wird normalerweise die Übertragung abgebrochen, um die nebenläufige Änderung nicht zu verlieren.
Zum Lösen dieser Situation kann der entsprechende Zweig vor dem erneuten `push` erneut ge`pull`t werden.

Alternativ, wenn die Situation durch Neuschreiben der Änderungsgeschichte entstanden ist, kann `git push --force` genutzt werden.
Es ist allerdings generell davon abzuraten, veröffentlichte Commit-Historie neuzuschreiben, da dies andere Mitarbeiter dazu zwingt, ihre Änderungen zu transplantieren (siehe `git rebase`).
Somit ist `git push --force` ohne weitere Koordination im Team eine schlechte Idee.

### Change Management
Die Übertragungsweise von Änderungen ist eine wichtige Entscheidung für ein git-Projekt.

#### Pull + Review-Modell
Die dezentrale Funktionsweise von git erlaubt es, Projekte ohne Verteilung von Schreibrechten auf fremde Rechner durchzuführen.
Hierbei wird eine Änderung von Person zu Person weitergegeben. Jeder Nutzer kann die Änderungen inspizieren, bevor sie weitergegeben werden.
Das fördert und erfordert Vertrauensverhältnisse zwischen den Projektteilnehmern.

Die Verteilung von Änderungen zum gesamten Team kann ad-hoc und diffus durchgefüht werden, oder zentralisierter wie im ‘Integration Manager Workflow’ oder dem ‘Benevolent Dictator Workflow’ (siehe <https://git-scm.com/book/en/v2/Distributed-Git-Distributed-Workflows>; letzterer ist das Modell der Linux-Kernel-Entwicklung)

In diesem Modell kann, sofern die Mitarbeiter in einem LAN arbeiten, `git daemon` genutzt werden, um Anderen Lesezugriff auf lokale Repositorien zu geben.
Ein rein technischer Vorteil dieses Vorgehens ist, dass in LANs sehr schnelle Übertragung möglich ist.

#### Push-Modell
Alternativ kann ein Server oder Dienst eingerichtet werden, auf den die Projektteilnehmer Schreibzugriff haben.
Dies ist in manchen geschlossenen Gesellschaften (z. B. Unternehmen) einfach, ist aber schwierig in Situationen mit mehreren Parteien mit verschiedenen Interessen (Open-Source, siehe <https://youtu.be/4XpnKHJAok8?t=18m11s>).

Der größte Vorteil ist Übersicht: es gibt eine eindeutige Stelle an der die kanonische Version der Projektdaten zu finden ist.

#### Mischformen
Die beiden vorangehenden Modelle können gemischt werden. Ein üblicher Workflow auf GitHub ist beispielsweise Folgendes:

Jeder Projektteilnehmer hat einen eigenen Bereich, auf den er/sie Schreibzugriff besitzt.
Änderungen werden jedoch über formale Pull Requests mit integrierter Diskussion auf das Gemeinschaftsrepositorium statt.
Somit befinden sich die Repositorien aller Teilnehmer an einem Ort (GitHub), ohne dass ein Projektteilnehmer Anderen Änderungen aufdrängen kann.

### git rebase
`git rebase` ist der bekannteste Vertreter einer Klasse von git-Befehlen, die die Commit-Historie nachträglich ändern.

Mit dem Befehl werden Änderungen einer Commit-Sequenz auf einen anderen Vorgänger ‚transplantiert’.

Dadurch kann beispielsweise bei einem durch nebenläufige Änderungen gescheiterten `push` eine lineare Entwicklungsgeschichte wiederhergestellt werden.

#### Kritik am History Rewriting
Die Nutzung von `rebase` ist kontrovers unter git-Nutzern.

Befürworter zeigen die einfacher strukturierte Historie, während Gegner anmerken, dass die durch `rebase` erstellten Commits niemals in der Form existiert haben und nicht getestet wurden.

----
Dieser Text wurde 2016 von Jona Stubbe in Deutschland unter [CC0](http://creativecommons.org/publicdomain/zero/1.0/) veröffentlicht
