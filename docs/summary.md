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

## RESTful Endpoints mit Express.js

### Warum REST?

**Jahr 2000**: Überlastung der Web-Backends (Server)

- **Client**: Wenig JavaScript
- **Backend**:
  - HTML-Seiten (statisch)
  - **Rendering von HTML und JSON**
  - Zustand aller User-Dialoge
  - Datenbank-Zugriffe
  - **Komplette Dialogsteuerung** und Kontrolle auf dem Server

**Heute**: Zustandslose Web-Backends (Server)

- **Client**: Viel mehr JavaScript (Frameworks)
- **Backend**:
  - Datenbankzugriffe mit Rückgabe von **JSON-Objekten**
  - **Keine Zustandsverwaltung** einzelner User mehr

### Was ist REST?

- REST steht für **"Representational State Transfer"**
  - **Repräsentation**: Darstellung einer Ressource (Daten + Metadaten)
  - **State**: Zustand der Anwendung, gegeben durch die Gesamtheit aller Repräsentationen der angezeigten Daten.
  - **Transfer**: Zustandsübergang durch Aufruf einer Ressource.
- **Roy Fielding** hat es in seiner Doktorarbeit im Jahr 2000 vorgestellt.
- REST **ist ein Architekturparadigma** zur Vereinfachung von verteilten Systemen.
- Es betont ...
  - **Skalierbarkeit** der Komponenteninteraktionen
  - Generierung von **Interfaces**
  - **Unabhängige Bereitstellung** von Komponenten
  - Verwendung von **Zwischenkomponenten** um die Interaktionslatenz zu reduzieren, die Sicherheit durchzusetzen und Legacy-Systeme zu encapsulieren.
- REST hat **ursprünglich keine Beziehung zu HTTP** oder speziell gestalteten URLs.

### Merkmale einer REST-Architektur

- Client-Server-Modell.
  Zustandslos: Jeder Request enthält alle Informationen zur Ausführung
- Einheitliche Schnittstelle für die Erstellung, Abfrage, Aktualisierung und Löschung von Ressourcen:
  - **POST**: Erstellen
  - **GET**: Abfragen
  - **PUT**: Aktualisieren
  - **DELETE**: Löschen
- **Ressourcen** sind das zentrale Konzept in REST:
  - Datensätze aus einer Datenbank
  - Textdateien
  - Grafiken
  - Videos
  - Audio-Clips
  - PDF-Dokumente
  - HTML-, CSS- und JS-Dateien von einer Web-Anwendung
  - Services einer SOA
- Jede Ressource hat eine **eindeutige URI/URL**
- Jede Ressource trägt **Caching-Informationen**

### Idempotente Schnittstellen

Sichere und idempotente Schnittstellen:

- **GET**: Read auf eine Ressource
- **PATCH/PUT**: Update auf die Ressource
- **DELETE**: Delete einer Ressource
- **HEAD**: Austausch von Request- und Response-Headern als Zusatzinformation für die übermittelten Daten/Ressourcen (content-size, last-modified, content-type etc.)
- **OPTIONS**: Was kann mit einer Ressource gemacht werden? (Meta-information über mögliche HTTP-Verben.)

Unsichere und nicht-idempotente Schnittstelle: **POST** (Create auf eine Ressource). Im Gegensatz zu DELETE können hier nach einem erneuten Anruf ohne Checks weitere Objekte erstellt werden.

### Einheitliche Schnittstellen

- Der **Pfad** einer URL kann statisch sein, z.B. /users, oder Plural.
- **URL-Parameter** sind variabel und werden in der Regel verwendet, um eine eindeutige Identifizierung der Ressource zu ermöglichen.
- **Query-Parameter** sind optionale Key-Value Paare, die in der Regel nur bei GET-Anfragen verwendet werden. Sie ermöglichen z.B. das sortieren von Ressourcen nach bestimmten Kriterien.
- Der **HTTP-Body** wird in der Regel verwendet, um JSON-Daten bei Anfragen wie PUT, POST, PATCH oder DELETE zu übertragen.

### Einheitliche Schnittstellen mit REST

**Pfad**:

```plaintext
http://127.0.0.1:3000/fruits
```

