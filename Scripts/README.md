# REAPER Region Performance Scripts

Dieses Verzeichnis enthaelt zwei unterschiedliche Live-Workflows fuer REAPER.

## Empfohlen: Queue + Envelopes

Dieser Workflow ist fuer Sessions gedacht, in denen:

- alle FX dauerhaft online bleiben
- alle Szenenwechsel als REAPER-Envelopes gezeichnet sind
- Regions nur als Formteile / Sprungziele dienen
- der Transport moeglichst ohne Stop/Start durchlaeuft

Passende Skripte:

- `JS_AmbiEncoder64_Motion_Map_GUI.lua`
- `JS_Write_AmbiEncoder64_Spat_Motion_Automation.lua`
- `JS_Queue_Region_By_Time_Selection_Or_Cursor.lua`
- `JS_Queue_Region_X_GUI.lua`
- `JS_Envelope_Region_Stepper_GUI.lua`
- `JS_Manual_Clickfree_Region_Stepper.lua`
- `JS_Apply_Send_Fades_To_All_Regions.lua`
- `JS_Region_Send_Fade_Writer_GUI.lua`
- `JS_Region_Fade_In_Out_GUI.lua`
- `JS_Export_Selected_Track_Plugin_And_Send_Automation.lua`
- `JS_Import_Selected_Track_Plugin_And_Send_Automation.lua`
- `JS_Export_Selected_Tracks_Send_Automation.lua`

Typischer Ablauf:

1. Projekt laden
2. Play einmal starten
3. waehrend Region A laeuft Region B waehlen
4. `Queue Region X` ausloesen
5. REAPER springt am Ende der aktuellen Region weich zur Zielregion

Alternative im selben Envelope-Workflow:

- `JS_Envelope_Region_Stepper_GUI.lua`
- `JS_Manual_Clickfree_Region_Stepper.lua`

Diese Variante spielt immer nur eine Region, stoppt am Regionsende, bereitet
die naechste Region vor und wartet auf manuelles `Enter` / `Play`.

`JS_Manual_Clickfree_Region_Stepper.lua` ist die schlanke No-GUI-Version:

1. Script einmal starten
2. Region ueber Cursor oder Time Selection waehlen
3. manuell `Play`
4. am Regionsende stoppt REAPER automatisch
5. naechste Region wird vorbereitet
6. wieder manuell `Play`

Script erneut starten = Stop / Deaktivieren.

## Legacy: Snapshot / Stop / Restart

Diese Skripte stammen aus einem aelteren Workflow, in dem:

- SWS Snapshots pro Region vorbereitet werden
- am Regionsende gestoppt wird
- danach die naechste Region armed auf manuellen Neustart wartet

Diese Skripte sind weiterhin nuetzlich, wenn du bewusst mit Stop/Restart und
Snapshot-Recall arbeiten willst, passen aber nicht ideal zu einem
durchlaufenden Queue-Workflow.

Legacy-Skripte:

- `JS_Auto_Arm_Snapshot_For_Manual_Region_Trigger.lua`
- `JS_Manual_Region_Stepper_Auto_Arm_Snapshot.lua`
- `JS_Manual_Region_Stepper_Auto_Arm_Snapshot_GUI.lua`
- `JS_Snapshot_Arm_And_Play_No_Latency_GUI.lua`
- `JS_Region_Cue_Player_No_Click_GUI.lua`

## Empfehlung

Wenn deine Session jetzt komplett envelope-basiert ist, nutze im Alltag nur den
Queue-Block oben und behandle den Snapshot-Block als Archiv / Referenz.

## Automation Export / Import

Fuer Plugin-Parameter- und Send-Automation gibt es ausserdem:

- `JS_Export_Selected_Track_Plugin_And_Send_Automation.lua`
- `JS_Import_Selected_Track_Plugin_And_Send_Automation.lua`
- `JS_Export_Selected_Tracks_Send_Automation.lua`

Workflow:

