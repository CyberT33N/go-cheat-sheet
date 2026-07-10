# go-cheat-sheet


# Install

## Linux
```
apt install golang -y
```

## Windows
- https://go.dev/dl/

Man kann die MSI-Datei herunterladen; sie fügt den Pfad automatisch hinzu.



```shell
notepad $PROFILE

# Add this
$env:Path += ";C:\Go\bin"
```










---

# gcc

## Install

### windows

# GCC unter Windows installieren und für Go/cgo einrichten

## Ziel

Diese Anleitung installiert **GCC über MSYS2/MinGW-w64**, fügt GCC dauerhaft zur Windows-Umgebungsvariable `PATH` hinzu und prüft anschließend die Verwendung mit Go/cgo.

Go benötigt unter Windows für cgo einen GCC-kompatiblen Compiler wie MinGW-w64. Die Datei `gcc.exe` muss über `PATH` erreichbar sein.

---

## 1. MSYS2 installieren

1. Lade den aktuellen MSYS2-Installer von der offiziellen MSYS2-Website herunter.
2. Starte den Installer.
3. Verwende möglichst den vorgeschlagenen Installationsordner:

```text
C:\msys64
```

4. Schließe die Installation ab.
5. Öffne anschließend über das Startmenü:

```text
MSYS2 UCRT64
```

MSYS2 empfiehlt bei Unsicherheit die **UCRT64-Umgebung**. Sie verwendet GCC, 64-Bit-x86 und die moderne Windows Universal C Runtime.

> Wichtig: Öffne nicht nur „MSYS2 MSYS“, sondern ausdrücklich **MSYS2 UCRT64**.

---

## 2. MSYS2 aktualisieren

Führe im geöffneten Fenster **MSYS2 UCRT64** folgenden Befehl aus:

```bash
pacman -Syu
```

Bestätige Rückfragen mit:

```text
Y
```

oder durch Drücken von:

```text
Enter
```

Falls MSYS2 das Terminal während der Aktualisierung schließt:

1. Öffne **MSYS2 UCRT64** erneut.
2. Führe den Befehl noch einmal aus:

```bash
pacman -Syu
```

MSYS2 verwendet `pacman` zur Installation und Aktualisierung seiner Pakete.

---

## 3. GCC installieren

Führe weiterhin im Fenster **MSYS2 UCRT64** diesen Befehl aus:

```bash
pacman -S mingw-w64-ucrt-x86_64-gcc
```

Bestätige die Installation mit `Enter` beziehungsweise `Y`.

Das offizielle GCC-Paket für die UCRT64-Umgebung heißt:

```text
mingw-w64-ucrt-x86_64-gcc
```

---

## 4. GCC innerhalb von MSYS2 prüfen

Führe im **MSYS2-UCRT64-Terminal** aus:

```bash
gcc --version
```

Danach:

```bash
which gcc
```

Der Pfad sollte ungefähr so aussehen:

```text
/ucrt64/bin/gcc
```

Unter Windows befindet sich die Datei normalerweise hier:

```text
C:\msys64\ucrt64\bin\gcc.exe
```

---

## 5. GCC mit PowerShell dauerhaft zu PATH hinzufügen

Öffne eine **neue normale PowerShell**.

Administratorrechte sind für die Änderung des benutzerspezifischen `PATH` nicht erforderlich.

Kopiere den folgenden vollständigen Code in PowerShell:

