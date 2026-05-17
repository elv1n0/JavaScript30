# JavaScript Clock – Erklärung des gesamten Codes

## Grundidee der Uhr

Die Uhr besteht aus drei Zeigern:

- Sekundenzeiger
- Minutenzeiger
- Stundenzeiger

Jeder Zeiger ist ein normales HTML-Element (`div`), das mit CSS rotiert wird.

Die Rotation passiert mit:

```js
element.style.transform = `rotate(Xdeg)`;
```

---

# HTML-Struktur

```html
<div class="clock">
  <div class="clock-face">
    <div class="hand hour-hand"></div>
    <div class="hand min-hand"></div>
    <div class="hand second-hand"></div>
  </div>
</div>
```

Jeder Zeiger besitzt:
- die Klasse `hand`
- zusätzlich eine eigene Klasse (`hour-hand`, `min-hand`, `second-hand`)

Dadurch können wir die Zeiger später in JavaScript auswählen.

---

# Zeiger auswählen

```js
const secondHand = document.querySelector('.second-hand');
const minuteHand = document.querySelector('.min-hand');
const hourHand = document.querySelector('.hour-hand');
```

`querySelector()` sucht das passende HTML-Element im Dokument.

---

# Die setDate()-Funktion

```js
function setDate() {

}
```

Diese Funktion:
- holt die aktuelle Uhrzeit
- berechnet die Winkel
- dreht die Zeiger

---

# Aktuelle Zeit holen

```js
const now = new Date();
```

`new Date()` erstellt ein Datums-/Zeitobjekt.

---

# Sekunden holen

```js
const seconds = now.getSeconds();
```

Liefert Werte von:

```txt
0 - 59
```

---

# Minuten holen

```js
const minutes = now.getMinutes();
```

Liefert:

```txt
0 - 59
```

---

# Stunden holen

```js
const hours = now.getHours();
```

Liefert:

```txt
0 - 23
```

---

# Winkel berechnen

## Sekundenzeiger

```js
const secondsDegrees =
  ((seconds / 60) * 360) + 90 + secondExtraRotation;
```

## Erklärung

### Schritt 1

```js
seconds / 60
```

Wandelt Sekunden in einen Prozentwert um.

Beispiel:

```txt
30 / 60 = 0.5
```

---

### Schritt 2

```js
(seconds / 60) * 360
```

Wandelt den Prozentwert in Grad um.

Beispiel:

```txt
0.5 * 360 = 180deg
```

---

### Schritt 3

```js
+ 90
```

Verschiebt den Startpunkt.

Ohne `+90` würde der Zeiger bei `0deg`
nach rechts zeigen.

Mit `+90` startet er korrekt auf 12 Uhr.

---

### Schritt 4

```js
+ secondExtraRotation
```

Verhindert den 360°-Sprung beim Übergang von:

```txt
59 -> 0
```

---

# Minutenzeiger

```js
const minutesDegrees =
  ((minutes / 60) * 360) +
  ((seconds / 60) * 6) +
  90 +
  minuteExtraRotation;
```

---

## Warum Sekunden berücksichtigen?

Ohne Sekunden:
- springt der Minutenzeiger jede Minute

Mit Sekunden:
- bewegt er sich weich/flüssig

---

## Warum `* 6`?

Ein Kreis hat:

```txt
360°
```

Eine Minute hat:

```txt
60 Sekunden
```

Also:

```txt
360 / 60 = 6°
```

Jede Sekunde bewegt den Minutenzeiger also um 6 Grad.

---

# Stundenzeiger

```js
const hoursDegrees =
  ((hours / 12) * 360) +
  ((minutes / 60) * 30) +
  90 +
  hourExtraRotation;
```

---

## Warum Minuten berücksichtigen?

Ohne Minuten:
- springt der Stundenzeiger jede volle Stunde

Mit Minuten:
- bewegt er sich langsam/flüssig

---

## Warum `* 30`?

12 Stunden auf 360° verteilt:

```txt
360 / 12 = 30°
```

Eine Stunde entspricht also 30 Grad.

---

# Der 360°-Bug

## Das Problem

Der Zeiger springt von:

```txt
444deg -> 90deg
```

CSS versucht diesen großen Rücksprung zu animieren.

Dadurch dreht sich der Zeiger einmal komplett herum.

