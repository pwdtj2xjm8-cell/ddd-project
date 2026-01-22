# Feature Specification: Produktionschargen-Management

**Feature Branch**: `001-produktionscharge`
**Created**: 2026-01-22
**Status**: Draft
**Input**: User description: "Produktionschargen-Management Microservice mit Event Sourcing"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Produktionscharge erstellen (Priority: P1)

Als Produktionsleiter möchte ich einen neuen Produktionsauftrag freigeben, um eine
Produktionscharge zu starten und deren Lebenszyklus zu verfolgen.

**Why this priority**: Dies ist die grundlegende Funktion, ohne die keine anderen
Features nutzbar sind. Eine Charge muss existieren, bevor Qualitätsprüfungen oder
Statusänderungen möglich sind.

**Independent Test**: Kann vollständig getestet werden, indem ein Produktionsleiter
eine neue Charge erstellt und deren Erstellung im System bestätigt wird.

**Acceptance Scenarios**:

1. **Given** der Produktionsleiter ist authentifiziert, **When** er einen neuen
   Produktionsauftrag freigibt mit Charge-Bezeichnung und Produktdaten, **Then**
   wird eine neue Produktionscharge erstellt mit Status "Freigegeben".

2. **Given** eine Charge wurde erstellt, **When** der Produktionsleiter die
   Chargen-Details abfragt, **Then** werden alle Stammdaten und der aktuelle
   Status angezeigt.

3. **Given** keine Charge-Bezeichnung angegeben, **When** der Produktionsauftrag
   freigegeben werden soll, **Then** wird ein Validierungsfehler zurückgegeben.

---

### User Story 2 - Automatische Qualitätsprüfung (Priority: P2)

Als System möchte ich nach Erstellung einer Produktionscharge automatisch eine
Qualitätsprüfung beim externen Qualitätsmanagementsystem anstoßen, damit der
Qualitätsstatus zeitnah ermittelt wird.

**Why this priority**: Die automatische Qualitätsprüfung ist der Kern des
Geschäftsprozesses und bestimmt, ob eine Charge freigegeben oder gesperrt wird.

**Independent Test**: Kann getestet werden, indem eine Charge erstellt wird und
überprüft wird, dass automatisch eine Qualitätsprüfung angefordert und das
Ergebnis verarbeitet wird.

**Acceptance Scenarios**:

1. **Given** eine Produktionscharge wurde erstellt (ProduktionschargeFreigegeben-Event),
   **When** die WennProduktionFertig-Policy ausgelöst wird, **Then** wird
   automatisch eine Qualitätsprüfung beim externen System angestoßen.

2. **Given** das externe Qualitätssystem liefert "Bestanden", **When** das
   QualitätsprüfungDurchgeführt-Event verarbeitet wird, **Then** wird die Charge
   automatisch freigegeben (Status: "Freigegeben für Auslieferung").

3. **Given** das externe Qualitätssystem liefert "Nicht bestanden", **When** das
   QualitätsprüfungDurchgeführt-Event verarbeitet wird, **Then** wird die Charge
   automatisch gesperrt (Status: "Gesperrt").

---

### User Story 3 - Chargen-Status einsehen (Priority: P3)

Als Produktionsleiter möchte ich den aktuellen Status aller Chargen einsehen können,
um einen Überblick über den Produktionsprozess zu haben.

**Why this priority**: Das Read Model für den Chargen-Status ist essentiell für
die tägliche Arbeit, baut aber auf den vorherigen Stories auf.

**Independent Test**: Kann getestet werden, indem mehrere Chargen in verschiedenen
Status erstellt werden und die Statusübersicht korrekte Informationen anzeigt.

**Acceptance Scenarios**:

1. **Given** mehrere Chargen existieren in verschiedenen Status, **When** der
   Produktionsleiter die Statusübersicht abruft, **Then** werden alle Chargen
   mit ihrem aktuellen Status angezeigt.

2. **Given** eine Charge existiert, **When** der Produktionsleiter nach einer
   bestimmten Charge-ID filtert, **Then** wird nur diese Charge mit Details angezeigt.

---

### User Story 4 - Qualitätsbericht einsehen (Priority: P4)

Als Produktionsleiter möchte ich den Qualitätsbericht einer Charge einsehen können,
um nachzuvollziehen, warum eine Charge freigegeben oder gesperrt wurde.

**Why this priority**: Wichtig für Nachvollziehbarkeit und Auditing, aber nicht
für den Kernprozess erforderlich.

**Independent Test**: Kann getestet werden, indem der Qualitätsbericht für eine
Charge mit abgeschlossener Prüfung abgerufen und dessen Inhalt verifiziert wird.

**Acceptance Scenarios**:

1. **Given** eine Qualitätsprüfung wurde für eine Charge durchgeführt, **When**
   der Produktionsleiter den Qualitätsbericht abruft, **Then** wird das
   Prüfungsergebnis (Bestanden/Nicht bestanden) mit Zeitstempel angezeigt.

---

### User Story 5 - Event-Historie einsehen (Priority: P5)

Als Produktionsleiter möchte ich die vollständige Event-Historie einer Charge
einsehen können, um alle Statusänderungen nachvollziehen zu können.

**Why this priority**: Wichtig für Compliance und Auditing, nutzt das Event
Sourcing Pattern für lückenlose Nachverfolgung.