```js
const DATA = [
  { id: 1, name: "Apfel", color: "gelb,rot" },
  { id: 2, name: "Birne", color: "gelb,grün" },
  { id: 3, name: "Banane", color: "gelb" },
];

app.get("/fruits", (req, res) => {
  res.send(DATA);
});
```

**URL-Parameter**:

```plaintext
http://127.0.0.1:3000/fruits/2
```

```js
const DATA = [
  { id: 1, name: "Apfel", color: "gelb,rot" },
  { id: 2, name: "Birne", color: "gelb,grün" },
  { id: 3, name: "Banane", color: "gelb" },
];

app.get("/fruits/:id", (req, res) => {
  const id = parseInt(req.params.id);

  const item = DATA.find((o) => o.id === id);

  res.send(item);
});
```

**Query-Parameter**:

```plaintext
http://127.0.0.1:3000/fruits/2
```

```js
const DATA = [
  { id: 1, name: "Apfel", color: "gelb,rot" },
  { id: 2, name: "Birne", color: "gelb,grün" },
  { id: 3, name: "Banane", color: "gelb" },
];

app.get("/fruits", (req, res) => {
  const id = parseInt(req.query.id);

  const item = DATA.find((o) => o.id === id);

  res.send(item);
});
```

**HTTP-Body (URL-Encoded)**

> Nutze `express.urlencoded`

```js
app.use(express.urlencoded({ extended: true }));

const DATA = [{ id: 1, name: "Apfel", color: "gelb,rot" }];

app.post("/fruits", (req, res) => {
  const { name, color } = req.body;

  if (DATA.find((o) => o.name === name)) {
    res.send("Duplicate name");
  } else {
    const id = Math.max(...DATA.map((o) => o.id)) + 1;
    const fruit = { id, name, color };
    DATA.push(fruit);
    res.send(fruit);
  }
});
```

**HTTP-Body (JSON)**

> Nutze `express.json`

```js
app.use(express.json());

const DATA = [{ id: 1, name: "Apfel", color: "gelb,rot" }];

app.post("/fruits", (req, res) => {
  const { name, color } = req.body;

  if (DATA.find((o) => o.name === name)) {
    res.send("Duplicate name");
  } else {
    const id = Math.max(...DATA.map((o) => o.id)) + 1;
    const fruit = { id, name, color };
    DATA.push(fruit);
    res.send(fruit);
  }
});
```

### Routenpfade in Express

```js
app.get("/ab?cd", function (req, res) {
  res.send("ab?cd");
}); // acdabcd

app.get("/ab+cd", function (req, res) {
  res.send("ab+cd");
}); // abcdabbbbcd

app.get("/ab.*cd", function (req, res) {
  res.send("ab.*cd");
}); // abcdabxcd

app.get("/ab(cd)?e", function (req, res) {
  res.send("ab(cd)?e");
});

app.get(/a/, function (req, res) {
  res.send("/a/");
}); // alles mit 'a' drin

app.get(/.*fly$/, function (req, res) {
  res.send("/.*fly$/");
});
```

### Middleware in Express

**Wildcard-Route**:

```js
app.all(/.*/, (req, res, next) => {
  console.log(`wildcard-route: ${req.method} ${req.url}`);
  next();
});
```

**Middleware** (empfohlen):

```js
app.use((req, res, next) => {
  console.log(`middleware: ${req.method} ${req.url}`);
  next();
});
```

Die `next()`-Methode führt immer den nächsten passenden Routen-Handler aus.

### Mehrere Callback-Handler

```js
let cb0 = function (req, res, next) {
  console.log("CB0");
  next();
};

let cb1 = function (req, res, next) {
  console.log("CB1");
  next();
};

let cb2 = function (req, res) {
  res.send("Hello from CB2!");
};

app.get("/example/c", [cb0, cb1, cb2]);
```

Wichtig: `next` nicht vergessen!

### Chaining Routes

Mehrere HTTP-Verben für eine Route können mithilfe von Chaining Routes zusammengefasst werden.

```js
app
  .route("/books")
  .get(function (req, res) {
    res.send("Get all books");
  })
  .post(function (req, res) {
    res.send("Add a book");
  });

app
  .route("/books/:id")
  .put(function (req, res) {
    res.send("Update the book");
  })
  .delete(function (req, res) {
    res.send("Delete the book");
  });
```

**Vorteile**: weniger fehleranfällig, leichter zu pflegen (da man die Route nur einmal schreibt)

