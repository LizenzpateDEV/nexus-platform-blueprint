# NPA-002 – Core Systems

## Ziel

Die Nexus Platform besteht aus acht zentralen Kernsystemen.

Jedes zukünftige Modul muss ausschließlich diese Kernsysteme verwenden.

---

# 1. Identity System

Verantwortlich für:

- Login
- Registrierung
- MFA
- OAuth
- SSO
- Sessions
- API Keys

---

# 2. Tenant System

Verantwortlich für:

- Unternehmen
- Standorte
- Lizenzen
- Branding
- Sprache
- Währung

---

# 3. Permission System

Verantwortlich für:

- Rollen
- Rechte
- Policies
- Feldberechtigungen

---

# 4. Data Platform

Verantwortlich für:

- Daten
- Dateien
- Historie
- Audit
- Suche
- Beziehungen

---

# 5. Module Engine

Verantwortlich für:

- Registrierung von Modulen
- Modul-Lebenszyklus
- Aktivierung
- Versionierung

---

# 6. Workflow Engine

Verantwortlich für:

- Prozesse
- Regeln
- Genehmigungen
- Automatisierung

---

# 7. AI Platform

Verantwortlich für:

- KI-Assistent
- Agenten
- Zusammenfassungen
- Analysen
- Vorschläge

---

# 8. Integration Platform

Verantwortlich für:

- REST API
- Webhooks
- DATEV
- Microsoft 365
- Google
- Shopify
- Stripe
- PayPal

---

## Architekturregel

Module kommunizieren niemals direkt miteinander.

Jede Kommunikation erfolgt ausschließlich über definierte Services und APIs.