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
3. Stabfunktion über die Entscheidungsleiter: jeder Stab bekommt Stütze, Träger,
   Nebenträger oder Verband. Die App zeigt je Gruppe alle drei Stufen (Algorithmus,
   KI, Mensch), offene Fälle stehen oben. Schwellen in der App verschiebbar.
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
- Den Vorlagentext in CLAUDE.md und AGENTS.md (Abschnitt Vorschau, Kommentar,
  Überschriften, Verweis auf DATEN.md) nicht ändern. Unsere eigenen Abschnitte werden
  nachgeführt, wenn wir etwas neu festlegen. Beide Dateien halten denselben Inhalt.
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
Die Stabfunktion (Etappe 3) und später die Randbedingungen (Etappe 4) durchlaufen
je drei Stufen. Jede Stufe liefert eine Funktion bzw. Randbedingung und eine
Konfidenz zwischen 0 und 1.
1. Algorithmus: zwei bis vier eigene Regeln in einer CSV-Tabelle. Jede Bedingung hat
   eine Grenze und einen Übergang, z. B. `neigung>=75±15`.
   Konfidenz = 0,5 + Abstand zur Grenze / (2 · Übergang), begrenzt auf 0 bis 1:
   genau auf der Grenze 0,5, weit drin 1, weit draußen 0. Mehrere Bedingungen in
   einer Regel: der kleinste Wert zählt. Es gewinnt die Regel mit der höchsten
   Konfidenz. Ab 0,8 entscheidet der Algorithmus.
2. KI: darunter beschreibt die App den Stab (Richtung, Länge, Querschnitt, Lager,
   Gelenke, was an den Enden und im Feld anschließt) und gibt den Vorschlag des
   Algorithmus mit. Die KI antwortet mit Funktion, eigener Konfidenz und Begründung.
   Ab 0,8 entscheidet die KI.
3. Mensch: alles darunter steht in der App oben als offen. Ich entscheide dort von
   Hand. Meine Entscheidung gilt mit Konfidenz 1,0 und wird mit Datum gespeichert
   (im Browser, Export als mensch_funktion.csv; übernommene Entscheidungen liegen in
   daten/tragwerksmodell/mensch_funktion.csv).
Die App zeigt bei jeder Gruppe alle drei Stufen. Jedes Element ist anklickbar und
zeigt Stufe, Konfidenz, Rechnung bzw. Begründung. Die Schwellen lassen sich in der
App verschieben (Test), Standard 0,8 / 0,8.
Regeln liegen in CSV-Dateien, nicht im Code: daten/tragwerksmodell/regeln.csv
(Stabfunktion), später regeln_randbedingungen.csv (Randbedingungen).

## KI-Stufe
- Es gibt keinen API-Schlüssel in der App. Du übernimmst beim Bauen die Rolle der KI:
  Ergebnis, Konfidenz und Begründung schreibst du als Daten in die App
  (daten/tragwerksmodell/ki_funktion.csv für die Stabfunktion, später
  ki_randbedingungen.csv), erkennbar als Stufe „KI“. Antworten für alle Stäbe
  schreiben, damit das Verschieben der Schwellen funktioniert.
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
- 2026-11-10: Schwellen 0,8 für Algorithmus und 0,8 für KI (Vorgabe Workshop,
  ersetzt den ersten Vorschlag 0,7 für die KI).
- 2026-11-10: Jeder Stab bekommt eine Funktion: Stütze, Träger, Nebenträger oder
  Verband. Regeln R1–R4 in regeln.csv (Merkmale neigung [°], hoehe [mm] aus der
  Querschnittsbezeichnung b/h, laenge [m]).
- 2026-11-10: Gruppen = Richtung · Querschnitt · Lage (Rand, wenn beide Stabenden auf
  derselben Außenkante des Grundrisses liegen, sonst Innen).
- Offen, bitte bestätigen:
  - Gehören neben Lagern und Gelenken weitere Randbedingungen dazu
    (z. B. Federn, Lasten)?
  - Sollen Fußpunkte eingespannt oder gelenkig angenommen werden, wenn die Regel
    nicht eindeutig ist?
  - Innere Querbalken in Y (250/500, 36 Stäbe): Nebenträger oder Träger? Die KI ist
    unsicher (0,60), weil die Deckenspannrichtung im Modell fehlt.
