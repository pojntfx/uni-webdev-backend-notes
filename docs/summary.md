---
author: [Felicitas Pojtinger]
date: "2023-01-28"
subject: "Uni Webdev Backend Summary"
keywords: [webdev-backend, hdm-stuttgart]
subtitle: "Summary for the webdev backend course at HdM Stuttgart"
lang: "de"
---

# Uni Webdev Backend Summary

## Meta

### Contributing

These study materials are heavily based on [professor Toenniessen's "Web Development Backend" lecture at HdM Stuttgart](https://www.hdm-stuttgart.de/bibliothek/studieninteressierte/bachelor/block?sgname=Mobile+Medien+%28Bachelor%2C+7+Semester%29&sgblockID=2573378&sgang=550041&blockname=Web+Development+Backend) and prior work of fellow students.

**Found an error or have a suggestion?** Please open an issue on GitHub ([github.com/pojntfx/uni-webdev-backend-notes](https://github.com/pojntfx/uni-webdev-backend-notes)):

![QR code to source repository](./static/qr.png){ width=150px }

If you like the study materials, a GitHub star is always appreciated :)

### License

![AGPL-3.0 license badge](https://www.gnu.org/graphics/agplv3-155x51.png){ width=128px }

Uni Webdev Backend Notes (c) 2023 Felicitas Pojtinger and contributors

SPDX-License-Identifier: AGPL-3.0
\newpage

## Themen der Vorlesung

1. **Einführung in Node.js** und einfache HTML-Fileserver
2. **RESTful Endpoints** mit Express.js
3. Die Template-Engine **EJS** und **Express-Sessions**
4. Datenbanken mit **MongoDB und Mongoose**
5. Bidirektionale Client-Server-Kommunikation mit **WebSocket**

## Einführung in Node.js

### Warum Node.js

- Node.js kann **auf dem Server** verwendet werden, im Gegensatz zum Browser
- JavaScript ist die am **häufigsten verwendete Sprache** im Web und durch die Arbeit im Frontend bekannt
- JavaScript eignet sich durch seinen **Event-Loop** besonders gut für HTTP-Server
- Non-Blocking IO ermöglicht es, viele **parallele Anfragen** zu verarbeiten
- Node.js ist **sehr schnell**
- **Einfach zu erlernen**, da das populäre Backend-Framework Express.js auf Node.js aufbaut.
- Aktuelle Version von Node.js ist **16.17.1 LTS (Long Term Support)**
- Bedeutende Anwender sind unter anderem **Microsoft, Yahoo, SAP, PayPal** und viele andere große Unternehmen verwenden Node.js irgendwo.
- Weitere Pakete können über den **Node Package Manager (npm)** hinzugefügt werden

**Vorteile:**

Sehr einfache APIs, **schnell zu lernen**

**Nachteile:**

- Etwas **kompliziertes Programmiermodell** (Event-basiert), typisch für JavaScript
- **Multithreading** in Node.js nur über Worker-Threads möglich

### Module

Node.js hat ein Modul-Konzept, das es ermöglicht, Funktionen und Variablen **in eigene Dateien auszulagern** und sie in anderen Dateien zu importieren.

Ein Beispiel dafür ist die Funktion `add(x,y)`, die in eine **separate Datei** namens 01d_Export.js ausgelagert wird:

```js
module.exports = function add(x, y) {
  return x + y;
};
```

In einer anderen Datei, z.B. 01d_AddiererFctModule.js, wird das Modul importiert und verwendet:

```js
const add = require("./01d_Export");
const a = 5,
  b = 7;
const s = add(a, b);
console.log(`${a} + ${b} = ${s}`);
```

- Mit Modulen kann der Code bei größeren Programmen übersichtlicher gestaltet werden und es ist **weniger fehleranfällig bei Änderungen** oder in der Wartung.
- Durch die Trennung des Codes in Module, **erhöht sich die Übersichtlichkeit** innerhalb des jeweiligen Moduls
- Einzelne Module können für sich finalisiert oder refaktoriert werden, **ohne dass es Auswirkungen auf den Rest des Codes** hat.
- Durch die Aufteilung des Codes in Module, wird es einfacher, die Arbeit unter Teammitgliedern aufzuteilen, was wiederum zu **weniger Merge-Konflikten** führt.

### Import & Export mit `require`/`module`

**Export-Varianten**:

a) Einzelne Methode oder Variable:

```js
module.exports = function add(x, y) {
  return x + y;
};
```

b) Mehrere Methoden oder Variablen über ein Objekt:

```js
module.exports = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
};
```

c) Mehrere einzelne Exports (mit der Convenience-Variable exports):