```powershell
# Installationspfad von GCC
$GccBin = "C:\msys64\ucrt64\bin"
$GccExe = Join-Path $GccBin "gcc.exe"

# Prüfen, ob GCC installiert ist
if (-not (Test-Path -LiteralPath $GccExe)) {
    throw "gcc.exe wurde nicht unter '$GccExe' gefunden. Prüfe die MSYS2-Installation."
}

# Aktuellen Benutzer-PATH lesen
$UserPath = [Environment]::GetEnvironmentVariable(
    "Path",
    [EnvironmentVariableTarget]::User
)

# Falls noch kein Benutzer-PATH existiert
if ([string]::IsNullOrWhiteSpace($UserPath)) {
    $PathEntries = @()
}
else {
    $PathEntries = @(
        $UserPath -split ";" |
        ForEach-Object { $_.Trim().TrimEnd("\") } |
        Where-Object { -not [string]::IsNullOrWhiteSpace($_) }
    )
}

# GCC-Pfad für den Vergleich normalisieren
$NormalizedGccBin = $GccBin.TrimEnd("\")

# Prüfen, ob GCC bereits im Benutzer-PATH vorhanden ist
$GccAlreadyInUserPath = $PathEntries |
    Where-Object {
        $_.Equals(
            $NormalizedGccBin,
            [StringComparison]::OrdinalIgnoreCase
        )
    }

if (-not $GccAlreadyInUserPath) {
    $NewUserPath = (($PathEntries + $NormalizedGccBin) -join ";")

    [Environment]::SetEnvironmentVariable(
        "Path",
        $NewUserPath,
        [EnvironmentVariableTarget]::User
    )

    Write-Host "GCC wurde dauerhaft zum Benutzer-PATH hinzugefügt."
}
else {
    Write-Host "GCC ist bereits im Benutzer-PATH enthalten."
}

# PATH auch in der aktuell geöffneten PowerShell aktualisieren
$CurrentPathEntries = @(
    $env:Path -split ";" |
    ForEach-Object { $_.Trim().TrimEnd("\") } |
    Where-Object { -not [string]::IsNullOrWhiteSpace($_) }
)

$GccAlreadyInCurrentPath = $CurrentPathEntries |
    Where-Object {
        $_.Equals(
            $NormalizedGccBin,
            [StringComparison]::OrdinalIgnoreCase
        )
    }

if (-not $GccAlreadyInCurrentPath) {
    $env:Path = "$env:Path;$NormalizedGccBin"
    Write-Host "GCC wurde auch zum PATH der aktuellen PowerShell hinzugefügt."
}

Write-Host ""
Write-Host "Gefundene GCC-Datei:"
Get-Item -LiteralPath $GccExe

Write-Host ""
Write-Host "GCC-Version:"
& $GccExe --version

Write-Host ""
Write-Host "Von PowerShell verwendeter GCC-Pfad:"
(Get-Command gcc -ErrorAction Stop).Source



gcc --version
where.exe gcc
go env CC
go env CGO_ENABLED
```

Der Code verwendet die von Microsoft dokumentierte Methode `SetEnvironmentVariable()`, um Umgebungsvariablen unter Windows dauerhaft zu ändern.

---

## 6. PowerShell und Programme neu starten

Schließe nach der PATH-Änderung vollständig:

* PowerShell
* Windows Terminal
* Visual Studio Code
* GoLand
* andere geöffnete Terminals oder Entwicklungsumgebungen

Öffne danach eine neue PowerShell.

Bereits laufende Programme übernehmen Änderungen an persistenten Umgebungsvariablen normalerweise nicht automatisch.

---

## 7. GCC in PowerShell prüfen

Führe in der neuen PowerShell aus:

```powershell
gcc --version
```

Prüfe anschließend, welche GCC-Datei verwendet wird:

```powershell
where.exe gcc
```

Oder mit PowerShell:

```powershell
Get-Command gcc
```

Erwarteter Pfad:

```text
C:\msys64\ucrt64\bin\gcc.exe
```

---

## 8. Go- und cgo-Konfiguration prüfen

Prüfe zunächst, ob Go installiert und erreichbar ist:

```powershell
go version
```

Prüfe den von Go verwendeten C-Compiler:

```powershell
go env CC
```

Die Ausgabe sollte normalerweise lauten:

```text
gcc
```

Prüfe, ob cgo aktiviert ist:

```powershell
go env CGO_ENABLED
```

Erwartete Ausgabe:

```text
1
```

Prüfe alle wichtigen Werte gemeinsam:

```powershell
go version
go env GOOS
go env GOARCH
go env CC
go env CGO_ENABLED
gcc --version
where.exe gcc
```

---

## 9. Go-Projekt erneut bauen

Wechsle in dein Projektverzeichnis:

```powershell
cd "C:\Pfad\zu\deinem\Projekt"
```

Lösche optional den Go-Build-Cache:

```powershell
go clean -cache
```

Lade fehlende Go-Abhängigkeiten:

```powershell
go mod tidy
```

Baue das Projekt:

```powershell
go build
```

Oder alle Pakete:

```powershell
go build ./...
```

Tests ausführen:

```powershell
go test ./...
```

---

## 10. cgo ausdrücklich aktivieren

Normalerweise ist cgo unter Windows automatisch aktiviert, wenn ein funktionierender C-Compiler verfügbar ist.

Für die aktuelle PowerShell-Sitzung:

```powershell
$env:CGO_ENABLED = "1"
go build
```

Dauerhaft für Go setzen:

```powershell
go env -w CGO_ENABLED=1
```

GCC ausdrücklich als Compiler festlegen:

```powershell
go env -w CC=gcc
```

Konfiguration prüfen:

```powershell
go env CC
go env CGO_ENABLED
```

---

## 11. Minimalen cgo-Test durchführen

Erstelle einen neuen Testordner:

```powershell
mkdir "$HOME\cgo-test"
cd "$HOME\cgo-test"
```

