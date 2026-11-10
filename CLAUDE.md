# Anweisungen an den Agenten

## Vorschau (Technik, bitte stehen lassen)
- Die App ist index.html im Hauptordner und lädt ihre Daten per fetch aus daten/ (relativer Pfad).
- htmlpreview.github.io führt nur Skripte aus, die in der Seite selbst stehen. Bibliotheken (SheetJS, three.js, rhino3dm.js) per document.createElement('script') nachladen und erst nach onload benutzen. Kein <script src>, kein type="module", kein window.onload.
- Nach jedem Schritt committen, pushen und den Link mit Commit-Hash geben:
  https://htmlpreview.github.io/?https://github.com/BENUTZER/REPO/blob/COMMIT/index.html

<!-- Alles ab hier schreibt ihr selbst, gern zusammen mit dem Agenten:
     „Frag mich nacheinander, was in diese Datei gehört.“
     Was hinein gehört, steht auf https://tae-workshop.netlify.app
     im Abschnitt „Anweisungen an den Agenten“. -->

## Ziel
Eine App (index.html) für Tragwerksplaner:innen. Sie liest das Stahlbetonmodell aus
daten/tragwerksmodell/TAE_betonbau_Export.xlsx, weist den Strukturelementen
Randbedingungen zu und stellt das Modell mit diesen Randbedingungen in 3D dar.
- Randbedingungen sind Knotenlager (uX, uY, uZ, φX, φY, φZ: fest oder frei) und
  Stabendgelenke (Anfang/Ende).
- Für jede Zuweisung ist sichtbar, wie sie entschieden wurde (Algorithmus, KI, Mensch),
  mit Konfidenz und Begründung (siehe Entscheidungsleiter).

Etappen, jeweils ein eigener Schritt mit Prüfung:
1. Modell lesen: Struktur aller Blätter zeigen, dann Knoten, Stäbe, Querschnitte,
   Materialien und vorhandene Lager als Tabelle mit Summen (Anzahl, Länge, Masse).
2. 3D-Darstellung: Stäbe als Linien (three.js), gefärbt nach Querschnitt, anklickbar
   mit Details. Drehen, Zoomen, Ansicht zurücksetzen.
3. Elemente einordnen: Stütze oder Träger (bzw. Unterzug) je Stab, Fußknoten und
   Anschlussknoten erkennen.
4. Randbedingungen zuweisen: Lager an Knoten und Gelenke an Stabenden über die
   Entscheidungsleiter. Ergebnis als Tabelle, Vergleich mit Blatt 1.7 Knotenlager.
5. Randbedingungen im 3D-Modell: Lagersymbole und Gelenkmarker, Umschaltung nach
   Entscheidungsstufe, Klick-Details, Export der Zuweisungen als CSV.

## Arbeitsweise
- Immer nur eine Etappe. Danach anhalten, das Ergebnis zeigen (Link mit Commit-Hash)
  und auf meine Prüfung und Freigabe warten. Erst dann die nächste Etappe beginnen.
- Kleine Schritte, nachfragen statt raten. Offene Fragen vor dem Bauen stellen.
- Du führst alle Befehle selbst aus (git, Prüfungen). Ich nutze kein Terminal.
- Die App ist eine einzelne HTML-Datei ohne Server, kein Build-Schritt.
- Vor dem Pushen selbst prüfen: Seite im Browser öffnen, Konsole ohne Fehler,
  angezeigte Zahlen plausibel. Was du nicht prüfen konntest, sagst du dazu.
- Bestehenden Text in CLAUDE.md und AGENTS.md nicht ändern, nur ergänzen.
  Beide Dateien halten denselben Inhalt.
- Antworten und Texte in der App auf Deutsch.

## Zahlen
- Einheiten immer mitführen und anzeigen: Koordinaten und Längen in m, Fläche in cm²,
  Wichte in kN/m³, E-Modul in N/mm², Gewicht in kg.
- Werte aus der Excel-Datei unverändert übernehmen, nicht runden beim Rechnen,
  erst bei der Anzeige (Längen 2 Nachkommastellen).