```js
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;
```

Wenn Sie jedoch **module.exports direkt zuweisen, werden alle vorherigen Exporte überschrieben.**

**Import-Varianten**:

a) Gesamtes Modul importieren:

```js
const fs = require("fs");
fs.readFile();

const readFile = require("fs").readFile;
readFile();
```

b) Destrukturierende Zuweisung:

```js
const { readFile } = require("fs");
readFile();

const { readFile, ...fs } = require("fs");
readFile();
fs.writeFile();
```

### Import & Export mit ES6

**Export**:

```js
export function add(x, y) {
  return x + y;
}

export function subtract(x, y) {
  return x - y;
}
```

**Default-Export**:

```js
export default (x, y) {
  return x - y;
}
```

**Import eines gesamten Moduls**:

```js
import * as math from "./math.js";

console.log(math.add(5, 2)); // Ausgabe: 7
console.log(math.subtract(5, 2)); // Ausgabe: 3
```

**Import mit destrukturierenden Zuweisung**:

```js
import { add as addition, subtract } from "./math.js";

console.log(addition(5, 2)); // Ausgabe: 7
console.log(subtract(5, 2)); // Ausgabe: 3
```

### Callbacks vs. Promises vs. Async/Await

**Callbacks:**

```js
const fs = require("fs");

fs.readFile("file.txt", function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

**Promises**:

```js
const fs = require("fs").promises;

fs.readFile("file.txt")
  .then((data) => console.log(data))
  .catch((err) => console.error(err));
```

**async/await**:

```js
const fs = require("fs").promises;

async function readFileExample() {
  try {
    const data = await fs.readFile("file.txt");
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}

readFileExample();
```

### Statischer Webserver

Mit Support für ein paar wenige MIME-Types:

```js
const http = require("http");
const fs = require("fs");
const { extname } = require("path");

const app = http.createServer((request, response) => {
  fs.readFile(__dirname + request.url, (err, data) => {
    const status = err ? 400 : 200;

    if (extname(request.url) == ".html")
      response.writeHead(200, { status, "Content-Type": "text/html" });
    if (extname(request.url) == ".js")
      response.writeHead(200, { status, "Content-Type": "text/javascript" });
    if (extname(request.url) == ".css")
      response.writeHead(200, { status, "Content-Type": "text/css" });

    response.write(data);
    response.end();
  });
});

app.listen(3000);

console.log("Listening on :3000");
```

Mit Support für ein alle MIME-Types:

```bash
$ npm install node-static
```

```js
const http = require("http");
const fileserver = new (require("node-static").Server)();

const app = http.createServer((request, response) => {
  fileserver.serve(request, response);
});

app.listen(3000);

console.log("Listening on :3000");
```

Mit Support für ein alle MIME-Types & Express:

```bash
$ npm install express
```

```js
const express = require("express");
const app = express();

app.use("/WDBackend", express.static(__dirname + "/public"));
app.listen(3000);

console.log("Listening on :3000");
```

### NPM: Pakete Installieren

- npm ist ein **Paketmanager** für Node.js (wie Maven bei Java oder pip bei Python)
- Mit npm können **Thirdy-Party-Libraries installiert** werden, die auf https://www.npmjs.com gesucht werden können.
- Installierte Pakete können **über `require` importiert** werden, ohne dass ein relativer Pfad angegeben werden muss.
- **Projektspezifische Installation**: `npm install paket-name` oder `npm i paket-name`
- **Globale Installation**: `npm i -g paket-name`
- **Installation von Entwicklungspaketen**: `npm i -D paket-name`
- `package-lock.json` enthält die **exakten Versionen** aller installierten Abhängigkeiten.
- Der `node_modules` Ordner enthält die **Dateien aller installierten Pakete.**
- Der `node_modules` Ordner sollte immer von **Git-Commits ausgeschlossen werden**
- Die `package-lock.json` sollte hingegen **immer committed** werden.

### NPM: `package.json`

- Kann mit `npm init` erstellt werden
- Unter `scripts` in der `package.json` Datei können Command-Line Befehle gespeichert werden, die später ausgeführt werden können, indem man sie in der Kommandozeile aufruft, z.B. `npm run start` oder `npm run test`.

### NPM: Paketauflösung

1. In einem **relativen Pfad** zur Datei, bis eine "package.json" Datei gefunden wird (npm Projekt Definition) und dort im "node_modules" Ordner
2. In den **global** installierten Paketen

**Best Practice**: Pakete sollten immer im Projekt installiert werden, damit dort alle Abhängigkeiten definiert sind. CLI-Tools können auch global, z.B. zur Projektinitialisierung, installiert werden.

```

```
