# Recepten — Munnichs kookboek

De app draait op <https://robmun.github.io/recepten/> en wordt vanuit deze repository gepubliceerd
door GitHub Pages. Alles wat in de hoofdmap staat is wat de telefoons ophalen.

## Wat staat waar

| Bestand | Wat het is |
| --- | --- |
| `index.html` | **De app.** Eén bestand van ongeveer 750 kB: HTML, CSS, alle code, de Firebase-SDK en de iconen. Dit bestand wordt gebouwd — nooit met de hand aanpassen. |
| `version.json` | Wat de app ophaalt om te zien of er een nieuwe versie is. Moet gelijklopen met `APP_VERSION` en `APP_DATE` in de bron. |
| `vorige/` | De vorige release (`index.html` en `version.json`), live op <https://robmun.github.io/recepten/vorige/>. Zo is er altijd een werkende versie als de nieuwe hapert. |
| `manifest.webmanifest`, `icon-*.png` | Voor het zetten op het beginscherm (PWA). |
| `bron/src.html` | **De bron.** Hier wordt aan gewerkt. |
| `bron/build.py` | Maakt `index.html` uit `bron/src.html`. |
| `bron/firestore.rules` | De toegangsregels van de database. Deze staan los van de app en worden in de Firebase-console geplakt (project `recepten-7d212` › Firestore › Rules). |
| `bron/firebase/` | Waar de ingebakken Firebase-SDK vandaan komt. Alleen nodig bij een SDK-upgrade. |
| `bron/test/` | Nagemaakte Firebase-backend voor de geautomatiseerde tests. |

## Een nieuwe versie uitbrengen

1. Kopieer eerst de huidige, nog live versie naar `vorige/`:
   `cp index.html version.json vorige/`
   Doe dit bij élke release, vóór het bouwen, en commit het mee.
2. Pas `bron/src.html` aan.
3. Zet bovenaan het versieblok `APP_VERSION` en `APP_DATE` op de nieuwe waarde.
   Het schema is **jaar.maand.volgnummer**: `26.9.30` is de dertigste versie van
   september 2026. Zet de wijziging ook vooraan in `APP_LOG`.
4. Bouwen: `python3 bron/build.py`
5. `version.json` bijwerken met hetzelfde versienummer, dezelfde datum en één of
   twee regels over wat er nieuw is. De app vergelijkt **eerst de datum**, dan het
   versienummer, dus een lagere datum betekent geen update.
6. Committen en pushen naar de hoofdtak, met een tag `v<versie>` (bijvoorbeeld `v26.9.31`). GitHub Pages zet het binnen een minuut live.
7. De app op de telefoons merkt het vanzelf en toont een balk *"Nieuwe versie"*.

Wijzigen de databaseregels? Dan die apart in de Firebase-console plakken **voordat**
de nieuwe `index.html` live gaat, anders lopen de toestellen tegen een weigering aan.

## Terugdraaien

Snelle uitweg: <https://robmun.github.io/recepten/vorige/> draait altijd de vorige
release. Die kopie haalt haar eigen `vorige/version.json` op, dus ze gaat niet zelf
om een update vragen. Ze gebruikt dezelfde Firebase-database als de gewone app.

`git revert <commit>` en pushen. Omdat `index.html` één bestand is, is er nooit een
halve versie live. Let op: zet `version.json` dan ook terug, anders blijven de
toestellen denken dat ze achterlopen.

## Testen

De tests draaien met Playwright tegen een nagemaakte Firebase (`bron/test/`), zodat er
nooit met de echte gegevens van het gezin wordt gerommeld. Ze starten een Chromium,
loggen in als twee of drie gezinsleden tegelijk en controleren wat ieder van hen ziet.

```
python3 -m http.server 8765      # in de hoofdmap
node <test>.js                   # in een tweede terminal
```

## Achtergrond

- Firebase-project `recepten-7d212`, gratis Spark-plan. Aanmelden staat uit: nieuwe
  gezinsleden worden in de Firebase-console aangemaakt.
- Gedeeld zijn: de recepten (`recipes`), de foto's (`photos`, `thumbs`) en de
  persoonlijke laag die iedereen mag zien (`gedeeld/{uid}`: notities, duimen, gemaakt).
- Privé zijn: favorieten, de boodschappenlijst en de volgorde van de winkelroute
  (`users/{uid}/prive/data`).
- Alle e-mailadressen in de code staan als FNV-hash, op dat van de beheerder na.
- De map `bron/` wordt door GitHub Pages ook uitgeserveerd. Dat is geen bezwaar —
  het is dezelfde code die in `index.html` staat — maar zet er dus nooit sleutels in.