### Modularisierung

Modularisierung von Routen in Express kann mithilfe von `express.Router` erreicht werden.

Erstellung einer Router-Datei `birds.js`:

```js
const express = require("express");
const router = express.Router();

// Middleware
router.use(function timeLog(req, res, next) {
  console.log("Time: ", Date.now());
  next();
});

// Routen
router.get("/", function (req, res) {
  res.send("Birds home page");
});

router.get("/about", function (req, res) {
  res.send("About birds");
});

module.exports = router;
```

Einbindung des Routers in die Anwendung `app.js`:

```js
const express = require("express");
const app = express();
const birds = require("./birds");

app.use("/birds", birds);

app.listen(3000);

console.log("Listening on :3000");
```

### Weitere Methoden von Express

**Result**:

- `res.status(code)`: Setzt den HTTP-Statuscode der Antwort (z.B. 200 für erfolgreiche Anfrage, 404 für nicht gefunden)
- `res.redirect(url)`: Leitet den Request an eine andere URL um
- `res.cookie(key, value, options)`: Setzt ein Cookie im Browser des Users, optionale Parameter können angegeben werden wie z.B. die Dauer des Cookies und ob es sicher übertragen werden soll
- `res.attachment(path_to_file)`: Sendet eine Datei als Attachment (z.B. Download)
- `res.download(path_to_file)`: Sendet eine Datei zum Download und zeigt eine entsprechende Benachrichtigung im Browser des Users

**Request**:

- `req.headers()`: Gibt ein Objekt mit allen HTTP-Request-Headern zurück
- `req.cookies()`: Gibt ein Objekt mit allen Cookies zurück, die im Request enthalten sind (benötigt die Middleware `cookie-parser`)

### Fehlerhandling

**404 als JSON zurückgeben**:

```js
app.use("/users", require("./routes/users"));
app.use("/products", require("./routes/products"));

// Middleware nach allen Routes
app.use((req, res) => {
  res.status(404);
  res.json({ message: "Not found" });
});
```

**Exceptions**:

- Wenn in einem Route-Handler eine Exception geworfen wird, sendet Express **standardmäßig eine HTML-Seite mit der Fehlermeldung und dem Stack-Trace** zurück.
- Das kann ein Sicherheitsproblem darstellen, da sensible Informationen preisgegeben werden können. Eine Lösung wäre, stattdessen eine vernünftige JSON-response zu senden:

```js
try {
  throw new Error("Something went wrong");
} catch (err) {
  res.status(500).json({ message: "InternalServerError" });
}
```

- Ein größeres Problem entsteht, wenn **eine Exception in einem Promise auftritt**, da es zu einem globalen Fehler UnhandledPromiseRejection im Node-Prozess kommt und keine Response gesendet wird.
- In zukünftigen Node-Versionen wird dieser Fehler nicht mehr global abgefangen und stattdessen der Prozess mit einem Fehlercode beendet, was zu einem **Absturz der gesamten Server-Anwendung** führen kann.
- **Lösung**: Route-Handler in try/catch packen, Exceptions der next-Funktion übergeben und eine eigene Error-Middleware einbauen.

```js
app.get("/", async (req, res, next) => {
  try {
    throw new Error("Something went wrong");
  } catch (err) {
    next(err);
  }
});

app.use((err, req, res, next) => {
  res.status(500);
  res.json({ message: "InternalServerError" });
  console.error(err);
});
```

### HTTP-Verben und HTML-Forms

- In Express kann man HTTP-Verben wie PATCH, PUT oder DELETE auf Endpoints mappen, die jedoch nur GET und POST verstehen
- Eine Lösung dafür ist die Verwendung einer speziellen Middleware wie method-override:

```js
const express = require("express");
const methodOverride = require("method-override");
const app = express();

app.use(methodOverride("_method"));
```

Jetzt kann man eine PATCH-Route definieren, die dann auch über ein Formular angesprochen werden kann:

```js
app.patch("/fruits", (req, res) => {
  // some code ...
});
```

In dem Formular muss dann der URL-Parameter `_method=patch` hinzugefügt werden:

```html
<form action="/fruits?_method=patch" method="post">...</form>
```

Jetzt wird die PATCH-Route aufgerufen, wenn das Formular abgeschickt wird.