---

# Lösung: Extra Rotation

## Variablen

```js
let secondExtraRotation = 0;
let minuteExtraRotation = 0;
let hourExtraRotation = 0;
```

Diese Variablen speichern zusätzliche volle Umdrehungen.

---

# Vorherige Werte speichern

```js
let previousSeconds = new Date().getSeconds();
let previousMinutes = new Date().getMinutes();
let previousHours = new Date().getHours();
```

Diese Werte merken sich:
- welchen Wert der Zeiger beim letzten Durchlauf hatte

---

# Übergänge erkennen

## Sekunden

```js
if (seconds === 0 && previousSeconds === 59) {
  secondExtraRotation += 360;
}
```

Das bedeutet:

```txt
59 -> 0
```

Der Zeiger hat eine komplette Runde gemacht.

Darum:

```js
+= 360
```

---

# Warum funktioniert das?

Ohne Extra Rotation:

```txt
444deg -> 90deg
```

Mit Extra Rotation:

```txt
444deg -> 450deg
```

Da:

```txt
450deg = 90deg + 360deg
```

bleibt der Zeiger optisch an derselben Position,
aber CSS sieht eine kleine Vorwärtsbewegung.

Dadurch bleibt die Animation weich.

---

# Warum speichern wir previousSeconds = seconds?

Am Ende:

```js
previousSeconds = seconds;
previousMinutes = minutes;
previousHours = hours;
```

Dadurch wird:
- der aktuelle Wert
- zum vorherigen Wert
- für den nächsten Funktionsaufruf

Beispiel:

## Jetzt

```txt
seconds = 37
previousSeconds = 36
```

Am Ende:

```js
previousSeconds = seconds;
```

Dann:

```txt
previousSeconds = 37
```

Beim nächsten Durchlauf kann JavaScript vergleichen:

```txt
37 -> 38
```

oder:

```txt
59 -> 0
```

---

# Zeiger drehen

```js
secondHand.style.transform =
  `rotate(${secondsDegrees}deg)`;
```

Dadurch wird der Zeiger tatsächlich bewegt.

Dasselbe passiert bei:
- Minuten
- Stunden

---

# Warum setDate() zusätzlich aufrufen?

```js
setDate();
setInterval(setDate, 1000);
```

---

## Problem ohne setDate()

`setInterval()` startet erst nach 1 Sekunde.

Die Uhr wäre also kurz falsch.

---

## Lösung

```js
setDate();
```

führt die Funktion sofort einmal aus.

Danach übernimmt:

```js
setInterval()
```

die regelmäßigen Updates.

---

# Finale Version der Uhr

```js
const secondHand = document.querySelector('.second-hand');
const minuteHand = document.querySelector('.min-hand');
const hourHand = document.querySelector('.hour-hand');

let previousSeconds = new Date().getSeconds();
let previousMinutes = new Date().getMinutes();
let previousHours = new Date().getHours();

let secondExtraRotation = 0;
let minuteExtraRotation = 0;
let hourExtraRotation = 0;

function setDate() {
  const now = new Date();

  const seconds = now.getSeconds();
  const minutes = now.getMinutes();
  const hours = now.getHours();

  if (seconds === 0 && previousSeconds === 59) {
    secondExtraRotation += 360;
  }

  if (minutes === 0 && previousMinutes === 59) {
    minuteExtraRotation += 360;
  }

  if (hours === 0 && previousHours === 23) {
    hourExtraRotation += 360;
  }

  const secondsDegrees =
    ((seconds / 60) * 360) +
    90 +
    secondExtraRotation;

  const minutesDegrees =
    ((minutes / 60) * 360) +
    ((seconds / 60) * 6) +
    90 +
    minuteExtraRotation;

  const hoursDegrees =
    ((hours / 12) * 360) +
    ((minutes / 60) * 30) +
    90 +
    hourExtraRotation;

  secondHand.style.transform =
    `rotate(${secondsDegrees}deg)`;

  minuteHand.style.transform =
    `rotate(${minutesDegrees}deg)`;

  hourHand.style.transform =
    `rotate(${hoursDegrees}deg)`;

  previousSeconds = seconds;
  previousMinutes = minutes;
  previousHours = hours;
}

setDate();
setInterval(setDate, 1000);
```

