# Datenbank-Management-Systeme

## Kurzüberblick

In dieser Vorlesung klären wir zunächst, warum wir überhaupt Datenbank-Management-Systeme (DBMS) benötigen. 
Anschließend betrachten wir die typische Server-Architektur von DBMS mit CLI-Tools und API-Servern. 
Darauffolgende diskutieren wir Abfragesprachen und Datenmodelle, sowie Gründe für Unterschiede zwischen SQL, ArangoQL, Influx und neo4j-Cypher. Nach einer kurzen Vorstellung der vier Systeme folgen praktische Query-Beispiele und zum Abschluss ein Live-Coding-Beispiel: Anbindung eines DBMS in Python mit FastAPI.

---

## 1 Definition von Datenbankmanagementsystemen

> Ein Datenbankmanagementsystem (DBMS) ist eine Software, die das Speichern, Verwalten und Abfragen von Daten in strukturierten oder unstrukturierten Formaten ermöglicht.  
> Es abstrahiert die physische Datenspeicherung und bietet Mechanismen für Datenintegrität, Transaktionen und Mehrbenutzerzugriffe.  
> Hierzu stellt es programmierbare Schnittstellen bereit, welche die Arbeit mit den entsprechenden Daten unterstützen.

Im Embedded-Bereich werden Daten oft direkt im Speicher als primitive Datentypen wie Integer oder Float kodiert, um Latenz und Overhead minimal zu halten. 
Zur effizienten Kommunikation zwischen Mikrocontroller und Peripherie kommen binäre Protokolle oder kompakte Formate wie Protocol Buffers zum Einsatz, um Bandbreite und Speicher zu optimieren.
Ein Unix-Dateisystem abstrahiert diese rohe Datenspeicherung, indem es Dateien als byte-sequenzielle Objekte mit Metadaten wie Berechtigungen und Zeitstempeln darstellt. 
Dabei verbirgt das OS die interne Struktur der Dateien und bietet Anwendungen eine einheitliche Schnittstelle für Dateioperationen. 
Mit komplexeren Zugriffsmustern stößt die Dateisystem-Abstraktion bei Transaktionen und gleichzeitigen Zugriffen an ihre Grenzen.
Umfangreichere Datenbank-Management-Systeme vertiefen die Abstraktion in einer Client-Server-Architektur, bei der ein Daemon permanent Anfragen entgegennimmt.
Relationale DBMS wie MariaDB und MySQL sind darauf optimiert, tabellenbasierte Informationen zu halten und relationskomplex gefiltert auszugeben.
InfluxDB als Spezialist für Zeitreihen speichert gesampelte Messdaten und exponiert mittels des Daemon-Prozesses influxd eine HTTP-API zur Datenerfassung und Analyse. 
Graphdatenbanken wie Neo4j organisieren Informationen als Knoten und Kanten und ermöglichen mit Cypher und einer REST-API komplexe Pfadabfragen über einen laufenden Server-Dienst.
Multi-Model-Systeme wie ArangoDB kombinieren Dokument-, Graph- und Key-Value-Modelle und stellen über eine RESTful HTTP-API namens AQL umfangreiche Abfragemöglichkeiten bereit.
Alle diese DBMS-Daemons abstrahieren die zugrunde liegende Storage-Schicht und bieten über Netzwerkprotokolle standardisierte APIs für konsistente CRUD-Operationen. 
Dieser nahtlose Übergang von direkter Kodierung über Filesystem-Abstraktion bis hin zu spezialisierten API-Daemons ermöglicht skalierbare und zuverlässige Datensysteme in modernen Anwendungen.

Allen dabei gemein ist jedoch stets, dass ein über den Daten stehendes Modell Metainformationen über das Gespeicherte halten muss und damit ein Interpretationsmuster für die Daten bietet. Es ist dieses persistente Interpretationsmuster und die dazugehörigen Ablage- und Abfragemechanismen, welche die Unterschiede zwischen DBMS ausmachen.


Daten werden in einfachen Speichersystemen primär als Bitfolgen persistiert und bei Bedarf wieder abgerufen, ohne dass Metadaten, Konsistenzprüfungen oder Zugriffskontrolle eingebaut sind. Komplexe Datenverwaltung hingegen umfasst den gesamten Lebenszyklus von Daten – inklusive Validierung, Organisation, Zugriffssicherheit, Archivierung und Governance –, um ihre Zuverlässigkeit und Wiederverwendbarkeit sicherzustellen.

### Definitionen  

#### Reine Datenspeicherung  
Die reine Datenspeicherung bezeichnet das einfache Ablegen von Informationen in einem Speichermedium wie Festplatten, SSDs oder optischen Datenträgern, wobei es nur um die physische Persistenz und spätere Abrufbarkeit geht. Es finden keine oder nur minimale Zusatzdienste statt: Weder Transaktionsmanagement noch komplexe Metadatenverwaltung oder automatische Konsistenzsicherungen sind vorgesehen.

#### Komplexe Datenverwaltung  
Datenverwaltung (Data Management) umfasst alle Prozesse zur standardisierten Erstellung, Sammlung, Speicherung, Pflege, Sicherung und letztlich Löschung von Daten im Sinne ihrer optimalen Nutzung. Sie beinhaltet Richtlinien und Werkzeuge für Datenqualität, Datenschutz, Lebenszyklusmanagement und Compliance, um Daten als strategische Ressource zu behandeln.

### Unterschiede im Detail  

#### Fokus und Ziele  
- **Speicherung:** Primär Persistenz und Wiederherstellung der Rohdaten ohne zusätzliche Intelligenz.  
- **Verwaltung:** Sicherstellung von Datenintegrität, Konsistenz, Zugriffskontrolle und Nachvollziehbarkeit während des gesamten Datenlebenszyklus.

