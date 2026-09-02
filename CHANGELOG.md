# Changelog: pdf-export-Addon (LMOnext)

Vollständige, selbstgeschriebene PDF-Export-Engine für Ergebnisse, Tabelle,
Spielplan und Teamvergleich (Direkter Vergleich) - ohne externe Bibliothek.

Stammt aus dem LMOnext-Core (vorher src/Pdf/PdfExporter.php +
frontend/pdf_export.php) und wurde als eigenständiges Addon ausgegliedert
(Beitrag: Nutzerwunsch), damit es installationsweit über Administrator →
Einstellungen → Optionen → Anzeigen/Darstellung sichtbar aktiviert/
deaktiviert werden kann.

## Version 1.1.0

- Eigene Sprachdateien ergänzt (addon/pdf-export/lang/de.php + en.php). Bei genauerer Prüfung waren 3 der zuvor als "geteilt" eingestuften Sprachschlüssel doch addon-exklusiv (keine echte Verwendung außerhalb dieses Addons): liga_col_nr, liga_pdf_footer, h2h_pdf_renamed_note - aus lang/frontend/*.php verschoben (siehe dortiger Core-Changelog-Eintrag 1.49.0). pdf_export.php ruft jetzt beim Laden explizit addonManager()->loadLanguages('pdf-export') auf, da dieses Addon bewusst NICHT über frontend_handlers/bootFrontend() geladen wird (siehe Version 1.0.0 unten) und loadLanguages() sonst nie automatisch aufgerufen würde.

## Version 1.0.0

- Erste Version als eigenständiges Addon. Übernommen 1:1 aus dem Core
  (PdfExporter.php Fileversion 1.11.0, pdf_export.php Fileversion 1.9.1) -
  keine inhaltliche Verhaltensänderung, nur der Speicherort und die
  Aktivierbarkeit über den Addon-Manager sind neu.
- ZUSÄTZLICH, noch vor der ersten Auslieferung entfernt (PdfExporter.php
  1.12.0, pdf_export.php 1.9.2): der SVG-Rasterungs-Fallback
  pdfRasterizeSvgViaRsvgConvert() (externes Kommandozeilenwerkzeug
  "rsvg-convert" via shell_exec()) wurde komplett gestrichen. Grund: JEDE
  Form von Shell-Ausführung - auch mit korrekt escapten Argumenten - wird
  vom Sicherheits-Scanner des Addon-Managers (siehe Core-CHANGELOG.md
  src/Addon/AddonManager.php ab 1.1.0) als verdächtiges Muster erkannt und
  hätte die Installation/Aktualisierung DIESES Addons über ZIP-Upload oder
  GitHub-Update dadurch abgelehnt. Der primäre, shell-freie Weg über die
  Imagick-PHP-Erweiterung (pdfRasterizeSvgViaImagick()) bleibt vollständig
  erhalten - Funktionsverlust nur auf Servern ohne Imagick-SVG-Unterstützung,
  die zusätzlich rsvg-convert separat installiert hätten (kleine
  Schnittmenge, da beide Wege meist dieselbe librsvg-Bibliothek nutzen).
  Gegen den Sicherheits-Scanner erneut simuliert getestet: keine Treffer
  mehr im gesamten Addon-Paket.
- WICHTIG - abweichend vom sonst üblichen Addon-Lademuster: dieses Addon
  deklariert bewusst KEINEN Eintrag unter "frontend_handlers" in addon.json.
  pdf_export.php wird stattdessen weiterhin GEZIELT und BEDINGT direkt in
  liga.php geladen (require_once nur dort, geschützt durch
  addonManager()->isEnabled('pdf-export')) - eine Einbindung über
  AddonManager::bootFrontend() würde die Datei bei JEDEM Frontend-Request
  laden (auch home.php, Mini-Addons usw.), obwohl sie ausschließlich von
  liga.php tatsächlich gebraucht wird. Das entspricht der bereits vor der
  Auslagerung bestehenden, bewussten Performance-Entscheidung (siehe
  Kommentar in frontend/bootstrap.php).
- Neue Einstellung im Core sichtbar: Administrator → Einstellungen →
  Optionen → Anzeigen/Darstellung → "PDF-Buttons anzeigen" (vorhandenes
  Feld "show_pdf_buttons") erscheint jetzt nur noch, wenn dieses Addon
  aktiviert ist.
- min_core_version: 1.9.2 (benötigt die addon-abhängige
  $showPdfButtons-Prüfung in liga.php, frontend/data_liga_pretraits.php und
  src/Liga/HeadToHeadTrait.php).

**Für bestehende Installationen, die von einer älteren LMOnext-Version ohne
dieses Addon aktualisieren:** Nach dem Core-Update ist der PDF-Export
zunächst NICHT mehr verfügbar (Button verschwindet, ?pdf=1/h2h_pdf=1
liefert nichts mehr), bis dieses Addon zusätzlich installiert UND aktiviert
wird.
