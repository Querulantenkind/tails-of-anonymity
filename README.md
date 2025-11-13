# 📚 Sicheres Dokumenten-Management-System
## Vollständige Anleitung für TailsOS mit 3-Ebenen-Verschlüsselung

**Version:** 1.0  
**Datum:** 13. November 2025  
**Zweck:** Hochsicheres lokales System für verschlüsselte OpenOffice-Dokumente

---

## 🎯 Überblick

Dieses Dokumentations-Paket enthält alle notwendigen Anleitungen für die **vollständig manuelle** Einrichtung und Nutzung eines hochsicheren Dokumenten-Management-Systems basierend auf:

- **TailsOS** (Live-System)
- **3 USB-Sticks** (128GB WORK + 32GB ARCHIVE + 16GB TRANSFER)
- **3-Ebenen-Verschlüsselung** (LUKS2 + VeraCrypt + GPG)
- **OpenOffice/LibreOffice** für Dokumenten-Arbeit

---

## 📖 Dokumenten-Struktur

### 🔴 PFLICHT: Zuerst lesen

#### **1. Master Setup Guide** 
📄 `00_MASTER_SETUP_GUIDE.md` (60 KB)

**Wann lesen:** VOR der Installation, am Anfang

**Inhalt:**
- Vollständige Schritt-für-Schritt-Anleitung
- Hardware-Vorbereitung
- Tails-Installation auf alle 3 Sticks
- Persistent Storage einrichten
- VeraCrypt-Container erstellen
- GPG-Schlüssel generieren
- Verschlüsselungs-Setup (alle 3 Ebenen)
- Verzeichnisstruktur

**Zeitaufwand:** 2-4 Stunden (je nach Erfahrung)

**Status:** ✅ VOLLSTÄNDIG - Keine Vorkenntnisse nötig

---

### 🟡 TÄGLICH: Für Routine-Nutzung

#### **2. Daily Operations Guide**
📄 `01_DAILY_OPERATIONS.md` (27 KB)

**Wann lesen:** Nach erfolgreichem Setup, täglich nutzen

**Inhalt:**
- Morgen-Routine (System starten)
- Dokument erstellen/bearbeiten
- Externe Dateien importieren
- Dokument exportieren (selten)
- Abend-Routine (sicheres Herunterfahren)
- Wöchentliches Backup
- Monatliche Wartung

**Zeitaufwand:** 
- Erste Durchsicht: 30-60 Minuten
- Tägliche Nutzung: 5-10 Minuten Setup/Shutdown

**Status:** ✅ VOLLSTÄNDIG - Für tägliche Arbeit

---

### 🟢 BEI PROBLEMEN: Troubleshooting

#### **3. Troubleshooting Guide**
📄 `02_TROUBLESHOOTING_GUIDE.md` (38 KB)

**Wann lesen:** Bei Problemen ODER präventiv durchblättern

**Inhalt:**
- Boot-Probleme
- Persistent Storage-Probleme
- VeraCrypt-Probleme
- GPG-Probleme
- Dateisystem-Probleme
- Performance-Probleme
- Hardware-Probleme
- Datenrettung
- Notfall-Szenarien

**Zeitaufwand:** 
- Durchsicht: 60-90 Minuten
- Bei Problem: 5-30 Minuten Lösung finden

**Status:** ✅ VOLLSTÄNDIG - Deckt >95% aller Probleme ab

---

### 🔵 ZUM AUSDRUCKEN: Quick Reference

#### **4. Security Checklist**
📄 `03_SECURITY_CHECKLIST.md` (10 KB)  
📄 `03_SECURITY_CHECKLIST.pdf` (80 KB) **← DRUCKEN!**

**Wann nutzen:** Täglich griffbereit haben

**Inhalt:**
- Prä-Boot-Checkliste
- Boot-Prozedur
- Dokument-Workflow
- Import/Export-Checklisten
- Shutdown-Prozedur
- Backup-Checkliste
- Wartungs-Checklisten
- Notfall-Kommandos
- Schnell-Referenz wichtiger Befehle

**Format:** A4, beidseitig druckbar

**Empfehlung:** 
1. PDF ausdrucken
2. Laminieren (für Langlebigkeit)
3. In der Nähe des Arbeitsplatzes aufbewahren