#### Funktionalitäten  
- **Reine Speicherung** bietet einfache CRUD-Operationen auf Byte-Arrays und Verzeichnisebene.  
- **Datenverwaltung** liefert erweitertes Transaktionsmanagement, Indexierung, Metadatenservices, Backup/Restore, Replikation und Data Governance.

#### Technologien und Beispiele  
- **Data Stores & Dateisysteme:** z. B. ext4, NTFS, S3-Buckets, Key-Value-Stores wie Redis, die einfache Persistenz ermöglichen.  
- **DBMS/DM-Tools:** Relationale Datenbanken (MySQL), Multi-Model-Systeme (ArangoDB), Zeitreihen-Datenbanken (InfluxDB), Graphdatenbanken (Neo4j) – sie setzen Daemons mit CLI-Tools und API-Servern ein, um umfassende Managementfunktionen bereitzustellen.

Durch diese Betrachtung wird ersichtlich, dass reine Datenspeicherung und komplexe Datenverwaltung unterschiedliche Verantwortungsbereiche abdecken: Die Speicherung liefert das Fundament, während die Verwaltung dafür sorgt, dass Daten konsistent, sicher und nutzbar bleiben.


### Funktionalitäten eines gehobenen DBMS

#### Konsistenz
Ein gehobenes DBMS sorgt dafür, dass Daten nach definierten Regeln immer in einem gültigen Zustand vorliegen. Änderungen werden nur übernommen, wenn sie sämtliche Integritätsbedingungen (z. B. Primär- und Fremdschlüssel, Check-Constraints) erfüllen. Dadurch wird verhindert, dass widersprüchliche oder fehlerhafte Daten in die Datenbank gelangen.

#### Transaktionen
Transaktionen bündeln eine Folge von Datenbankoperationen zu einer atomaren Einheit. Erst wenn alle Teiloperationen erfolgreich abgeschlossen sind, wird die Transaktion “committed” und die Änderungen werden dauerhaft. Schlägt eine Teiloperation fehl, kann die Transaktion zurückgesetzt werden und die Datenbank bleibt im ursprünglichen Zustand.

#### Mehrbenutzerzugriff
Ein leistungsfähiges DBMS erlaubt simultane Zugriffe zahlreicher Clients, ohne dass es zu Inkonsistenzen kommt. Über Locking-Mechanismen oder Multiversion Concurrency Control (MVCC) werden Lese- und Schreibkonflikte isoliert behandelt, sodass jeder Benutzer eine konsistente Sicht auf die Daten erhält.

#### Rollback
Rollback bezeichnet die Fähigkeit, eine laufende oder bereits abgeschlossene Transaktion bei Fehlern oder Inkonsistenzen vollständig zurückzunehmen. Alle im Rahmen der Transaktion vorgenommenen Änderungen werden rückgängig gemacht, um die Integrität der Datenbank sicherzustellen.

#### ACID, CRUD und weitere Paradigmen
- **ACID**: Atomicity, Consistency, Isolation, Durability – Garantien, die zusammen hohe Zuverlässigkeit und Stabilität bieten.  
- **CRUD**: Create, Read, Update, Delete – die grundlegenden Operationen zur Verwaltung von Datenobjekten.  
- **Weitere Paradigmen**: Je nach DBMS kommen zusätzliche Konzepte wie Stored Procedures, Trigger, Sharding, Replikation oder eventuelle Konsistenz (bei verteilten Systemen) zum Einsatz, um Performance, Skalierbarkeit und Ausfallsicherheit zu optimieren.

---

## Organisation von DBMS  
Dateisysteme organisieren Daten auf Blockebene in Inodes und Verzeichnisbäumen, wobei Metadaten wie Zugriffsrechte und Zeitstempel in Superblöcken und Inodes verwaltet werden.
SQL-Datenbanken abstrahieren Speicher mit Seiten in Heaps und B-Baum-Indizes und speichern Schema- und Objektinformationen in Systemkatalogen.  
InfluxDB nutzt eine Time-Structured Merge Tree (TSM) Engine mit WAL, Cache und kompakten TSM-Dateien pro Shard, um Zeitreihendaten effizient zu erfassen und abzufragen.  
Neo4j realisiert ein native Property-Graph-Modell mit festgrößigen Datensatz-Dateien für Knoten, Kanten und Eigenschaften, die durch direkte Zeiger verknüpft sind.  
ArangoDB speichert verschachtelte JSON-Dokumente in VelocyPack-Form und nutzt RocksDB (LSM-Baum) als zugrunde liegende Schlüssel-Wert-Engine; Metadaten liegen in Systemattributen wie `_key`, `_id` und `_rev`.

### Dateisysteme  
Unix-Dateisysteme halten je Datei oder Verzeichnis einen Inode mit Metadaten (Größe, Rechte, Zeitstempel) und Block­zeigern für Datenblöcke vor.  
Verzeichnisse sind einfache Dateien mit Einträgen `<Name, Inode>` und bilden so einen hierarchischen Baum, dessen Wurzel das Root-Verzeichnis ist.  
Integritätsregeln wie Journaling und Prüfsummen (z. B. in ext4) schützen Metadaten und Daten vor Inkonsistenzen nach Abstürzen.

### SQL-Datenbanken  
Daten liegen in fixed-Size Pages (z. B. 8 KB) vor, die in Heaps ohne Index oder in B-Baum-Strukturen mit Clustered/Non-Clustered-Indizes organisiert sind.  
Die Hierarchie reicht vom Server über Datenbanken und Schemata zu Tabellen, Seiten, Datensätzen und Attributen.  
Systemkataloge verwalten Metadaten zu Tabellen, Spalten, Indizes und Benutzern, während Fremdschlüssel und Check-Constraints Beziehungen und Integrität erzwingen.

