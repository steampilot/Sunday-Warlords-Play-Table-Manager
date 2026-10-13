# Sunday Warlords Play Table Manager

Eine Webanwendung für den Tabletop-Game-Verein Sunday Warlords. Mitglieder sollen damit Spieltische für Vereinstreffen reservieren, Mitspieler eintragen und Vereins-Gelände für ihre Spiele buchen können.

Das Repository startet aktuell bei null. Dieses README hält den vereinbarten Umfang der ersten Version (MVP) und die nächsten Umsetzungsschritte fest.

## Ziel des MVP

- Vereinsmitglieder können sich anmelden.
- Administrierende Mitglieder verwalten Benutzerkonten und die Stammdaten der Anwendung.
- Ein Mitglied kann einen Tisch für einen freigegebenen Vereinstrefftag und einen Halbtag reservieren.
- Bei einer Reservierung können weitere Vereinsmitglieder aus der Mitgliederliste sowie Gäste per Namen eingetragen werden.
- Für die Reservierung wird entweder „Eigenes Gelände“ oder ein verfügbares Vereins-Geländeset ausgewählt.
- Mitglieder sehen und verwalten ihre eigenen Reservierungen. Administratoren können alle Reservierungen verwalten.
- Absagen bleiben als Historie erhalten und geben Tisch, Vereins-Geländeset und Mitglieder für den Zeitraum wieder frei.

## Vereinbarte Regeln

### Datum und Zeit

- Das Reservierungsdatum wird als Kalenderdatum (`DATE`) gespeichert.
- Im MVP sind nur vom Verein freigegebene Vereinstrefftage buchbar.
- Das Datum wird direkt in der Reservierung gespeichert und nicht über eine verpflichtende Trefftag-ID referenziert. So kann die Anwendung später auch Reservierungen an anderen Wochentagen erlauben, ohne das Datumsfeld oder bestehende Reservierungen umzustellen.
- Es gibt genau zwei Timeslots:
  - Vormittag: 10:00–13:00
  - Nachmittag: 13:00–16:00
- Eine Reservierung gilt für genau einen Halbtag. Wer beide Zeiträume benötigt, legt zwei Reservierungen an.
- Eine Ganztagesreservierung ist nicht Teil des MVP.

### Tische und Personen

- Tische werden administrativ gepflegt und nach Primarchen benannt.
- Pro Tisch, Datum und Timeslot ist höchstens eine aktive Reservierung zulässig.
- Der reservierende Benutzer zählt automatisch als Teilnehmer und wird nicht nochmals als Mitspieler gespeichert.
- Zusätzliche Vereinsmitglieder werden aus der Mitgliederliste ausgewählt.
- Gäste können ohne Benutzerkonto oder Vereinsmitgliedschaft mit Namen hinzugefügt werden. Die Oberfläche darf dafür eine durch Semikolons getrennte Eingabe anbieten; gespeichert werden die Gäste einzeln.
- Ein Vereinsmitglied darf am selben Datum und Timeslot nicht an mehreren Tischen eingetragen sein. Das gilt auch für den reservierenden Benutzer.
- Im MVP gibt es keine automatische Kapazitätsbegrenzung pro Tisch.

### Gelände

- Pro Reservierung wird genau eine Geländeoption gewählt.
- „Eigenes Gelände“ ist vorausgewählt und blockiert kein Vereins-Geländeset.
- Alternativ kann ein Vereins-Geländeset aus der Liste gewählt werden.
- Ein Vereins-Geländeset kann am selben Datum und Timeslot nur einer aktiven Reservierung zugeordnet sein – unabhängig davon, welcher Tisch reserviert wurde.
- In der ersten Version kann die Auswahl als Radio-Gruppe oder Dropdown umgesetzt werden. Eine Mehrfachauswahl per Checkbox ist eine mögliche spätere Erweiterung.

## Vorläufiges Datenmodell

Die endgültigen Feldnamen und CakePHP-Migrationen werden während der Implementierung festgelegt. Als fachlicher Entwurf sind folgende Tabellen vorgesehen:

| Tabelle | Zweck |
| --- | --- |
| `users` | Mitglieder- und Administratorkonten, Anmeldedaten, Rolle und Aktivstatus |
| `tables` | Tischliste mit Name und Aktivstatus |
| `terrain_sets` | Vereins-Geländesets mit Name und Aktivstatus |
| `bookable_dates` | Im MVP freigegebene Vereinstrefftage |
| `reservations` | Reservierender Benutzer, Tisch, Datum, Timeslot, optionale Referenz auf ein Vereins-Geländeset und Absageinformationen |
| `reservation_members` | Zuordnung zusätzlicher Vereinsmitglieder zu einer Reservierung |
| `reservation_guests` | Namen der Gäste einer Reservierung |