- Nichts schätzen oder erfinden. Fehlt ein Wert, wird er als fehlend angezeigt.
- Jeder Wert hat eine Quelle, die in der App sichtbar ist (Datei und Blatt bzw. Spalte).
- Emissionsfaktoren nie schätzen. Falls die App Emissionen zeigt, kommen die Faktoren
  nur aus daten/gemeinsam/faktoren_oekobaudat.csv, angezeigt mit ÖKOBAUDAT-UUID und Version.
- Koordinatensystem wie RFEM: Z zeigt nach unten, negative Z-Werte liegen oben.
  Für die 3D-Ansicht wird Z gespiegelt, damit das Gebäude aufrecht steht.

## Entscheidungsleiter
Jede Randbedingung durchläuft drei Stufen:
1. Algorithmus: feste Regeln aus der Geometrie (z. B. Knoten auf der untersten Ebene
   Z = 0 mit nur Stützen darüber: Fußpunkt, eingespannt). Liefert Konfidenz 0 bis 1.
   Ab 0,8 wird das Ergebnis übernommen.
2. KI: unter 0,8 entscheidet die KI-Stufe mit Konfidenz und Begründung.
   Ab 0,7 wird das Ergebnis übernommen.
3. Mensch: darunter entscheide ich. Die App markiert diese Fälle und lässt mich wählen.
   Meine Entscheidung gilt mit Konfidenz 1,0 und wird mit Datum gespeichert.
Die Schwellen 0,8 und 0,7 sind unsere Festlegung und können unter
„Unsere Entscheidungen“ geändert werden.
Jedes Element in der App ist anklickbar und zeigt Stufe, Konfidenz, Regel bzw.
Begründung. Die Regeln stehen in daten/tragwerksmodell/regeln_randbedingungen.csv,
nicht im Code versteckt.

## KI-Stufe
- Es gibt keinen API-Schlüssel in der App. Du übernimmst beim Bauen die Rolle der KI:
  Ergebnis, Konfidenz und Begründung schreibst du als Daten in die App
  (daten/tragwerksmodell/ki_randbedingungen.csv), erkennbar als Stufe „KI“.
- Falls später ein Schlüssel genutzt wird: nur in .env, .env steht in .gitignore,
  nie in Code, Repo oder Chat.

## Fakten zu den Daten
Siehe daten/DATEN.md.

Für TAE_betonbau_Export.xlsx (geprüft):
- Blätter wie in DATEN.md beschrieben. Zeile 1 Titel, Zeilen 2–3 Kopf, Daten ab Zeile 4.
- 100 Knoten im Raster X = 0 / 7,5 / 15 / 22,5 / 30 m, Y = 0 / 6 / 12 / 18 m,
  Z = 0 / −3,8 / −7,1 / −10,4 / −13,7 m (Fuß plus vier Ebenen).
- 204 Stäbe, alle „Balkenstab“, je Stab eine Linie mit zwei Knoten („1,2“),
  keine Gelenke eingetragen.
- 6 Querschnitte (Rechtecke 500/500, 400/400, 350/700, 250/600, 250/500),
  2 Materialien (C30/37, C35/45, je 25 kN/m³).
- 1.7 Knotenlager: ein Lagersatz für die 20 Knoten auf Z = 0 (Liste mit Komma),
  alle sechs Freiheitsgrade fest („x“ = fest).
- Lastfälle LF1–LF5 und Kombinationen LK1–LK3 sind vorhanden, werden für die
  Randbedingungen nicht gebraucht.

## Unsere Entscheidungen
Hier tragen wir während des Bauens nach, was wir festgelegt haben (mit Datum).
Du schlägst einen Eintrag vor, sobald wir etwas entscheiden, und trägst ihn nach meinem OK ein.
- Datenquelle ist nur TAE_betonbau_Export.xlsx, nicht die .3dm-Datei.
- Blatt 1.7 Knotenlager dient als Vergleich für die Lager, nicht als Eingabe.
- Offen, bitte bestätigen:
  - Gehören neben Lagern und Gelenken weitere Randbedingungen dazu
    (z. B. Federn, Lasten)?
  - Sollen Fußpunkte eingespannt oder gelenkig angenommen werden, wenn die Regel
    nicht eindeutig ist?
  - Sind die Schwellen 0,8 (Algorithmus) und 0,7 (KI) so richtig?
