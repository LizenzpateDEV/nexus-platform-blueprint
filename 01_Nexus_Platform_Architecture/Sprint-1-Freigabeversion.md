# NexusERP AI – Sprint 1: Finale Freigabeversion (v1.0)

**Rolle:** Lead Software Engineer
**Status:** Freigabe. Löst den Entwurf `Sprint-1-Technical-Implementation-Plan.md` ab.
**Grundlage:** akzeptierte ADR-0004/0005/0006 + die acht ratifizierten Entscheidungen unten.
**Inhalt:** ausschließlich Architektur/Planung – **kein Code.**

---

## 0. Entscheidungsgrundlage (verbindlich)

**Akzeptierte ADRs:** ADR-0004 (Workspace Architecture), ADR-0005 (Domain Boundaries), ADR-0006 (Application Structure). ADR-0002 (Shared Schema + RLS) ist durch die Entscheidungen 3/4 faktisch mitbestätigt (RLS-Scoping setzt Shared Schema voraus).

Ratifizierte Kernentscheidungen:
1. Workspace Architecture ist verbindlich.
2. Gültige Hierarchie: **Tenant → Workspace → Modul.**
3. Fachtabellen führen **`tenant_id` und `workspace_id`.**
4. **`workspace_id`** ist der primäre operative RLS-Scope für Fachdaten.
5. Rollenmodell unterstützt **Tenant-Rollen und Workspace-Rollen.**
6. Domänen werden als **Pakete** umgesetzt.
7. Apps werden nach **Produkt/Experience** geschnitten.
8. Für Sprint 1 bleibt **`apps/web` die einzige Produkt-App.**

---

## 1. Sprint-1-Ziel & Scope

**Ziel:** Ein lauffähiges, mandanten- und workspace-fähiges Plattform-Fundament nach Clean Architecture und DDD, auf dem ab Phase 3 die Fach-Domänen aufsetzen. Deckt die fundamentale Ebene von Roadmap Phase 2 ab: Identity, Tenant/Workspace, Permissions.

**In Scope:** Monorepo/Tooling/CI, lokale Supabase-Umgebung; Tenant- und Workspace-Fundament inkl. `tenant_id` + `workspace_id` + RLS ab Migration 1; Identity/Auth (Supabase Auth, SSR-Sessions); RBAC mit Tenant- und Workspace-Rollen; Data-Platform-Primitive (Audit, Basis-Entity-Konventionen); Module-Engine-Skelett (Aktivierung pro Workspace); API-Konventionen + Skelett-Endpunkte; UI-Shell mit Workspace-Switcher; Ausfüllen der von Sprint 1 berührten Specs (10_Database, 11_API, Core-Platform Identity/Tenant/Workspace/Permissions).

**Explizit NICHT in Scope:** alle Fach-Domänen (CRM, Sales, Orders, Invoices, …) – keine Fachlogik ohne Spec; Workflow-, AI- und Integration-Domäne (nur Platzhalterpakete); MFA/SSO; White-Label-Branding; Feld- und KI-Berechtigungs-Durchsetzung (Modell wird vorbereitet).

---

## 2. Finale Projektstruktur

Nach ADR-0006: Domänen als Pakete, Apps nach Produkt; in Sprint 1 genau eine Produkt-App. „Workspace & Tenant" bilden gemäß ADR-0005 **eine** Plattform-Domäne, daher liegt Workspace-Logik im Paket `platform/tenant`.

```
nexus/
├─ apps/
│  └─ web/                         # NexusERP AI – einzige Produkt-App (S1)
├─ packages/
│  ├─ platform/                    # 8 Kernsysteme als Bounded Contexts
│  │  ├─ identity/                 # S1
│  │  ├─ tenant/                   # S1  (enthält Workspace)
│  │  ├─ permission/               # S1
│  │  ├─ data/                     # S1  (Audit, Basis-Entity)
│  │  ├─ module-engine/            # S1  (Skelett, Aktivierung/Workspace)
│  │  ├─ workflow/                 # Platzhalter
│  │  ├─ ai/                       # Platzhalter (additiv, keine Abhängigkeit)
│  │  └─ integration/              # Platzhalter
│  ├─ modules/                     # Fach-Domänen (ADR-0005) – leer in S1
│  ├─ shared/                      # S1  – minimaler Shared Kernel
│  ├─ ui/                          # S1  – Design System
│  └─ config/                      # S1  – tsconfig/eslint/prettier/tailwind
├─ supabase/
│  ├─ migrations/                  # tenant_id + workspace_id + RLS ab Migration 1
│  └─ seed/                        # Default-Rollen
├─ turbo.json
└─ pnpm-workspace.yaml
```