### InfluxDB  
InfluxDB gliedert Daten nach Datenbank → Retention Policy → Shard → TSM-Dateien, wobei jede TSM-Datei komprimierte, zeitorientierte Spaltenblöcke enthält.  
Ein In-Memory-Index und WAL ermöglichen schnelle Schreibzugriffe; der Metastore-Ordner hält Benutzer-, DB- und Policy-Metadaten.  
Beziehungen zwischen Messreihen entstehen über Tag-Sets und Field-Sets, Integrität wird durch WAL-Flush und atomare Datei-Operationen sichergestellt.

### Neo4j  
Neo4j speichert Knoten und Beziehungen in separaten Store-Dateien: fixe Record-Größen mit Zeigern auf Property-Record-Ketten und Relationship-Chains.  
Die Hierarchie bildet ein Satz von Store-Dateien (Nodes, Relationships, Properties, Labels) statt klassischer Ordner; Labels und Indizes dienen der Metadatenverwaltung.  
Integritätsregeln und ACID-Eigenschaften werden durch das transaktionale Logging und Lock-Mechanismen des Daemons durchgesetzt.

### ArangoDB  
ArangoDB verwendet JSON-Dokumente (VelocyPack) in Collections, die zu Shards zusammengefasst in RocksDB-LSM-Strukturen abgelegt werden.  
Dokument-Systemattribute (`_key`, `_id`, `_rev`, bei Edge-Dokumenten `_from`, `_to`) speichern Metadaten und definieren Kanten im Graphenmodell.  
Fremdschlüssel-ähnliche Beziehungen entstehen durch Edge-Collections, Integrität und ACID-Transaktionen auf Einzelservern garantiert die Engine.  

---

## Module-Architektur der fünf Systeme

### 1. Dateisystem (z. B. ext4)
- **VFS-Layer**: Abstraktionsschicht im Kernel, die einheitliche API für verschiedene Dateisysteme bietet  
- **FS-Treiber**: ext4-spezifische Implementierung für Inode- und Blockverwaltung  
- **Journal-Modul**: Journaling zur Crash-Sicherheit und schnellen Wiederherstellung  
- **Verzeichnisverwaltung**: Verwaltung von Einträgen `<Name, Inode>` in Verzeichnisblöcken  
- **Inode-Manager**: Verwaltung von Metadaten (Größe, Rechte, Zeitstempel) in Inodes  
- **Cache-Manager**: Page- und Buffer-Cache im Kernel für performantes Lesen/Schreiben  

### 2. Relationale SQL-Datenbank (z. B. PostgreSQL)
- **Postmaster**: Verbindungskoordinator, spawnt Backend-Prozesse für jede Client-Session  
- **Parser & Rewriter**: Übersetzung von SQL-Strings in interne Repräsentationen  
- **Query-Optimizer & Planner**: Kostenbasierte Planerstellung für effiziente Ausführung  
- **Executor**: Abarbeitung des Ausführungsplans und Datentransformation  
- **Storage Manager**: Pufferverwaltung, Shared Buffers, WAL-Manager  
- **Systemkataloge**: Metadaten zu Tabellen, Indizes, Nutzern  
- **Hintergrundprozesse**: Checkpointer, Autovacuum, Stats Collector  

### 3. Zeitreihen-Datenbank (InfluxDB)
- **influxd-Daemon**: Zentrale Serverinstanz  
- **HTTP/API-Modul**: Exponiert Line Protocol, InfluxQL/Flux über REST  
- **Write-Ahead Log (WAL)**: Puffer für Schreibvorgänge  
- **TSM-Engine**: Zeitstrukturierte Merge-Tree-Dateien für langfristige Speicherung  
- **In-Memory-Cache**: Schnelle Lesezugriffe auf aktuelle Daten  
- **Retention Policy Manager**: automatisches Löschen alter Zeitreihen  
- **Task-Engine**: Ausführung kontinuierlicher Abfragen  

### 4. Graphdatenbank (Neo4j)
- **neo4j-server**: Hauptdaemon, der Bolt- und HTTP-Schnittstellen bereitstellt  
- **Kernel Store**: Native Property-Graph-Speicherung in Nodes-, Relationships- und Property-Files  
- **Page Cache**: Memory-Mapping der Store-Dateien für schnellen Zugriff  
- **Cypher-Engine**: Parser, Optimizer, Runtime für Cypher-Abfragen  
- **Index-Provider**: native B-Baum- und Lucene-Indizes  
- **Transaktions-Log**: Write-Ahead-Log für ACID-Sicherheit  
- **Causal Clustering**: Core-Server, Read Replicas, Discovery-Service  

### 5. Multi-Model-DB (ArangoDB)
- **arangod-Daemon**: zentraler Server-Prozess für Einzelserver und Cluster  
- **Storage Engine**: RocksDB (LSM-Baum) oder MMFiles  
- **AQL-Query-Engine**: Parser, Optimizer, Executor für die Abfragesprache  
- **Foxx Microservice-Engine**: Integrierte JavaScript-Laufzeit  
- **Agency**: RAFT-basierter Konfigurationsmanager im Cluster  
- **Coordinator/DBServer**: Koordination und Datensharding im Cluster  
- **Index-Manager**: Handhabung verschiedener Index-Typen (_key_, _hash_, _skiplist_, Geo, Fulltext)  
- **HTTP/API-Server**: RESTful Exposition aller Funktionen  

