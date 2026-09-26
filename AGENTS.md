# AGENTS.md — Tunetastic (fork MikaKrul)

## Build-beleid (verplicht voor elke agent)

- **Platform: alleen Windows x64.** ARM64 en andere architecturen worden niet
  gebouwd. Geen matrix-builds toevoegen.
- **Geen signing.** `AppxPackageSigningEnabled` blijft `False`. Nooit een
  certificaat, thumbprint of timestamp-server toevoegen.
- **Geen secrets.** Workflows mogen alleen `GITHUB_TOKEN`-loze of default
  rechten (`contents: read`) gebruiken. Nooit een repository secret vereisen
  (zoals `VERSION_BUMP_PAT`).
- **Geen update-mechanismen.** Geen auto-updater, versie-sync of store-logica
  toevoegen aan builds of workflows.
- **Eén knop-workflow:** `.github/workflows/build-exe.yml` (handmatig via
  Actions > "Build EXE (x64)" > "Run workflow"). Geen nieuwe workflow-bestanden
  aanmaken zonder expliciete vraag.

## Lokaal bouwen (unsigned, unpackaged, x64)

```powershell
dotnet restore Tunetastic.csproj --configfile nuget.config -p:Platform=x64 -p:RuntimeIdentifier=win-x64 -p:SelfContained=true -p:WindowsPackageType=None -p:AppxPackageSigningEnabled=false
dotnet publish Tunetastic.csproj --no-restore -c Release -p:Platform=x64 -p:RuntimeIdentifier=win-x64 -p:SelfContained=true -p:WindowsAppSDKSelfContained=true -p:WindowsPackageType=None -p:AppxPackage=false -p:GenerateAppxPackageOnBuild=false -p:AppxPackageSigningEnabled=false -p:CreateMSIXPackage=false -o .\publish
```

Resultaat: `.\publish\Tunetastic.exe` (uitpakken en starten, geen installatie).

Vereisten: Windows 10/11, .NET 9 SDK, workloads **.NET desktop development**
en **Windows application development**.

## GitHub Actions babysitten

```powershell
gh workflow run "Build EXE (x64)" --repo MikaKrul/Tunetastic --ref <branch>
gh run watch <run-id> --repo MikaKrul/Tunetastic
```

- Blijf in een loop controleren tot de run groen is.
- Bij falen: log ophalen (`gh run view <id> --log-failed`), oorzaak fixen,
  pushen en opnieuw triggeren.
- Respecteer altijd het build-beleid hierboven bij fixes.
