# SystemHeatMKS

SystemHeatMKS is an optional add-on for [SystemHeat](https://github.com/KSPModStewards/SystemHeat) that adds SystemHeat reporting and loop integration for supported MKS/USI parts.

It is deliberately separate from SystemHeat core. SystemHeat remains responsible for the heat-loop simulation and user interface; this package supplies the MKS-specific bridge code and ModuleManager configuration.

## Scope

The current package covers MKS parts that use the following USI modules:

- `USI_Converter` — MKS converters, including Material Processing Units.
- `USI_Harvester` — MKS drills and other USI harvesters.
- `ModuleHeatPump` — MKS thermal control systems and active radiators.

On affected MKS parts, the package removes the duplicate legacy `ModuleCoreHeat`/`ModuleOverheatDisplay` handling and adds a SystemHeat loop. The original MKS modules remain in place and continue to control conversion, harvesting, and heat-pump operation.

Stock Convert-O-Trons and stock drills are intentionally not patched here. They are left to the official optional `SystemHeatConverters` and `SystemHeatHarvesters` packages so that there is one owner for those parts and no duplicate heat accounting.

## Requirements

- Kerbal Space Program 1.11.0–1.12.x; developed against 1.12.5.
- MKS/USI parts and their normal dependencies.
- SystemHeat core from the official [KSPModStewards/SystemHeat](https://github.com/KSPModStewards/SystemHeat) repository.
- ModuleManager.

The add-on is optional: SystemHeat can run without it, and MKS can run without SystemHeatMKS using its normal thermal modules.

## Installation

Install the contents of `GameData/SystemHeatMKS` into the KSP `GameData` directory alongside the SystemHeat core directory:

```text
KSP/
└── GameData/
    ├── SystemHeat/
    │   └── Plugin/SystemHeat.dll
    └── SystemHeatMKS/
        ├── Plugin/SystemHeatMKS.dll
        ├── Patches/MKS.cfg
        └── Versioning/SystemHeatMKS.version
```

Do not place `SystemHeatMKS.dll` in the `SystemHeat/Plugin` directory. Keeping the assemblies separate makes the add-on removable and prevents MKS-specific code from becoming a core dependency.

## Building

The project expects the KSP installation at the path configured in `Source/SystemHeatMKS.csproj` and expects a Release build of the sibling SystemHeat project.

1. Build `SystemHeat` in Release configuration.
2. Build `Source/SystemHeatMKS.csproj` in Release configuration.
3. The post-build target copies the DLL and MKS patch into the local `GameData/SystemHeatMKS` staging directory.
4. Include the version file when packaging a release.

Generated `bin`, `obj`, and staged plugin output are ignored by Git. Source and configuration are tracked; release binaries should be produced from the matching source revisions.

## Testing checklist

In the KSP editor and in flight:

1. Load an MKS vessel containing at least one converter, harvester, and heat pump/radiator.
2. Open SystemHeat Heat Management and confirm MKS converter/harvester heat appears under Generated.
3. Enable a thermal control system and confirm its cooling appears under Removed as a negative flux.
4. Change the editor Loop ID and confirm the part and its SystemHeat module move to the selected loop without a flickering PAW.
5. Confirm stock Convert-O-Trons and stock drills are handled only when the corresponding official SystemHeat optional patches are installed.
6. Check `KSP.log` and `ModuleManager.ConfigCache` for patch errors after a clean load.

The separated package has been built and installed. Runtime validation of the separated packaging should be performed in KSP with the matching SystemHeat core and MKS installation.

## Known limitations

The current bridge reports the MKS module's configured electrical/thermal behavior to SystemHeat; it does not replace MKS's internal simulation. In particular, it does not yet model every MKS efficiency or temperature modifier curve, and it does not make SystemHeat responsible for MKS thermal throttling or shutdown decisions.

Re-entry heating remains stock KSP thermal behavior and is not routed through SystemHeat loops. SystemHeatMKS also requires a compatible SystemHeat core assembly; the two projects should be updated and packaged together.

See [DEVELOPMENT.md](DEVELOPMENT.md) for the implementation details and development history.