1. Quell-Track waehlen
2. Export-Script starten
3. Datei speichern
4. Ziel-Track vorbereiten
5. Sicherstellen, dass die benoetigten Plugin- und Send-Envelopes dort bereits existieren
6. Import-Script starten

Wichtig:

- Exportiert werden nur Plugin- und Send-Envelopes des ausgewaehlten Tracks
- Importiert wird auf vorhandene passende Ziel-Envelopes
- Fehlende Ziel-Envelopes werden bewusst nicht automatisch per Chunk-Hack erzeugt

Nur Send-Automation von mehreren selektierten Tracks sichern:

- `JS_Export_Selected_Tracks_Send_Automation.lua`

## AmbiEncoder_64 Spat Motion Automation

Empfohlen fuer die Bedienung:

- `JS_AmbiEncoder64_Motion_Map_GUI.lua`

Die GUI zeigt die Quellen als `S0` bis `S63`. Intern schreibt das Script weiter
auf die einsbasierten ICST/OSC-Parameterbloecke, also `S0 -> OSC/ICST Source 1`,
`S1 -> OSC/ICST Source 2`. Pro Source kannst du links mit `On` festlegen, ob sie
geschrieben wird, und daneben eine Motion-Form anklicken. So sind z. B. nur
`S0` und `S9` moeglich. Preset-Buttons setzen
alle Sources oder Gruppen schnell auf `Auto`, `Random`, `All Line`,
`All Circle`, `All Step`, `1-8 Arc` oder `9-16 Circle`; `All Src`, `None Src`,
`1-8 Src` und `9-16 Src` steuern die aktive Source-Auswahl. `Clear Sel` leert
alle Motion-Zuweisungen und deaktiviert alle Sources, damit du die Muster
danach komplett manuell neu anklicken kannst. Der Button
`Write Automation + Region` nutzt intern die Writer-Action und schreibt danach
XYZ-Automation plus Region.

Die GUI ist direkt an Source-Indexe gebunden. Sie schreibt also weiterhin auf
das feste ICST-Layout:

```text
S1 = X/Y/Z 11/12/13
S2 = X/Y/Z 16/17/18
S3 = X/Y/Z 21/22/23
```

`Motion amount` skaliert die Auslenkung der gewaehlten Bewegungsform um den
Center herum. `1.0` entspricht der normalen X/Y/Z-Spannweite, `2.0` macht
Zigzag, S-Kurven und andere Muster deutlich sichtbarer. Werte werden am Ende
formbewahrend begrenzt, damit Kreis und Spirale nicht durch hartes Clipping zu
Square-Formen werden.

`Use Z motion` steuert, ob die Bewegungsform auch die Z-Achse moduliert. Wenn
der Toggle aus ist, bleibt Z konstant auf `Z center`; X/Y schreiben weiterhin
die gewaehlte Form. Circle und Spiral verwenden fuer X/Y einen gemeinsamen
Radius, damit sie in der Automation nicht als Oval erscheinen.

Die schlanke Textdialog-Action bleibt als Fallback:

`JS_Write_AmbiEncoder64_Spat_Motion_Automation.lua` schreibt automatisch
verschiedene Spatial-Bewegungen fuer bis zu 64 Quellen in die FX-Envelopes von
`ICST AmbiEncoder_64`. Die aktuelle Version erkennt die XYZ-Parameter aus dem
Plugin-Fenster, z. B. `Point 1: X`, `Point 1: Y`, `Point 1: Z`.
Zusaetzlich ist das feste ICST-Layout als Fallback hinterlegt:

```text
Source 1: X/Y/Z = UI-Parameter 11/12/13
Source 2: X/Y/Z = UI-Parameter 16/17/18
Source 3: X/Y/Z = UI-Parameter 21/22/23
...
Source n: X/Y/Z = 11/12/13 + ((n - 1) * 5)
```

Workflow:

1. Track mit `ICST AmbiEncoder_64` selektieren
2. Loop / Time Selection fuer die gewuenschte Bewegungsdauer setzen
3. Script als REAPER-Action starten
4. Sources, Steps/sec, Bewegungstypen, X/Y/Z-Spannweiten und Region-Namen
   bestaetigen