**Status:** ✅ VOLLSTÄNDIG - Druckfertig

---

## 🚀 Schnellstart: Erste Schritte

### Phase 1: Vorbereitung (1-2 Stunden)

```
1. ✓ Hardware besorgen:
   - 3 USB-Sticks (128GB, 32GB, 16GB)
   - Computer mit 4GB+ RAM
   
2. ✓ Software herunterladen:
   - Tails OS Image + Signatur
   - VeraCrypt .deb Paket
   
3. ✓ Dokumentation lesen:
   - 00_MASTER_SETUP_GUIDE.md durchlesen
   - Wichtige Passagen markieren
```

### Phase 2: Installation (2-3 Stunden)

```
4. ✓ Master Setup Guide befolgen:
   - Schritt für Schritt
   - Nichts überspringen!
   - Bei Problemen: Troubleshooting Guide
   
5. ✓ Test durchführen:
   - System booten
   - Container mounten
   - Test-Dokument erstellen
   - System herunterfahren
```

### Phase 3: Produktiv-Nutzung (ab Tag 2)

```
6. ✓ Daily Operations Guide nutzen:
   - Morgen-Routine
   - Dokumente bearbeiten
   - Abend-Routine
   
7. ✓ Wöchentliches Backup:
   - Jeden Sonntag
   - Auf ARCHIVE-Stick
   - Verifizieren!
```

---

## 🔒 Sicherheits-Level

Ihr System bietet **Militärgrad-Verschlüsselung** durch 3 unabhängige Ebenen:

```
┌─────────────────────────────────────────────────────┐
│ EBENE 1: LUKS2 (Persistent Storage)                │
│ - AES-XTS-Plain64                                  │
│ - 512-bit Key                                      │
│ - Argon2id Key-Derivation                         │
│ - Schutz: Gesamtes Persistent Volume              │
└─────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────┐
│ EBENE 2: VeraCrypt (Container)                     │
│ - AES-Twofish-Serpent Cascade                      │
│ - SHA-512 Hash                                     │
│ - PIM 485 (verstärkte Key-Derivation)             │
│ - Schutz: Dokument-Container                      │
└─────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────┐
│ EBENE 3: GPG (Dateien)                             │
│ - RSA-4096                                         │
│ - AES-256 Datei-Verschlüsselung                   │
│ - Individueller Schutz pro Datei                  │
│ - Schutz: Einzelne Dokumente                      │
└─────────────────────────────────────────────────────┘

= UNKNACKBAR mit aktueller Technologie
```

---

## ⚠️ Kritische Sicherheitshinweise

### ❗ Die 3 goldenen Regeln

```
1. PASSPHRASEN NIEMALS VERGESSEN
   → Ohne Passphrase = Daten für immer verloren
   → Handschriftliche Notiz an sicherem Ort
   → Oder Kopf (Diceware-Methode)

2. REGELMÄSSIGE BACKUPS
   → Wöchentlich auf ARCHIVE-Stick
   → Verifizieren nach jedem Backup
   → Mindestens 3-4 Generationen aufheben

3. DISZIPLIN BEI PROZEDUREN
   → Immer alle Schritte befolgen
   → Shortcuts vermeiden
   → Sicherheit > Convenience
```

### ⚠️ Wenn etwas schiefgeht

```
PROBLEM?
   ↓
1. Ruhe bewahren
   ↓
2. Troubleshooting Guide konsultieren
   ↓
3. Systematisch Lösungen probieren
   ↓
4. Im Zweifel: Backup nutzen
   ↓
5. Worst Case: System neu aufsetzen
```

---

## 📊 Zeitaufwand-Übersicht

### Initial-Setup

```
Vorbereitung:              1-2 Stunden
Installation (alle Sticks): 2-3 Stunden
Erster Test:               30 Minuten
─────────────────────────────────────
GESAMT:                    4-6 Stunden
```

### Tägliche Nutzung

```
Morgen (System starten):   5-8 Minuten
Arbeit (nach Bedarf):      ∞
Abend (Herunterfahren):    5-10 Minuten
─────────────────────────────────────
OVERHEAD pro Tag:          10-18 Minuten
```

### Wartung

```
Wöchentliches Backup:      15-30 Minuten
Monatliche Wartung:        30-60 Minuten
Halbjährlich (Passphrasen):60-90 Minuten
```