Initialisiere ein Go-Modul:

```powershell
go mod init cgo-test
```

Erstelle die Datei `main.go`:

```powershell
@'
package main

/*
#include <stdlib.h>
*/
import "C"

import "fmt"

func main() {
    fmt.Println("cgo und GCC funktionieren.")
}
'@ | Set-Content -Encoding UTF8 main.go
```

Programm bauen:

```powershell
go build
```

Programm starten:

```powershell
.\cgo-test.exe
```

Erwartete Ausgabe:

```text
cgo und GCC funktionieren.
```

---

# Kompakte PowerShell-Version

Dieser Block fügt ausschließlich den GCC-Ordner zum Benutzer-PATH hinzu:

```powershell
$GccBin = "C:\msys64\ucrt64\bin"

if (-not (Test-Path "$GccBin\gcc.exe")) {
    throw "gcc.exe wurde unter '$GccBin' nicht gefunden."
}

$UserPath = [Environment]::GetEnvironmentVariable("Path", "User")
$Entries = @($UserPath -split ";" | Where-Object { $_ })

if ($Entries -notcontains $GccBin) {
    $NewPath = (($Entries + $GccBin) -join ";")
    [Environment]::SetEnvironmentVariable("Path", $NewPath, "User")
}

if (($env:Path -split ";") -notcontains $GccBin) {
    $env:Path += ";$GccBin"
}

gcc --version
```

---

# Vollautomatische Prüfung

```powershell
$GccBin = "C:\msys64\ucrt64\bin"

Write-Host "=== GCC-Datei ==="
Test-Path "$GccBin\gcc.exe"

Write-Host "`n=== GCC in PATH ==="
Get-Command gcc -ErrorAction SilentlyContinue

Write-Host "`n=== GCC-Version ==="
gcc --version

Write-Host "`n=== Go-Version ==="
go version

Write-Host "`n=== Go-C-Compiler ==="
go env CC

Write-Host "`n=== cgo-Status ==="
go env CGO_ENABLED
```

---

# Fehlerbehebung

## Fehler: `gcc is not recognized`

Prüfe:

```powershell
Test-Path "C:\msys64\ucrt64\bin\gcc.exe"
```

Falls die Ausgabe `True` ist, aber `gcc` nicht gefunden wird:

```powershell
$env:Path += ";C:\msys64\ucrt64\bin"
gcc --version
```

Danach PowerShell und deine Entwicklungsumgebung neu starten.

---

## Fehler: `gcc.exe` wurde nicht gefunden

Prüfe, ob MSYS2 in einem anderen Ordner installiert wurde:

```powershell
Get-ChildItem "C:\" -Filter gcc.exe -Recurse -ErrorAction SilentlyContinue |
    Select-Object -First 20 FullName
```

Dieser Suchlauf kann etwas umfangreicher sein.

Passe anschließend den Pfad im Skript an, beispielsweise:

```powershell
$GccBin = "D:\msys64\ucrt64\bin"
```

---

## Fehler: Falsches GCC wird verwendet

Zeige alle gefundenen GCC-Dateien:

```powershell
where.exe gcc
```

Zeige die von PowerShell ausgewählte Datei:

```powershell
(Get-Command gcc).Source
```

Für Go kann der gewünschte Compiler ausdrücklich festgelegt werden:

```powershell
go env -w CC="C:\msys64\ucrt64\bin\gcc.exe"
```

Prüfen:

```powershell
go env CC
```

---

## Fehler bleibt in Visual Studio Code bestehen

1. Schließe alle VS-Code-Fenster.
2. Prüfe im Task-Manager, ob noch `Code.exe` läuft.
3. Beende verbleibende VS-Code-Prozesse.
4. Starte VS Code neu.
5. Öffne ein neues integriertes Terminal.
6. Prüfe:

```powershell
gcc --version
go env CC
go env CGO_ENABLED
```

---

## Installation kurz zusammengefasst

Im **MSYS2-UCRT64-Terminal**:

```bash
pacman -Syu
pacman -S mingw-w64-ucrt-x86_64-gcc
gcc --version
```

In einer normalen **PowerShell**:

```powershell
$GccBin = "C:\msys64\ucrt64\bin"
$UserPath = [Environment]::GetEnvironmentVariable("Path", "User")
$Entries = @($UserPath -split ";" | Where-Object { $_ })

if ($Entries -notcontains $GccBin) {
    [Environment]::SetEnvironmentVariable(
        "Path",
        (($Entries + $GccBin) -join ";"),
        "User"
    )
}

$env:Path += ";$GccBin"

gcc --version
go env CC
go env CGO_ENABLED
go build
```

https://www.msys2.org/