**Interner Aufbau jedes Bounded Context** (Clean Architecture, Abhängigkeit nach innen):

```
packages/platform/tenant/
├─ domain/          # Entities, VOs, Repository-Interfaces, Domain Events – KEINE Framework-Abhängigkeit
├─ application/     # Use Cases, Ports, DTOs, Mapper
├─ infrastructure/ # Supabase/Postgres-Adapter
└─ index.ts         # öffentliche Schnittstelle
```

Abhängigkeitsregeln (per ESLint-Import-Grenzen erzwungen): Fach-Domänen dürfen Plattform-Domänen nutzen, nicht umgekehrt; keine Domäne importiert aus `platform/ai`; Zugriff auf fremde Domänen nur über deren `index.ts`, nie auf fremde `infrastructure`/Tabellen.

---

## 3. Finaler Datenbank-Scope

**Engine:** PostgreSQL via Supabase. **Mandantenmodell:** Shared Database, Shared Schema mit Row-Level Security.

**Scoping-Regel (verbindlich):**
- **Fachtabellen (Module):** `tenant_id` **und** `workspace_id`; RLS primär auf `workspace_id`, `tenant_id` transitiv/denormalisiert für tenant-weite Verarbeitung und Abrechnung.
- **Identity-/Tenant-/Workspace-Verwaltungstabellen:** Scope `tenant_id`; workspace-bezogene Zuordnungen zusätzlich `workspace_id`.

**Wichtige Präzisierung:** In Sprint 1 existieren noch **keine** Fachtabellen (Module kommen ab Phase 3). Sprint 1 **verankert die Regel** in den 10_Database-Specs, baut den RLS-Helper/das Muster und die Basis-Entity-Konventionen, sodass künftige Modultabellen sie erben. Angewandt wird `workspace_id` in S1 dort, wo Plattformtabellen workspace-bezogen sind (z. B. Mitgliedschaften, Workspace-Rollen).

**Basis-Entity-Konventionen** (Standard für künftige Fachtabellen, wo sinnvoll auch für Plattformtabellen):
- `id` UUID-PK (Empfehlung UUIDv7 – **offen: ADR-0001**), `snake_case`
- `tenant_id`, `workspace_id` (Fachtabellen)
- `created_at`, `updated_at`, `created_by`, `updated_by`
- `deleted_at` (Soft-Delete als adoptierter Standard; Policy verfeinerbar)

**Migrations:** vorwärtsgerichtet; `tenant_id` + `workspace_id` + RLS ab Migration 1 – nicht nachrüstbar.

**Konkrete Plattformtabellen in Sprint 1:**
- `tenants` (Wurzelgrenze)
- `workspaces` (`tenant_id`)
- `users` (Verknüpfung zu Supabase `auth.users`, `tenant_id`)
- `workspace_members` (`tenant_id`, `workspace_id`, `user_id`)
- `roles` (`tenant_id`, `scope` ∈ {tenant, workspace})
- `permissions` (Aktions-Stammdaten `resource.action`), `role_permissions`
- `user_tenant_roles` (Tenant-Rollen), `user_workspace_roles` (`workspace_id`, Workspace-Rollen)
- `audit_log` (append-only, `tenant_id`, optional `workspace_id`)
- `module_registry` + `workspace_module_activation` (Module-Engine-Skelett) – optional in S1

**RLS-Muster:** tenant-bezogene Tabellen über den `tenant_id`-Claim des JWT; workspace-bezogene Tabellen zusätzlich über Mitgliedschaft im aktiven Workspace (`workspace_members`).

---

## 4. Finales Auth-/Tenant-/Workspace-Modell

