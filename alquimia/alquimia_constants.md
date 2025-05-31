# alquimia_constants.c

## Overview

This file defines various global constants used throughout the Alquimia library. These include string length limits, identifiers for geochemistry engines, names for common geochemical quantities, and error codes.

## Key Components

This file does not define functions or classes. Its primary role is to provide globally accessible constant values.

## Important Variables/Constants

### String Lengths

*   `kAlquimiaMaxStringLength` (const int): Defines the maximum length for general strings within Alquimia (512 characters). This is crucial for buffer allocations and string manipulations to prevent overflows.
*   `kAlquimiaMaxWordLength` (const int): Defines the maximum length for shorter string tokens or words (32 characters).

### Geochemistry Engine Strings

These constants are used to identify different geochemistry simulation engines that Alquimia can interface with.

*   `kAlquimiaStringPFloTran` (const char*): String identifier for the PFloTran engine ("PFloTran").
*   `kAlquimiaStringCrunchFlow` (const char*): String identifier for the CrunchFlow engine ("CrunchFlow").

### Geochemical Quantity Strings

These constants define standardized names for various geochemical properties or quantities. This promotes consistency when referring to these values across different parts of the code or in input/output files.

*   `kAlquimiaStringTotal` (const char*): Identifier for total aqueous concentration ("total_aqueous").
*   `kAlquimiaStringTotalSorbed` (const char*): Identifier for total sorbed concentration ("total_sorbed").
*   `kAlquimiaStringTotalAqueousPlusSorbed` (const char*): Identifier for the sum of total aqueous and total sorbed concentrations ("total_aqueous_plus_sorbed").
*   `kAlquimiaStringFree` (const char*): Identifier for free ion concentration ("free").
*   `kAlquimiaStringPH` (const char*): Identifier for pH ("pH").
*   `kAlquimiaStringMineral` (const char*): Identifier for mineral phase related quantities ("mineral").
*   `kAlquimiaStringGas` (const char*): Identifier for gas phase related quantities ("gas").
*   `kAlquimiaStringCharge` (const char*): Identifier for electrical charge ("charge").
*   `kAlquimiaStringTotalGaseous` (const char*): Identifier for total gaseous concentration ("total_gaseous").

### Error Codes

These integer constants represent specific error conditions that can occur within Alquimia.

*   `kAlquimiaNoError` (const int): Represents no error (0).
*   `kAlquimiaErrorInvalidEngine` (const int): Error code for an invalid or unrecognized geochemistry engine (1).
*   `kAlquimiaErrorUnknownConstraintName` (const int): Error code for an unrecognized constraint name (2).
*   `kAlquimiaErrorUnsupportedFunctionality` (const int): Error code indicating that a requested functionality is not supported by the current engine or configuration (3).
*   `kAlquimiaErrorEngineIntegrity` (const int): Error code indicating an internal error or integrity issue within the geochemistry engine (4577).

## Usage Examples

```c
#include "alquimia/alquimia_constants.h"
#include <stdio.h>
#include <string.h>

void check_engine_type(const char* engine_name) {
    if (strcmp(engine_name, kAlquimiaStringPFloTran) == 0) {
        printf("Engine is PFloTran.\n");
    } else if (strcmp(engine_name, kAlquimiaStringCrunchFlow) == 0) {
        printf("Engine is CrunchFlow.\n");
    } else {
        printf("Unknown engine.\n");
    }
}

int get_simulation_status() {
    // ... some operation ...
    // if (/* some error condition */) {
    //     return kAlquimiaErrorInvalidEngine;
    // }
    return kAlquimiaNoError;
}

int main() {
    char species_name[kAlquimiaMaxStringLength];
    strncpy(species_name, "H+", kAlquimiaMaxStringLength -1);
    species_name[kAlquimiaMaxStringLength -1] = '\0';

    printf("Max string length: %d\n", kAlquimiaMaxStringLength);
    printf("pH identifier: %s\n", kAlquimiaStringPH);

    check_engine_type(kAlquimiaStringCrunchFlow);

    int status = get_simulation_status();
    if (status == kAlquimiaNoError) {
        printf("Simulation successful.\n");
    } else if (status == kAlquimiaErrorInvalidEngine) {
        printf("Error: Invalid engine specified.\n");
    }

    return 0;
}
```

## Dependencies and Interactions

*   **Internal Dependencies**:
    *   `alquimia/alquimia_constants.h`: This is the header file that declares these constants, making them available to other parts of the Alquimia library and to client code.
*   **External Libraries**:
    *   None directly in this file, but the constants defined here are used in conjunction with standard C library functions like `strcmp` or `strncpy` in other files.
*   **Interactions with Other Components**:
    *   These constants are widely used across the Alquimia codebase, particularly in:
        *   Parsing input files or commands that specify engine types or geochemical parameters.
        *   Setting up and querying data structures that store geochemical information.
        *   Reporting errors and status conditions.
        *   Interfacing with different geochemistry engines, where these string identifiers might be used to map Alquimia's internal representations to engine-specific ones.
    *   The string length constants are critical for safe memory allocation and string handling operations everywhere.
    *   Error codes provide a standardized way for functions to communicate success or failure, allowing calling code to react appropriately.
```