## Rolle von „Server“ und Client-Server-Modell
In allen fünf Systemen bezeichnet „Server“ den Daemon-Prozess, der dauerhaft im Hintergrund läuft, auf Netzwerk- oder Socket-Verbindungen horcht und Anfragen entgegennimmt. Das **Client-Server-Modell** trennt die **Serverinstanz** (Datenhaltung und -verarbeitung) von den **Clients**, die über Netzwerkprotokolle (TCP/IP) kommunizieren. Diese Architektur erlaubt skalierbare Mehrbenutzerzugriffe und klare Verantwortungsgrenzen.

## API
APIs stellen eine standardisierte Schnittstelle zur Verfügung, über die Programme Daten lesen, schreiben und verwalten können.  
- **REST**: Ressourcenorientiertes HTTP-Interface mit Endpunkten für CRUD-Operationen  
- **gRPC**: Binäres Protokoll auf HTTP/2-Basis für performante, typisierte Aufrufe  
APIs abstrahieren das DBMS-intern, sodass Treiber und Microservices unabhängig vom zugrunde liegenden Speichersystem arbeiten.

## Kommandozeilen-Clients
CLI-Tools (z. B. `psql`, `mysql`, `influx`, `cypher-shell`, `arangosh`):  
- Stellen interaktive und skriptfähige Oberflächen bereit  
- Rufen über Netzwerk-Sockets dieselben Server-APIs auf  
- Dienen Administration, Debugging und Datenmigration  
- Erlauben Ad-hoc-Abfragen, Batch-Jobs und Automatisierung

---

## Konzepte von DDL und DML
 
DDL und DML sind fundamentale Abstraktionen in jedem Datenmanagementsystem:  
- **DDL (Data Definition Language)** beschreibt formal den Aufbau und die Anpassung von Datenstrukturen und -schemata.  
- **DML (Data Manipulation Language)** regelt das Einfügen, Ändern, Löschen und Abfragen der darin enthaltenen Daten.  
Beide Konzepte sind unabhängig vom konkreten Datenmodell und finden sich in relationalen, dokumentenorientierten, zeitserienbasierten und graphbasierten Systemen wieder. Sie ermöglichen durch die Trennung von Struktur- und Inhaltsoperationen eine hohe Flexibilität, Integrität und Skalierbarkeit im Umgang mit Daten.


### Data Definition Language (DDL)  
DDL dient der **Definition** und **Verwaltung** formaler Datenstrukturen:

- **Abstrakte Beschreibungen**  
  Deklarative Befehle zum Erstellen, Ändern oder Löschen von Datenobjekten wie Tabellen, Schemata, Kollektionen oder Graphentypen.  
- **Metadatenverwaltung**  
  Jeder DDL-Befehl aktualisiert einen zentralen Metadatenspeicher (Systemkatalog), der Informationen über Datentypen, Beziehungen, Zugriffsrechte und Validierungsregeln enthält.  
- **Integritätsregeln**  
  Definition von Constraints (z. B. Primär-/Fremdschlüssel, Unique- oder Check-Constraints), die automatisch überwacht und durchgesetzt werden.  
- **Versionierung & Migration**  
  Unterstützung von Schema-Änderungen über versionierte Skripte, Migrations-Tools und Rollback-Mechanismen.

### Data Manipulation Language (DML)  
DML umfasst alle **Operationen auf den Inhalten** definierter Strukturen:

- **Datenzugriff & -änderung**  
  Basisoperationen wie Insert, Update, Delete und – häufig mitgezählt – Select, um Datensätze anzulegen, zu modifizieren oder zu entfernen.  
- **Transaktionale Steuerung**  
  Bündelung von DML-Befehlen in Transaktionen für atomare Ausführung, Konsistenz und Rollback-Fähigkeit.  
- **Performance-Optimierungen**  
  Einsatz von Indizes, Batching, Query Caching und Prepared Statements zur Reduktion von Latenz und Ressourcennutzung.  
- **Datenabfragen**  
  Kombination von Schreib- und Leseoperationen, wobei Select-Abfragen (teilweise als DQL betrachtet) integraler Bestandteil sind.



### 1. Dateisystem
#### DDL
- Struktur definieren: Befehle wie `mkdir`, `rmdir` und `ln` legen Verzeichnisse, Unterverzeichnisse und Links an oder entfernen sie.  
- Dateien anlegen: `touch` initialisiert neue Dateien ohne Inhalt.  

#### DML
- Daten lesen und schreiben: `cat`, `echo 'Text' > datei.txt` oder Umleitungen (`>`, `>>`).  
- Daten verändern: `cp`, `mv`, `rm` kopieren, verschieben oder löschen bestehender Dateien.  

---