**Authentifizierung:** Supabase Auth mit E-Mail/Passwort und E-Mail-Verifikation; SSR-Sessions über `@supabase/ssr` (Cookies, Server Components/Actions). MFA/SSO für spätere Phasen vorgesehen.

**Tenant:** Organisations-, Lizenz- und Abrechnungsgrenze. **Onboarding (adoptiert aus ADR-0004):** Registrierung legt Tenant **und** einen Standard-Workspace an. **Sprint-1-Annahme:** ein Tenant je Benutzerkonto (Mehr-Tenant-Konten zurückgestellt – siehe offene Restfragen).

**Workspace:** operativer Container innerhalb des Tenants. Ein Benutzer erhält über `workspace_members` Zugriff auf einen oder mehrere Workspaces. Nach Login → zuletzt aktiver Workspace, sonst Workspace-Auswahl. **Routing (S1-Standard):** Workspace als Pfadsegment (`/w/{workspace}/…`); Tenant vorerst über JWT-Default auf einem gemeinsamen Host (Subdomain-pro-Tenant zurückgestellt bis Deployment-Entscheidung).

**Token/Kontext:** JWT trägt `tenant_id`; der aktive Workspace wird aus der Route bestimmt und gegen die Mitgliedschaft validiert, bevor Datenzugriffe erfolgen.

**Navigation:** workspace-bezogene Modul-Navigation plus ein Tenant-Verwaltungsbereich (Benutzer, Workspaces, Lizenzen, Einstellungen) für Träger von Tenant-Rollen.

---

## 5. Finales RBAC-Modell

**Zwei Geltungsbereiche (Entscheidung 5):**
- **Tenant-Rollen:** z. B. **Tenant Owner** (volle Tenant-Kontrolle inkl. Abrechnung, Workspaces, Benutzer), **Tenant Admin** (Benutzer/Workspaces verwalten, ohne Abrechnung).
- **Workspace-Rollen:** z. B. **Workspace Admin** (Workspace, Mitglieder, Modul-Aktivierung verwalten), **Workspace Member** (aktivierte Module gemäß Rechten nutzen).

*(Das genaue Default-Rollenset ist ein Vorschlag zur Bestätigung – siehe offene Restfragen.)*

**Datenmodell:** `roles(scope, tenant_id)`, `permissions(resource.action)`, `role_permissions`, Zuweisung getrennt über `user_tenant_roles` und `user_workspace_roles`. Rechte als `resource.action` (z. B. `tenant.workspace.create`, `permission.role.assign`, `workspace.member.invite`).

**Durchsetzung (Defense in Depth):** RLS für Tenant-/Workspace-Isolation und groben Zugriff **plus** ein `PermissionCheck`-Port in den Use Cases für Aktionsrechte.

**Feld- und KI-Berechtigungen:** Modell so angelegt, dass sie abbildbar sind (NPA-002, ADR-0003-Intention); **Durchsetzung in S1 zurückgestellt** (nicht blockierend).

**Sprint-1-Umfang:** Rollen mit Scope, Rechtedefinitionen für Plattformaktionen, `PermissionCheck`-Port, Seed der Default-Rollen.

---

## 6. Offene Restfragen

### 6a. Freigabe-Checkliste – adoptierte Standards (mit einem „Go" bestätigt oder überschrieben)
Diese sind nicht mehr fachlich blockierend; ich setze sie als Standard, sofern du nicht widersprichst:
1. **ADR-0001** formal bestätigen: PK-Strategie **UUIDv7**, `snake_case`, Pflicht-Audit-Spalten, `deleted_at`-Soft-Delete, vorwärtsgerichtete Migrations.
2. **User↔Tenant-Kardinalität:** S1 = ein Tenant je Benutzerkonto; Mehr-Tenant-Konten später.
3. **Routing/Host:** S1 = Workspace im Pfad, Tenant über JWT-Default auf gemeinsamem Host; Subdomain-pro-Tenant mit Deployment entscheiden.
4. **Onboarding:** Signup legt Tenant + Standard-Workspace an.
5. **Default-Rollenset:** Tenant Owner/Admin, Workspace Admin/Member.