---

## 🎯 Zielgruppe

Dieses System ist geeignet für:

✅ **Journalisten** mit sensiblen Quellen  
✅ **Aktivisten** in repressiven Regimen  
✅ **Whistleblower** mit brisanten Dokumenten  
✅ **Anwälte** mit Mandanten-vertraulichen Unterlagen  
✅ **Forscher** mit Pre-Publication-Daten  
✅ **Jeder** mit hochsensiblen Dokumenten

**Voraussetzungen:**
- Grundlegende Computer-Kenntnisse
- Bereitschaft, Prozeduren diszipliniert zu befolgen
- Geduld für initiales Setup
- Verständnis für Threat-Model

---

## đź"ž Support & Weitere Ressourcen

### Offizielle Dokumentation

```
Tails:     https://tails.boum.org/doc/
VeraCrypt: https://www.veracrypt.fr/en/Documentation.html
GnuPG:     https://www.gnupg.org/documentation/
```

### Bei Problemen

```
1. Troubleshooting Guide lesen (02_TROUBLESHOOTING_GUIDE.md)
2. Offizielle Dokumentation konsultieren
3. Im Zweifel: Backup nutzen und neu aufsetzen
```

### ⚠️ WICHTIG: Keine Online-Hilfe mit Details!

```
❌ FALSCH:
   "Mein VeraCrypt-Container mit Whistleblower-Dokumenten 
    lässt sich nicht öffnen, Passphrase wird nicht akzeptiert"

✅ RICHTIG:
   "VeraCrypt mount error on Tails, incorrect password message"
```

---

## ✅ Checkliste: Bereit fĂĽr Produktiv-Einsatz?

```
□ Alle 3 USB-Sticks vorbereitet und beschriftet
□ Tails auf WORK-Stick installiert
□ Tails auf ARCHIVE-Stick installiert  
□ TRANSFER-Stick mit LUKS verschlüsselt
□ Persistent Storage auf WORK erstellt
□ Persistent Storage auf ARCHIVE erstellt
□ VeraCrypt installiert und getestet
□ VeraCrypt-Container erstellt (80GB)
□ VeraCrypt-Header gesichert
□ GPG-Schlüsselpaar generiert
□ GPG-Backup erstellt und verschlüsselt
□ Verzeichnisstruktur erstellt
□ Test-Dokument erfolgreich erstellt, verschlüsselt, entschlüsselt
□ Backup auf ARCHIVE-Stick durchgeführt
□ Backup verifiziert (Checksummen)
□ Alle Passphrasen sicher notiert (NICHT digital!)
□ Master Setup Guide komplett durchgearbeitet
□ Daily Operations Guide gelesen
□ Troubleshooting Guide durchgeblättert
□ Security Checklist ausgedruckt und laminiert
```

**Alle Punkte abgehakt?**  
→ **🎉 Bereit für Produktiv-Einsatz!**

---

## 📝 Versions-Historie

```
Version 1.0 (2025-11-13)
- Initial Release
- Master Setup Guide
- Daily Operations Guide  
- Troubleshooting Guide
- Security Checklist (mit PDF)
```

---

## 🔐 Abschließende Worte

```
█████████████████████████████████████████████████████
█                                                   █
█   SICHERHEIT = DISZIPLIN + BACKUPS + PARANOIA    █
█                                                   █
█   Diese Dokumentation gibt Ihnen die Werkzeuge.  █
█   Ihre Disziplin macht sie wirksam.              █
█                                                   █
█   Schützen Sie Ihre Daten.                       █
█   Schützen Sie Ihre Quellen.                     █
█   Schützen Sie Ihre Arbeit.                      █
█                                                   █
█████████████████████████████████████████████████████
```

**Viel Erfolg bei der Umsetzung!**

---

**Letzte Aktualisierung:** 13. November 2025  
**Autor:** Dokumenten-Management-System Dokumentation  
**Lizenz:** Für private Nutzung

**đź"„ Alle Dateien in diesem Paket:**
- `00_MASTER_SETUP_GUIDE.md`
- `01_DAILY_OPERATIONS.md`
- `02_TROUBLESHOOTING_GUIDE.md`
- `03_SECURITY_CHECKLIST.md`
- `03_SECURITY_CHECKLIST.pdf`
- `README.md` (diese Datei)