Ein fehlendes `terrain_set_id` in `reservations` steht für „Eigenes Gelände“. Die reservierende Person wird über `reservations.user_id` erfasst; zusätzliche Mitglieder werden in `reservation_members` gespeichert. Die konkreten Timeslot-Werte sollen stabile Schlüssel wie `morning` und `afternoon` verwenden.

Inaktive Mitglieder, Tische und Gelände-Sets sollen für vorhandene Reservierungen nachvollziehbar bleiben, aber nicht mehr für neue Reservierungen angeboten werden. Abgesagte Reservierungen werden nicht gelöscht. Der Tisch und das Gelände steht aber einer weiteren Reservation wieder zur verfügung.

Eine reservation hat Status Gebucht, Durchgeführt, Abgesagt, Storniert. Diese Status sollen in der Datenbank als ENUM oder vergleichbarer Typ abgebildet werden, um die Konsistenz zu gewährleisten.

## Geplante Technologie

- **Backend:** CakePHP; genaue Framework- und PHP-Version wird beim Projekt-Setup anhand der verfügbaren Laufzeit und aktuellen CakePHP-Anforderungen festgelegt.
- **Prototyp-Datenbank:** SQLite, ohne separaten laufenden Datenbankserver.
- **Späterer Betrieb:** Ein Wechsel zu MySQL/MariaDB ist vorgesehen. Schema und Anwendung sollen deshalb möglichst portabel bleiben. Vor dem produktiven Wechsel müssen Datenmigration und Datenbank-spezifische Constraints geprüft und getestet werden.

## Nächste Schritte

1. **Projektgrundlage erstellen**
   - Verfügbarkeit und Versionen von PHP und Composer prüfen.
   - CakePHP-Projekt mit passender, unterstützter Version initialisieren.
   - SQLite-Verbindung konfigurieren und den lokalen Entwicklungsstart dokumentieren.
2. **Datenbankschema umsetzen**
   - CakePHP-Migrationen für Benutzer, Tische, Gelände-Sets, freigegebene Tage und Reservierungen erstellen.
   - Zuordnungstabellen für zusätzliche Mitglieder und Gäste hinzufügen.
   - Rollen, Aktivstatus, Absagehistorie und Fremdschlüssel abbilden.
3. **Stammdaten und Anmeldung**
   - Anmeldung und Abmeldung implementieren.
   - Kontenanlage oder Einladungen durch Administratoren ermöglichen.
   - Admin-Verwaltung für Mitglieder, Tische, Trefftage und Gelände-Sets bereitstellen.
4. **Reservierungsregeln absichern**
   - Buchung nur für freigegebene Tage und aktive Stammdaten zulassen.
   - Konflikte für Tisch, Vereins-Geländeset und alle beteiligten Mitglieder pro Datum und Halbtag verhindern.
   - Prüfung und Speicherung so gestalten, dass parallele Buchungen keine Doppelbelegung erzeugen.
   - Absagen historisieren und Ressourcen wieder verfügbar machen.
5. **Reservierungsoberfläche erstellen**
   - Verfügbare Tage, Tische und Geländeoptionen anzeigen.
   - Reservierende Person automatisch berücksichtigen.
   - Zusätzliche Mitglieder und Gäste erfassen.
   - Eigene Reservierungen anzeigen, ändern und absagen; Admins dürfen alle Reservierungen verwalten.
6. **Tests und Betriebsreife**
   - Fachliche Regeln und Grenzfälle automatisiert testen, insbesondere Doppelbuchungen und Absagen.
   - Produktionskonfiguration, Backups und spätere SQLite-zu-MySQL/MariaDB-Migration planen.

## Noch zu entscheiden

- PHP- und CakePHP-Zielversion beim Setup.
- Details zum Einladungsablauf und Passwort-zurücksetzen.
- Konkrete Administrations- und Berechtigungsdetails.
- Hosting und produktive Datenbank, bevor die Anwendung online geht.
- Ob die spätere Wochentagsbuchung weiterhin eine manuell gepflegte Freigabeliste benötigt oder über erweiterte Verfügbarkeitsregeln gesteuert wird.