### 2. Relationale SQL-Datenbank
#### DDL
- Objektdefinition: `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `CREATE INDEX`, `DROP INDEX` deklarieren Schema-Objekte.  
- Schemaverwaltung: `CREATE SCHEMA`, `GRANT`, `REVOKE` verwalten Schemata und Zugriffsrechte.  

#### DML
- Datenabfrage: `SELECT ... FROM ... WHERE ...` liest Datensätze aus.  
- Datenmanipulation: `INSERT`, `UPDATE`, `DELETE` fügen Datensätze hinzu, ändern oder entfernen sie.  
- Transaktionen: `BEGIN`, `COMMIT`, `ROLLBACK` bündeln Operationen atomar.  

---

### 3. Zeitreihen-Datenbank (InfluxDB)
#### DDL
- Speicherbereiche und Richtlinien: `CREATE DATABASE`, `DROP DATABASE`, `CREATE RETENTION POLICY`, `DROP RETENTION POLICY` legen Datenbanken und Aufbewahrungsregeln fest.  
- Kontinuierliche Abfragen: `CREATE CONTINUOUS QUERY` definiert automatische Langzeit-Auswertungen.  

#### DML
- Schreibzugriff: Line Protocol via CLI oder HTTP (`INSERT ...`) fügt Messpunkte ein.  
- Abfragen: `SELECT ... FROM ... WHERE time > ...` filtert und aggregiert Zeitreihen.  
- Löschung: `DELETE FROM` entfernt Datenpunkte in einem Zeitbereich.  

---

### 4. Graphdatenbank (Neo4j)
#### DDL
- Schema-Elemente: `CREATE CONSTRAINT ON (n:Label) ASSERT n.prop IS UNIQUE` und `CREATE INDEX FOR (n:Label) ON (n.prop)` definieren Constraints und Indizes.  

#### DML
- Knotenerstellung: `CREATE (n:Label {props})` fügt Knoten mit Eigenschaften hinzu.  
- Beziehungserstellung: `CREATE (a)-[:REL]->(b)` legt Kanten zwischen Knoten an.  
- Abfrage/Lesezugriff: `MATCH (n)-[r]->(m) RETURN n, r, m` liest Strukturen aus.  
- Änderung/Löschung: `SET n.prop = value`, `DELETE r` ändern oder entfernen Elemente.  

---

### 5. Multi-Model-DB (ArangoDB)
#### DDL
- Collections und Graphen: Über REST-API (`POST /_api/collection`) oder JavaScript-API (`db._createDocumentCollection("col")`, `db._createEdgeCollection("edges")`).  
- Indexe und Views: `POST /_api/index` bzw. `db._createView("viewName", "arangosearch", config)` definieren zusätzliche Strukturen.  

#### DML
- AQL-Befehle:  
  - `INSERT { ... } INTO col`  
  - `UPDATE { ... } IN col`  
  - `REMOVE doc IN col`  
  - `UPSERT { ... } INSERT { ... } UPDATE { ... } IN col`  
- Abfrage: `FOR doc IN col FILTER doc.prop == value RETURN doc`.  

---

### Best Practices  
1. **Trennung von DDL und DML**: Schema-Änderungen getrennt von Datenänderungen durchführen, um Übersicht und Sicherheit zu erhöhen.  
2. **Explizite Transaktionsgrenzen**: Transaktionen klar abgrenzen, da viele DDL-Operationen auto-committed sind.  
3. **Versionskontrolle**: DDL-Skripte in VCS integrieren und Migrations-Tools nutzen für automatisierte Deployments.  
4. **Monitoring & Tuning**: DML-Workloads durch Query-Plan-Analyse, Lock-Wait-Tracking und Index-Überwachung optimieren.


### Unterschiedliche Paradigmen

Moderne Datenbank-Management-Systeme bieten verschiedene Paradigmen, weil unterschiedliche Einsatzszenarien und Entwicklerpräferenzen spezifische Stärken erfordern. Eine Abfragesprache muss sowohl klar und prägnant beschreiben können, was getan werden soll, als auch eine effiziente Ausführung durch das System ermöglichen. Durch die Trennung in deklarative und imperative Ansätze sowie in schema-basierte und schemalose Modelle entsteht eine Bandbreite an Gestaltungsmöglichkeiten, aus der jede Anwendung das passende Werkzeug wählen kann.

#### Deklarativ vs. Imperativ

Im **deklarativen Paradigma** beschreibt der Anwender das gewünschte Ergebnis ohne Festlegung, wie es erreicht wird. Beispiele sind SQL oder Cypher, in denen man formuliert, welche Daten man haben möchte, und der Datenbank-Optimizer selbst einen optimalen Ausführungsplan erzeugt. Dieser Ansatz erleichtert es, komplexe Abfragen kurz und lesbar zu formulieren und erlaubt der Engine, unabhängig von der Nutzerlogik automatische Optimierungen vorzunehmen.

Im **imperativen Paradigma** gibt der Nutzer detaillierte Anweisungen zur Abarbeitung, etwa Schritt für Schritt durch eine gespeicherte Prozedur oder ein Skript. Hier steuert man explizit Lese- und Schreibschritte, Cursor-Bewegungen oder Schleifen. Dieser Stil bietet maximale Kontrolle über Ablauf und Performance, erfordert aber mehr Detailarbeit seitens des Entwicklers und birgt ein höheres Fehlerrisiko bei Änderungen.

#### Schema vs. Schemalos

**Schema-basierte Systeme** definieren im Vorfeld klare Strukturen: Tabellen mit Spalten, Datentypen und Constraints. Diese Festlegung erlaubt strikte Validierung, optimierte Speicherung und Indexierung. Typische Vertreter sind relationale Datenbanken wie MySQL oder PostgreSQL. Sie sorgen für hohe Datenintegrität und Konsistenz, eignen sich aber weniger gut für sehr heterogene oder sich dynamisch ändernde Datensätze.

**Schemalose Systeme** verzichten auf starre Vorgaben und erlauben das Speichern beliebig strukturierter Dokumente oder Key-Value-Paare. Document Stores wie MongoDB oder Multi-Model-Systeme wie ArangoDB bieten hier maximale Flexibilität und eine schnelle Anpassung an neue Anforderungen. Allerdings liegt die Verantwortung für Datenkonsistenz und -validierung oft stärker beim Entwickler oder einer zusätzlichen Anwendungsschicht.

### Vielfältigkeit der Abfragesprachen

Es existieren so viele Abfragesprachen, weil keine universelle Lösung alle Performance-, Konsistenz- und Modellierungsanforderungen gleichermaßen optimal bedienen kann. Zeitreihen-Datenbanken wie InfluxDB nutzen eine auf Metriken und Zeitfenster spezialisierte Sprache, Graph-Datenbanken setzen auf Pfad- und Beziehungsabfragen, und dokumentorientierte Systeme bevorzugen JSON-basierte Filter- und Aggregationsbefehle. Jede dieser Sprachen ist auf das jeweilige Datenmodell zugeschnitten und stellt Operatoren, Optimierungen und Datentypen bereit, die im anderen Kontext nur schwer oder ineffizient abbildbar wären. Durch diese Spezialisierung lassen sich komplexe Analysen und Operationen mit minimalem Aufwand und maximaler Performance realisieren.

In der Summe bieten unterschiedliche Paradigmen und Sprachen ein reichhaltiges Werkzeugset, mit dem man je nach Anforderung von strikter Konsistenz über flexible Entwicklung bis hin zu hochperformanter Analyse exakt das passende DBMS-Interface wählen kann.

---

## 4. Vorstellung praktischer Beispiele der ausgewählten DBMS

Im Folgenden werden konkrete Einsatzszenarien für die fünf besprochenen Systeme skizziert, um ihre jeweilige Stärke im Produktiveinsatz zu verdeutlichen.

### Dateisystem (ext4)
Das ext4-Dateisystem ist in nahezu allen Linux-Distributionen die Standard-Wahl für Root- und Datenpartitionen, da es hohe Stabilität und gute Performance bei großen Datenmengen bietet. Dank Journaling übersteht ext4 unerwartete Systemabstürze ohne Datenverlust, weshalb es in Unternehmensservern und Cloud-VMs für persistente Speichervolumes eingesetzt wird. Auch bei Hochleistungsrechnern (HPC) dienen ext4-Partitionen als schneller Scratch-Speicher für rechenintensive wissenschaftliche Workloads.

### Relationale SQL-Datenbank (PostgreSQL)
PostgreSQL findet breite Anwendung in Data-Warehouse-Umgebungen, wo es komplexe Analysen und OLAP-Workloads bewältigt, etwa in Business-Intelligence-Plattformen großer Konzerne. Web-Anwendungen wie Instagram nutzen PostgreSQL als primäre Datenbank, um Milliarden von Nutzerprofilen und Mediendateien performant zu verwalten. FlightAware setzt PostgreSQL für die Verarbeitung und Verteilung von Echtzeit-Flugverkehrsdaten ein, um Nutzer weltweit präzise Tracking-Informationen in Echtzeit zu liefern.

### Zeitreihen-Datenbank (InfluxDB)
InfluxDB wird intensiv im DevOps- und Monitoring-Bereich eingesetzt, um Metriken aus Servern, Containern und Anwendungen zu sammeln und in Dashboards darzustellen. In Industrie-IoT-Projekten speist InfluxDB Sensordaten von Fertigungsstraßen in Echtzeit, um Produktionsausfälle frühzeitig zu erkennen und Wartungszyklen zu optimieren. Mobile und Web-APIs nutzen InfluxDB, um Zeitstempel-basierte Ereignisse wie Nutzerinteraktionen oder Netzwerk-Logs performant abzulegen und abzufragen.

### Graphdatenbank (Neo4j)
Neo4j wird häufig für Fraud-Detection-Systeme in Finanzinstituten verwendet, da es komplexe Netzwerke von Transaktionen und Beziehungen in Echtzeit analysieren kann. Telekom- und Netzwerkanbieter setzen Neo4j für Topologie-Management und Routing-Optimierung ein, um Netzausfälle schnell zu lokalisieren und zu beheben. Im Bereich Wissensmanagement modellieren Unternehmen soziale Netzwerke, Stammbaum-Daten und Infrastruktur-Beziehungen als Graphen, um hochgradig vernetzte Daten intuitiv zu durchsuchen.

### Multi-Model-DB (ArangoDB)
ArangoDB kommt in Recommendation Engines zum Einsatz, indem es Produktkataloge als Dokumente speichert und Nutzerverhalten in Graphstrukturen abbildet, um personalisierte Vorschläge zu generieren. Fluggesellschaften wie FlightStats nutzen ArangoDB, um Referenzdaten zu Flughäfen, Airlines und Fluggeräten konsolidiert abzulegen und zeitlich versioniert bereitzustellen. In Identity-&-Access-Management-Systemen erleichtert ArangoDB das Verwalten von Rollen, Berechtigungen und hierarchischen Organisationsstrukturen dank seiner flexiblen Multi-Model-Fähigkeiten.

---

## 5. praktische Query-Beispiele

In diesem Kapitel zeigen wir für jedes der fünf besprochenen Systeme ein kurzes, praktisches Query-Beispiel. Dabei gehen wir auf Syntaxunterschiede und den Data Flow zwischen Client und Server (bzw. Shell) ein.

---

### 5.1 Dateisystem (ext4)

**Beispiel**: Suche nach Dateien, die mehr als 1 MB groß sind, und zähle die Zeilen in jeder Datei.

```bash
find /var/log -type f -size +1M -print0 \
  | xargs -0 wc -l
