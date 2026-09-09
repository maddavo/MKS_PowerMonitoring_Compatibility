# SystemHeatMKS development notes

## Why this is a separate module

The first implementation put the MKS bridge code and patches into the SystemHeat fork. That made the core plugin carry MKS-specific behavior and made it difficult to install, test, or remove the integration independently.

The integration is now split into:

- **SystemHeat core** — the general heat-loop simulation and UI, including the editor Loop ID fix.
- **SystemHeatMKS** — the MKS/USI bridge assembly and MKS-only ModuleManager patch.

This follows the way SystemHeat's optional configuration packages are structured while still allowing MKS support to contain the small amount of code required for runtime module inspection.

## Architecture

`Source/ModuleSystemHeatUSIBridges.cs` contains three `PartModule` bridges:

### MKS converters

`ModuleSystemHeatUSIConverter` locates the part's `USI_Converter` module and its linked `ModuleSystemHeat` module. It sums the converter's `ElectricCharge` input ratios and exposes the resulting heat flux to SystemHeat. The editor uses the converter's configured values for stable vessel planning; flight updates use the active converter state.

The current bridge uses a heat multiplier of `0.9` and an outlet temperature of `600 K`.

### MKS harvesters

`ModuleSystemHeatUSIHarvester` follows the same pattern for `USI_Harvester`. This supports MKS drills and other USI harvesting parts without replacing the harvester module itself.

The current bridge uses a heat multiplier of `1.0` and an outlet temperature of `350 K`.

### MKS heat pumps

`ModuleSystemHeatUSIHeatPump` locates MKS's `ModuleHeatPump`, reads its active state and `maxEnergyTransfer`, and reports active cooling as negative SystemHeat flux. Cooling is cleared when the part is disabled.

The bridge intentionally uses the MKS heat pump as the authority for whether cooling is active. SystemHeat displays and routes the flux; it does not duplicate the MKS radiator control logic.

## ModuleManager configuration

`GameData/SystemHeatMKS/Patches/MKS.cfg` targets MKS/USI part naming families and conditionally applies patches only when the relevant MKS module exists.

For converters and harvesters it:

1. Removes legacy `ModuleCoreHeat` and `ModuleOverheatDisplay` from the affected part.
2. Adds one SystemHeat loop with the MKS module ID `mks`.
3. Adds the corresponding bridge module.

For MKS heat pumps it adds a SystemHeat loop and heat-pump bridge while preserving the MKS `ModuleHeatPump`.

Removing the legacy modules is a patch to the affected MKS part configuration. It is not a source-code change to the MKS mod. The purpose is to prevent the same MKS part from being represented simultaneously by two thermal systems.

## Editor stability

SystemHeat's editor simulation can rebuild or replace module state while a vessel is being edited. The MKS bridge therefore uses editor-only updates for planning values and flight updates for live values. This avoids the rapid PAW flicker previously seen on MKS MPUs, where `System Flux` alternated between zero and the active value and caused the remaining PAW fields and controls to jump vertically.

The SystemHeat core also keeps the editor Loop ID change functional. The loop assignment must be changed through the SystemHeat simulator in both editor and flight; otherwise a changed editor value can appear correct in the vessel UI without being carried into the actual loop membership.

## Deliberate exclusions

The package does not patch:

- Stock Convert-O-Tron 125 or 250.
- Stock drills/harvesters.
- Re-entry or aerodynamic heating.
- DynamicBatteryStorage thermal behavior.

Stock converters and harvesters are owned by the official optional `SystemHeatConverters` and `SystemHeatHarvesters` packages. Keeping them out of this package avoids overlapping patches and preserves the official packages' methodology and balancing.

DynamicBatteryStorage is an electrical monitor in this project context. Its MKS compatibility work is separate from SystemHeatMKS.

## Development history

The integration evolved through these stages:

1. MKS electrical and thermal behavior was investigated using the Systems Monitor, DynamicBatteryStorage, SystemHeat, and a representative MKS converter vessel.
2. MKS converter and harvester heat was added to SystemHeat, while MKS heat pumps were connected as cooling sources.
3. Editor-only stable updates fixed the MKS PAW flicker.
4. The initial stock converter/harvester patch was removed so the official SystemHeat optional patches could handle stock parts.
5. The MKS bridge and patch were removed from SystemHeat core and moved into this standalone project.

The corresponding SystemHeat core separation commit is `6fc57ab` (`Move MKS integration out of SystemHeat core`). The standalone package began with `a93964f` (`Add standalone System Heat MKS support module`).

## Future compatibility work

Potential future work should preserve the separation above. Any change should first establish whether the behavior belongs in SystemHeat core, an official optional SystemHeat configuration package, or this MKS add-on. Particular care is needed around MKS dynamic module variants, thermal efficiency curves, and heat-pump shutdown behavior so that the bridge reports state without taking ownership of MKS simulation rules.