## Die Template-Engine EJS und Express-Sessions

### Einführung in EJS

- EJS ist eine Template-Engine für JavaScript
- Ermöglicht die Generierung von HTML-Seiten oder Snippets im Web-Backend
- Express-Server verwendet vorhandene HTML-Templates, füllt diese mit Daten aus der Datenbank, und generiert damit fertiges HTML (ganze Seiten oder Snippets)

### Verwendung von EJS

Der Code auf dem **Server**, der die EJS-Template-Engine verwendet, sieht wie folgt aus:

```js
const express = require("express");
const app = express();

app.set("view engine", "ejs");

app.get("/user", (req, res) => {
  const user = {
    name: "John Doe",
    email: "johndoe@example.com",
    phone: "555-555-5555",
  };
  res.render("user-template", { user });
});
```

Das **Template** `template.ejs` im Unterverzeichnis `views`, welcher ein JavaScript-Objekt `{vorname, adresse, telefon}` übergeben wird:

```html
<html>
  <body>
    <h1>User Information</h1>
    <table>
      <tr>
        <td>Name:</td>
        <td><%= user.name %></td>
      </tr>
      <tr>
        <td>Email:</td>
        <td><%= user.email %></td>
      </tr>
      <tr>
        <td>Phone:</td>
        <td><%= user.phone %></td>
      </tr>
    </table>
  </body>
</html>
```

### Schleifen in EJS

**Server**:

```js
const express = require("express");
const app = express();

app.set("view engine", "ejs");

const DATA = [
  { id: 1, name: "Apfel", color: "gelb,rot" },
  { id: 2, name: "Birne", color: "gelb,grün" },
  { id: 3, name: "Banane", color: "gelb" },
];

app.get("/fruits", (req, res) => {
  res.render("all", { fruits: DATA }); // all.ejs Template
});

app.get("/fruits/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const fruit = DATA.find((o) => o.id === id);
  res.render("fruit", fruit); // fruit.ejs Template
});
app.listen(3000);

console.log("EJS server running on localhost:3000");
```

**Template**:

```html
<html>
  <body>
    <table>
      <tr>
        <th>Name</th>
        <th>Farbe</th>
      </tr>
      <% fruits.forEach( o => { %>
      <tr>
        <td><%= o.name %></td>
        <td><%= o.color %></td>
      </tr>
      <% }) %>
    </table>
  </body>
</html>
```

### State mit Cookies durch `cookie-parser`

- npm-Package `cookie-parser` ermöglicht zustandsbehaftete Server
- Cookies sind name-value-Paare, gesendet von Server, gespeichert im Browser
- Ermöglichen Identifizierung des Aufrufers bei zukünftigen Requests
- Beispiel: Verwaltung von Warenkorb eines Users auf e-Commerce-Website

So können **Cookies gesetzt** werden:

```js
const cookieParser = require("cookie-parser");

app.use(cookieParser());

response.cookie("userID", "xyz12345"); // Einzelner Cookie

response
  .cookie("userID", "xyz12345")
  .cookie("verein", "VfB Stuttgart", { maxAge: 90000 }); // Mehrere Cookies, der zweite mit 90000 milli secs Lebensdauer
```

So können **Cookies ausgelesen werden**:

```js
const cookieParser = require("cookie-parser");

app.use(cookieParser());

const cookies = request.cookies;

let userID = cookies.userID;
let verein = cookies.verein;
```

### State mit Cookies durch `express-session`

Mit dem npm-Package `express-session` kann man zustandsbehaftete Server bauen:

```js
const express = require("express");
const session = require("express-session");

const app = express();

app.use(
  session({
    secret: "mykey", // Für Encoding und Decoding des Cookies
    resave: false, // Nur speichern nach Änderung
    saveUninitialized: true, // Anfangs immer speichern
    cookie: { maxAge: 5000 }, // Ablaufzeit in Millisekunden
  })
);

app.get("/", function (req, res) {
  if (req.session.count) {
    // Eine Session kann beliebige Attribute bekommen
    req.session.count++;
    res.setHeader("Content-Type", "text/html");
    res.write("<p>count: " + req.session.count + "</p>");
    res.end();
  } else {
    req.session.count = 1;
    res.end("Willkommen zu der Sitzung. Refresh!");
  }
});
```
