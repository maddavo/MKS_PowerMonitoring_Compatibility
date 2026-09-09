# Changelog

## 0.1.0.0

Initial standalone MKS support package.

- Added `SystemHeatMKS.dll` with bridges for MKS/USI converters, harvesters, and heat pumps.
- Added MKS-only SystemHeat loop configuration.
- Preserved the underlying MKS converter, harvester, drillhead, and heat-pump modules.
- Removed duplicate legacy `ModuleCoreHeat` and `ModuleOverheatDisplay` handling from affected MKS converter and harvester parts through configuration patches.
- Kept stock Convert-O-Trons and stock drills outside this package so the official `SystemHeatConverters` and `SystemHeatHarvesters` patches can handle them.
- Moved MKS integration out of SystemHeat core so the core plugin remains general-purpose.
- Documented editor/flight behavior and the known boundary with stock re-entry heating.