Die Source-Auswahl akzeptiert z. B. `1-64`, `1-8` oder `1 3 7 12`. Das Script
sucht passende Parameter anhand von Namen mit `X`, `Y`, `Z`, `azim`, `elev`,
`dist` oder `radius` und Source-Nummern. Wenn die Parameter pro Typ sauber
sortiert sind, funktioniert auch die Reihenfolge als Fallback.

`Motion map` verteilt Bewegungsformen pro Source. `auto` verteilt automatisch
mehrere Formen. Einzelne Zuweisungen funktionieren so:

```text
1=line 2=arc_up 3=zigzag 4=circle
```

Auch Bereiche sind moeglich:

```text
1-8=line 9-16=arc_up 17-32=step 33-64=spiral
```

Verfuegbare Formen:

- `line`: diagonale Linie
- `arc_up`: Bogen nach oben
- `arc_down`: Bogen nach unten
- `s_curve`: S-Kurve
- `step`: Treppenbewegung
- `zigzag`: Zickzack
- `circle`: Kreis / Umlauf
- `spiral`: Spiralbewegung
- `fourier_xyz`: additive XYZ-Fourierkurve
- `heart_curve`: Herzkurve
- `cardioid`: Cardioid / Nierenkurve
- `rose8`: Rosenkurve mit 8 Blaettern
- `bernoulli`: Lemniskate von Bernoulli
- `astroid`: Astroid
- `epicycloid`: Epizykloide
- `lissajous`: gekreuzte Sinusbewegung

Alle Formen werden auf die aktuelle Loop / Time Selection skaliert: Startpunkt
liegt am Anfang der Selection, Endpunkt am Ende der Selection.

`Region name` erzeugt nach dem Schreiben automatisch eine REAPER-Region exakt
ueber der aktuellen Time Selection. Wenn dort bereits eine Region mit exakt
gleicher Start- und Endposition liegt, wird diese umbenannt statt doppelt
angelegt. `Overwrite region = yes` aktualisiert stattdessen eine vorhandene
Region mit gleichem Namen auf die aktuelle Time Selection und entfernt weitere
gleichnamige Dubletten. Ein leerer Region-Name ueberspringt diesen Schritt.
Die Region ist der Render-Container fuer B-Format-Exports; die Automation
bleibt timeline-basiert auf dem AmbiEncoder-Track.

Bei XYZ-Parametern werden die Dialogfelder so interpretiert:

- `X/Az center`: X-Mitte als Azimuth-Offset, `0` entspricht X = `0.5`
- `X/Az spread`: X-Bewegungsbreite, `360` entspricht voller X-Breite
- `Y/El center`: Y-Mitte als Elevation-Offset, `0` entspricht Y = `0.5`
- `Y/El spread`: Y-Bewegungsbreite, `180` entspricht voller Y-Breite
- `Z/Dist center`: Z-Mitte direkt im Plugin-Wertebereich `0..1`
- `Z/Dist spread`: Z-Bewegungsbreite direkt im Plugin-Wertebereich `0..1`

Standardmaessig:

- vorhandene Punkte in der Time Selection werden geloescht
- eine vorhandene gleichnamige Region wird fuer wiederholte Takes ueberschrieben
- der selektierte Track wird auf `Latch` gesetzt
- globaler Automation Override wird deaktiviert; nur der selektierte Track wird
  auf `Latch` gesetzt
- vorhandene Envelope-Lanes auf dem Track werden ausgeblendet; sichtbar bleiben
  danach nur die neu geschriebenen Ziel-Envelopes
- pro Ziel-Envelope wird ein 5-ms End-Guard geschrieben, damit der FX am Ende
  sauber auf dem Zielwert bleibt; vor dem Selection-Start wird kein Extra-Punkt
  geschrieben
- eine Region `BFormat_TS` wird ueber die Time Selection geschrieben
- Quellen bekommen automatisch unterschiedliche Kurvenfamilien:
  Line, Arc, S-Kurve, Step, Zigzag, Kreis, Spiral und Lissajous