```

* **Syntax**: Imperativ, kombiniert Unix-Tools über Pipes und Optionen.
* **Data Flow**: `find` traversiert Inode-Baum im VFS; Pfade werden per Null-Separator an `wc` übergeben; `wc` liest jede Datei direkt vom Block-Device.

---

### 5.2 Relationale SQL-Datenbank (PostgreSQL)

**Beispiel**: Auslesen der Kundendaten mit Bestellungen im letzten Monat.

```sql
SELECT c.id, c.name, COUNT(o.id) AS bestellungen
FROM kunden c
JOIN bestellungen o
  ON o.kunden_id = c.id
WHERE o.bestelldatum >= now() - interval '1 month'
GROUP BY c.id, c.name
ORDER BY bestellungen DESC;
```

* **Syntax**: Deklarativ, nutzt JOINs, GROUP BY, und Funktionen (z.B. `now()`).
* **Data Flow**: Client (z.B. `psql` oder Anwendung) sendet SQL-String via TCP/IP an den Postgres-Server. Server parst, optimiert und führt den Plan aus, Ergebnisse werden zurückgesendet.

---

### 5.3 Zeitreihen-Datenbank (InfluxDB)

**Beispiel** (InfluxQL): Durchschnittliche CPU-Load pro Minute der letzten Stunde.

```sql
SELECT MEAN("load1") AS avg_load
FROM "cpu_metrics"
WHERE time >= now() - 1h
GROUP BY time(1m)
```

```flux
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu_metrics" and r._field == "load1")
  |> aggregateWindow(every: 1m, fn: mean)
