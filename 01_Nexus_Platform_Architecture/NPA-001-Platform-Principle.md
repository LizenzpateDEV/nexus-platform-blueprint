# NPA-001 – Platform Principle

## Status

Approved

---

## Zweck

NexusERP AI ist keine einzelne ERP-Anwendung.

NexusERP AI ist das erste Produkt auf der Nexus Platform.

Die Nexus Platform bildet die technische Grundlage für alle zukünftigen Produkte.

---

## Architekturprinzip

Alle gemeinsamen Funktionen gehören zur Plattform.

Alle geschäftsspezifischen Funktionen gehören in Module.

---

## Plattform übernimmt

- Authentifizierung
- Benutzerverwaltung
- Rollen & Rechte
- Mandanten
- Datenhaltung
- Dokumente
- Workflow Engine
- KI-Plattform
- Integrationen
- Benachrichtigungen
- Dashboard Framework
- API Gateway
- Audit Log
- Suche

---

## ERP Module übernehmen

- CRM
- Artikel
- Angebote
- Aufträge
- Rechnungen
- Lager
- Einkauf
- Finanzen
- Projekte
- Dokumente

---

## Grundprinzip

Die Plattform darf niemals Geschäftslogik eines ERP-Moduls enthalten.

Module dürfen ausschließlich über definierte APIs miteinander kommunizieren.

---

## KI-Prinzip

Jede Funktion muss vollständig ohne KI funktionieren.

KI unterstützt den Nutzer durch:

- Vorschläge
- Automatisierung
- Analysen
- Zusammenfassungen
- Assistenten

KI ersetzt niemals Pflichtfunktionen.

---

## Ziel

Eine Plattform schaffen, auf der später weitere Produkte entstehen können:

- NexusCRM
- NexusDocs
- NexusHR
- NexusFlow
- NexusBI