**Independent Test**: Kann getestet werden, indem eine Charge durch mehrere
Statusänderungen geführt wird und die Event-Historie alle Schritte zeigt.

**Acceptance Scenarios**:

1. **Given** eine Charge hat mehrere Statusänderungen durchlaufen, **When** der
   Produktionsleiter die Event-Historie abruft, **Then** werden alle Events
   chronologisch mit Zeitstempel angezeigt.

---

### Edge Cases

- Was passiert, wenn das externe Qualitätssystem nicht erreichbar ist?
  → System speichert den Aufruf und führt Retry durch; Charge bleibt im Status
  "Warte auf Qualitätsprüfung".

- Was passiert bei doppeltem Freigabe-Command für dieselbe Charge?
  → Invariante im Aggregate verhindert dies; Fehler wird zurückgegeben.

- Was passiert, wenn eine bereits gesperrte Charge erneut gesperrt werden soll?
  → Idempotente Verarbeitung; keine Statusänderung, kein Fehler.

- Was passiert bei ungültigen Status-Übergängen (z.B. von "Gesperrt" zu "Freigegeben")?
  → Aggregate-Invariante lehnt ab; nur explizite Entsperrung möglich (falls
  implementiert in Zukunft).

## Requirements *(mandatory)*

### Functional Requirements

**Chargen-Erstellung:**
- **FR-001**: System MUST Produktionschargen mit eindeutiger ID erstellen können
- **FR-002**: System MUST Charge-Bezeichnung und Produktdaten bei Erstellung validieren
- **FR-003**: System MUST bei erfolgreicher Erstellung ein ProduktionschargeFreigegeben-Event speichern

**Qualitätsprüfung:**
- **FR-004**: System MUST nach Chargen-Erstellung automatisch eine Qualitätsprüfung anstoßen (Policy)
- **FR-005**: System MUST das externe Qualitätsmanagementsystem für die Prüfung aufrufen (simuliert als Mock im MVP)
- **FR-006**: System MUST Qualitätsprüfungsergebnisse als QualitätsprüfungDurchgeführt-Event speichern

**Statusmanagement:**
- **FR-007**: System MUST bei "Bestanden"-Ergebnis die Charge automatisch freigeben
- **FR-008**: System MUST bei "Nicht bestanden"-Ergebnis die Charge automatisch sperren
- **FR-009**: System MUST ungültige Status-Übergänge durch Aggregate-Invarianten verhindern

**Read Models:**
- **FR-010**: System MUST einen ChargeStatus-Read Model bereitstellen (Status: Freigegeben, In Prüfung, Freigegeben für Auslieferung, Gesperrt)
- **FR-011**: System MUST einen Qualitätsbericht-Read Model bereitstellen (Ergebnis: Bestanden, Nicht bestanden)
- **FR-012**: System MUST Event-Historie pro Charge abrufbar machen

**API:**
- **FR-013**: System MUST Command-Endpunkte für ProduktionsauftragFreigeben bereitstellen
- **FR-014**: System MUST Query-Endpunkte für ChargeStatus und Qualitätsbericht bereitstellen
- **FR-015**: System MUST Query-Endpunkt für Event-Historie bereitstellen

**Event Sourcing:**
- **FR-016**: System MUST alle Zustandsänderungen als Events persistieren
- **FR-017**: System MUST den aktuellen Zustand aus Events rekonstruieren können

### Key Entities

- **Produktionscharge (Aggregate Root)**: Repräsentiert eine Produktionscharge mit
  eindeutiger ID, Bezeichnung, Produktdaten, Status und Erstellungszeitpunkt.
  Einzige Aggregate Root im System.

- **ProduktionschargeFreigegeben (Event)**: Domänen-Event bei Erstellung einer Charge.
  Enthält Charge-ID, Bezeichnung, Produktdaten und Zeitstempel.

- **QualitätsprüfungDurchgeführt (Event)**: Domänen-Event nach Abschluss einer
  Qualitätsprüfung. Enthält Charge-ID, Ergebnis (Bestanden/Nicht bestanden),
  Prüfungszeitpunkt.

- **ChargeStatus (Read Model)**: Projektion für schnellen Statusabruf.
  Enthält Charge-ID, Bezeichnung, aktueller Status, letzte Aktualisierung.

- **Qualitätsbericht (Read Model)**: Projektion für Qualitätsprüfungsergebnisse.
  Enthält Charge-ID, Prüfungsergebnis, Prüfungszeitpunkt.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Produktionsleiter kann eine neue Charge in unter 10 Sekunden erstellen
- **SC-002**: System verarbeitet Qualitätsprüfungsergebnisse innerhalb von 5 Sekunden
  nach Eingang
- **SC-003**: Status-Abfragen liefern Ergebnisse in unter 1 Sekunde
- **SC-004**: 100% der Statusänderungen sind in der Event-Historie nachvollziehbar
- **SC-005**: Alle ungültigen Status-Übergänge werden vom System abgelehnt (0% Durchlass)
- **SC-006**: System läuft stabil in einem Container ohne manuelle Eingriffe

## Assumptions

- MVP verwendet Mock-Implementation für das externe Qualitätsmanagementsystem
- Authentifizierung ist für MVP nicht im Scope; wird als gegeben angenommen
- Ein Produktionsleiter ist der einzige Akteur im System
- Alle Zeiten beziehen sich auf lokale Systemzeit
- Docker-Container läuft auf Standard-Hardware ohne besondere Anforderungen