```

* **Syntaxunterschiede**: InfluxQL ist SQL-ähnlich, Flux ist funktional-pipeline-basiert.
* **Data Flow**: Client schickt HTTP-Request an `influxd`-Daemon; Server liest TSM-Dateien, aggregiert Daten und gibt JSON oder CSV zurück.

---

### 5.4 Graphdatenbank (Neo4j)

**Beispiel**: Finde Pfade zweiter Ordnung zwischen zwei User-Knoten.

```cypher
MATCH (a:User {id: 1})-[:FOLLOWS*2]-(b:User)
RETURN b.id, b.name;
```

* **Syntax**: Graphmuster mit `MATCH` und Pfadlängenoperator `*2`.
* **Data Flow**: Cypher-Query wird per Bolt- oder HTTP-Protokoll zum Neo4j-Daemon gesendet. Kernel Store liest Node- und Relationship-Files, führt Pattern-Matching aus und liefert die Tupel zurück.

---

### 5.5 Multi-Model-DB (ArangoDB)

**Beispiel**: Finde alle Produkte, die von einem bestimmten Lieferanten stammen, und deren Bewertungen über 4 liegen.

```aql
FOR p IN products
  FILTER p.supplierId == @supplierId
  FOR r IN reviews
    FILTER r.productId == p._key AND r.rating > 4
  RETURN {
    produkt: p.name,
    durchschnitt: r.rating
  }
```

* **Syntax**: AQL vereint dokumentorientierte `FOR`-Schleifen mit Filter-Logik und verschachtelten Queries.
* **Data Flow**: Client sendet AQL über HTTP/REST an den `arangod`-Daemon. Query-Engine parst und optimiert, liest VelocyPack-Dokumente aus RocksDB oder MMFiles, kombiniert Ergebnisse und liefert JSON.

---

## 6. Integration in Python

### 6.1 Dateisystem (ext4)

```python
import os

def list_large_files_and_line_counts(root):
    for dirpath, _, filenames in os.walk(root):
        for fname in filenames:
            path = os.path.join(dirpath, fname)
            try:
                size = os.path.getsize(path)
                if size > 1_000_000:  # >1MB
                    with open(path, 'r', errors='ignore') as f:
                        line_count = sum(1 for _ in f)
                    print(f"{path}: {line_count} lines")
            except (OSError, UnicodeDecodeError):
                continue

if __name__ == "__main__":
    list_large_files_and_line_counts("/var/log")
```

* **Beschreibung**: Traversiert Verzeichnisbaum, filtert Dateien >1 MB und zählt Zeilen.
* **Data Flow**: Python-Prozess liest direkt das Dateisystem über `os`-Calls.

---

### 6.2 PostgreSQL (psycopg2)

```python
import psycopg2

 def query_customers_last_month():
    conn = psycopg2.connect(
        host="localhost", dbname="mydb", user="user", password="pass"
    )
    with conn:
        with conn.cursor() as cur:
            cur.execute("""
                SELECT c.id, c.name, COUNT(o.id) AS orders
                FROM customers c
                JOIN orders o ON o.customer_id = c.id
                WHERE o.order_date >= NOW() - INTERVAL '1 month'
                GROUP BY c.id, c.name
                ORDER BY orders DESC;
            """)
            for row in cur.fetchall():
                print(row)

if __name__ == "__main__":
    query_customers_last_month()
```

* **Beschreibung**: Führt aggregierte Abfrage der letzten Bestellungen aus.
* **Data Flow**: SQL-String wird über TCP/IP an den Postgres-Server gesendet.

---

### 6.3 InfluxDB (influxdb-client)

```python
from influxdb_client import InfluxDBClient

 def query_avg_cpu_load():
    client = InfluxDBClient(
        url="http://localhost:8086", token="my-token", org="my-org"
    )
    query = '''
    from(bucket: "metrics")
      |> range(start: -1h)
      |> filter(fn: (r) => r._measurement == "cpu_metrics" and r._field == "load1")
      |> aggregateWindow(every: 1m, fn: mean)
    '''
    result = client.query_api().query(query)
    for table in result:
        for record in table.records:
            print(f"{record.get_time()}: {record.get_value()}")

if __name__ == "__main__":
    query_avg_cpu_load()
