# AGENTS.md — Tunetastic (fork MikaKrul)

## Build-beleid (verplicht voor elke agent)

- **Platform: alleen Windows x64.** ARM64 en andere architecturen worden niet
  gebouwd. Geen matrix-builds toevoegen.
- **MSIX-packaging met signing.** Het pakket moet als update over de originele
  Tunetastic-installatie heen kunnen installeren: Identity-Naam
  (`AMit-KP.Tunetastic`) en Publisher-CN (`CN=B90A5939-...`) in
  `Package.appxmanifest` nooit wijzigen, en de manifest-versie altijd hoger
  zetten dan de laatst uitgebrachte versie. Signing loopt via de secrets
  `TUNETASTIC_SIGN_PFX` (base64) en `TUNETASTIC_SIGN_PASSWORD`; de publieke
  `.cer` staat in `Assets/Store/`. Nooit een PFX of wachtwoord in de repo zetten.
- **Geen andere secrets.** Buiten de signing-secrets geen repository secrets
  vereisen (zoals `VERSION_BUMP_PAT`).
- **Geen update-mechanismen.** Geen auto-updater, versie-sync of store-logica
  toevoegen aan builds of workflows.
- **Eén knop-workflow:** `.github/workflows/build-msix.yml` (handmatig via
  Actions > "Build MSIX (x64)" > "Run workflow"). Geen nieuwe workflow-bestanden
  aanmaken zonder expliciete vraag.

## Lokaal bouwen (unsigned, unpackaged, x64 — alleen om te testen)

De CI bouwt het signed MSIX-pakket. Lokaal bouwen kan alleen unsigned
(niet te installeren, wel te testen):

```powershell
dotnet restore Tunetastic.csproj --configfile nuget.config -p:Platform=x64 -p:RuntimeIdentifier=win-x64 -p:SelfContained=true -p:WindowsPackageType=None -p:AppxPackageSigningEnabled=false
dotnet publish Tunetastic.csproj --no-restore -c Release -p:Platform=x64 -p:RuntimeIdentifier=win-x64 -p:SelfContained=true -p:WindowsAppSDKSelfContained=true -p:WindowsPackageType=None -p:AppxPackage=false -p:GenerateAppxPackageOnBuild=false -p:AppxPackageSigningEnabled=false -p:CreateMSIXPackage=false -o .\publish
```

Resultaat: `.\publish\Tunetastic.exe` (uitpakken en starten, geen installatie).

Vereisten: Windows 10/11, .NET 9 SDK, workloads **.NET desktop development**
en **Windows application development**.

## GitHub Actions babysitten

```powershell
gh workflow run "Build MSIX (x64)" --repo MikaKrul/Tunetastic --ref <branch>
gh run watch <run-id> --repo MikaKrul/Tunetastic
```

- Blijf in een loop controleren tot de run groen is.
- Bij falen: log ophalen (`gh run view <id> --log-failed`), oorzaak fixen,
  pushen en opnieuw triggeren.
- Respecteer altijd het build-beleid hierboven bij fixes.