### 6b. Zurückgestellt (nicht blockierend für Sprint 1)
- Durchsetzung von Feld- und KI-Berechtigungen (Modell ist vorbereitet).
- Historie/Versionierung, Suchstrategie (FTS/`pgvector`) – Data Platform, Phase 2+.
- KI-Provider und Datenresidenz – **DSGVO-relevant**, vor AI-Features zu klären.
- MFA/SSO – spätere Phase.
- Fachliche Modul-Specs – vor Bau der Module, nicht vor dem Fundament.
- Compliance-Anforderungen (DSGVO/GoBD/DATEV) formal spezifizieren – **vor** Finance/Invoices, nicht vor Sprint 1.

### 6c. Echt blockierend
Keine mehr, sofern 6a bestätigt wird. Die zuvor größte Blockade (undefinierter Workspace-Begriff) ist durch ADR-0004 aufgelöst.

---

## 7. Konkreter Implementierungsablauf für Sprint 1

Reihenfolge nach Abhängigkeiten. WP10 (Specs) läuft durchgängig mit.

| WP | Bereich | Inhalt |
|----|---------|--------|
| WP1 | Monorepo & Tooling | pnpm + Turborepo, TS-/ESLint-/Prettier-/Tailwind-Config, Import-Grenzen-Lint, CI |
| WP2 | Supabase & DB-Basis | lokale Umgebung, Migrations-Baseline, RLS-Helper, Basis-Entity-/Scoping-Konventionen (10_Database) |
| WP3 | Identity | Supabase Auth, SSR-Sessions, Registrierung/Login, `tenant_id`-Claim |
| WP4 | Tenant & Workspace | `tenants`/`workspaces`/`workspace_members`, Tenant+Workspace-Auflösung, Onboarding (Signup → Tenant + Standard-Workspace) |
| WP5 | RBAC | Rollen mit Scope, Rechte, Zuweisungen, `PermissionCheck`-Port, Seed Default-Rollen |
| WP6 | Data Platform | `audit_log` (append-only), Basis-Entity-Konventionen angewandt |
| WP7 | Module Engine | Skelett `module_registry` + Aktivierung pro Workspace |
| WP8 | API | v1-Envelope, Fehlerformat, Auth-/Tenant-/Workspace-Middleware; Endpunkte: Health, `/me`, `/tenant`, `/workspaces` (11_API) |
| WP9 | UI-Shell | Auth-Screens, App-Shell, Workspace-Switcher, Tenant-Admin-Platzhalter, Modul-Navigations-/Dashboard-Platzhalter (Core-Platform Navigation/Dashboard) |
| WP10 | Specs | 10_Database, 11_API, Core-Platform Identity/Tenant/Workspace/Permissions ausfüllen |

**Hinweis:** WP3 (Identity) und WP4 (Tenant/Workspace) greifen über das Onboarding ineinander und werden eng verzahnt umgesetzt.

---

## 8. Definition of Done (Sprint 1)

- RLS ist auf allen tenant-/workspace-bezogenen Tabellen aktiv und getestet: **weder tenant- noch workspace-übergreifender** Zugriff möglich.
- Registrierung legt Tenant + Standard-Workspace an; Login führt in ein workspace-bezogenes, leeres Dashboard.
- Default-Rollen (Tenant- und Workspace-Scope) sind geseedet; der `PermissionCheck`-Port ist aufrufbar.
- API: `GET /api/v1/me` liefert Benutzer, Tenant, zugängliche Workspaces und Rollen; `/workspaces` und Health funktionieren.
- Workspace-Switcher funktioniert (bei mehreren Workspaces); der Tenant-Admin-Bereich ist durch Tenant-Rollen geschützt.
- CI ist grün inkl. Architektur-Import-Grenzen, Typecheck und Migrations.
- Die von Sprint 1 berührten Specs sind ausgefüllt; ADR-0004/0005/0006 sind akzeptiert (erledigt).

---

**Nächster Schritt:** Bestätige die Freigabe-Checkliste 6a (oder überschreibe einzelne Punkte). Danach ist Sprint 1 startklar; Code entsteht wie vereinbart erst nach deiner Go-Freigabe für WP1.