```

* **Beschreibung**: Aggregiert CPU-Load der letzten Stunde.
* **Data Flow**: HTTP-Request an den `influxd`-Daemon, Ergebnis als Records.

---

### 6.4 Neo4j (neo4j-driver)

```python
from neo4j import GraphDatabase

 def find_second_degree_friends(uri, user, password, user_id):
    driver = GraphDatabase.driver(uri, auth=(user, password))
    query = """
    MATCH (a:User {id: $uid})-[:FOLLOWS*2]-(b:User)
    RETURN b.id AS id, b.name AS name
    """
    with driver.session() as session:
        result = session.run(query, uid=user_id)
        for record in result:
            print(record["id"], record["name"])
    driver.close()

if __name__ == "__main__":
    find_second_degree_friends("bolt://localhost:7687", "neo4j", "pass", 1)
```

* **Beschreibung**: Sucht User zweiter Ordnung.
* **Data Flow**: Bolt-Protokoll zum Neo4j-Daemon, Pattern-Matching on Server.

---

### 6.5 ArangoDB (python-arango)

```python
from arango import ArangoClient

 def find_highly_rated_products(supplier_id):
    client = ArangoClient(hosts="http://localhost:8529")
    db = client.db("mydb", username="user", password="pass")
    query = """
    FOR p IN products
      FILTER p.supplierId == @supplierId
      FOR r IN reviews
        FILTER r.productId == p._key AND r.rating > 4
      RETURN { product: p.name, rating: r.rating }
    """
    bind_vars = {"supplierId": supplier_id}
    cursor = db.aql.execute(query, bind_vars=bind_vars)
    for doc in cursor:
        print(doc)

if __name__ == "__main__":
    find_highly_rated_products("supplier123")
```

* **Beschreibung**: Findet Produkte mit hoher Bewertung.
* **Data Flow**: AQL über HTTP/REST an `arangod`, Ergebnis als JSON.

---

## 7. Zusammenfassung

Fassen wir die wichtigsten Erkenntnisse der Vorlesung zusammen:

### 7.1 Kernpunkte im Überblick

* **Warum DBMS?**

  * Abstraktion von physischer Datenspeicherung und effiziente Verwaltung großer Datenmengen.
  * Sicherstellung von Integrität, Konsistenz, Transaktionen und Mehrbenutzerzugriff.
  * Automatisierte Mechanismen für Backup, Recovery, Replikation und Skalierung.

* **Typen von Datenbank-Systemen**

  * **Relationale DBMS** (z. B. MariaDB, PostgreSQL): Tabellenbasiert, stark schemaorientiert, SQL.
  * **Zeitreihen-Datenbanken** (InfluxDB): Optimiert für zeitgestempelte Metriken, InfluxQL / Flux.
  * **Graphdatenbanken** (Neo4j): Knoten-Kanten-Modell, Pfadabfragen mit Cypher.
  * **Multi-Model-Systeme** (ArangoDB): Kombination aus Dokument-, Graph- und Key-Value-Modellen, AQL.
  * **Dateisysteme** (ext4): Grundlegende Persistenz ohne Metadatenmanagement und Transaktionen.

* **Abfragesprachen**

  * **Deklarativ vs. Imperativ**: SQL/Cypher vs. Shell-Kommandos, strukturierte vs. prozedurale Ansätze.
  * **Schema vs. schemalos**: Strenge Validierung vs. Flexibilität in Dokumenten- und Key-Value-Stores.
  * **Spezialisierte DSLs**: Flux für Zeitreihen, AQL für Multi-Model, Cypher für Graphen.


### 7.2 Entscheidungskriterien für das passende DBMS

* **Datenmodell & Use-Case**

  * Tabellarische Geschäfts­transaktionen → Relationale DBMS.
  * Sensordaten, Zeit­serienanalysen → Zeitreihen-Datenbanken.
  * Vernetzte Daten, Netzwerk­analysen → Graphdatenbanken.
  * Heterogene Daten, mehrere Modelle in einer Anwendung → Multi-Model-Systeme.
  * Einfache Datei­ablage ohne Metadatenbedarf → nativer Dateisystemzugriff.

* **Skalierbarkeit & Performance**

  * Vertikale vs. horizontale Skalierung (Sharding, Replikation).
  * Latenz­anforderungen (Echtzeit vs. Batch).
  * Schreib- vs. Lese­intensive Workloads.

* **Konsistenzanforderungen**

  * Strikte ACID-Garantien vs. eventual consistency.
  * Multi­regionale Replikation und Konflikt­lösung.

* **Betriebs- und Wartungsaufwand**

  * Administrativer Overhead (Backups, Schema­Migrationen).
  * Verfügbarkeit als Managed Service vs. Self‑Hosted.

* **Ökosystem & Integration**

  * Verfügbarkeit von Treibern, Libraries und GUI‑Tools.
  * Cloud‑Integrationen, CI/CD‑Pipelines, Monitoring.


### 7.3 Ausblick auf aktuelle Trends

* **Cloud-Angebote & DBaaS**

  * Voll managed Services (z. B. Amazon RDS, Azure Cosmos DB, Google Cloud Spanner).
  * Elastische Skalierung, automatische Patches, Pay‑as‑you‑go.

* **Multi-Model-Konzepte**

  * Flexible Datenhaltung in einem System, nahtlose Kombination von Dokumenten, Graphen und Key-Value.
  * Einfache Migration zwischen Modellen und hybride Anwendungsfälle.

* **NewSQL-Datenbanken**

  * Relationale Semantik mit horizontaler Skalierung (z. B. CockroachDB, Google Spanner).
  * Globale Konsistenz, verteilte Transaktionen ohne Performance­einbußen.

* **Edge & IoT**

  * Lokale Datenbanken direkt auf Geräten (SQLite, TinyDB) mit späterer Synchronisation.
  * Leichtgewichtige Time-Series-Datenbanken für eingebettete Systeme.

---
