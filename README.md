# MKS Power Monitoring Compatibility

Module Manager compatibility patches for using MKS power-consuming converters
with vessel power-planning tools.

## Current patch

`zzz_MKS_DynamicBatteryStorage.cfg` adds MKS `USI_Converter` modules to
Dynamic Battery Storage's Systems Monitor. It reuses the monitor's existing
stock `ModuleResourceConverterPowerHandler`, so MKS converters are read from
their selected recipe and included in the SPH/VAB power summary.

The patch is deliberately standalone and does not overwrite MKS or Dynamic
Battery Storage files.

## Installation

Copy the repository's `GameData/MKS_PowerMonitoring_Compatibility` folder into
the KSP installation's `GameData` folder. The patch requires MKS, USITools, and
Dynamic Battery Storage; without those mods it does nothing.

## Scope

This first patch targets Systems Monitor/Dynamic Battery Storage. AmpYear uses
a separate implementation and will be handled only after its installed
behaviour has been verified.
