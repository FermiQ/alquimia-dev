# alquimia_interface.c

## Overview

This file is responsible for creating and initializing an `AlquimiaInterface` structure. This structure contains function pointers that define the contract for how Alquimia interacts with a specific underlying geochemistry engine (like PFloTran or CrunchFlow). It acts as a factory for selecting and setting up the correct set of functions based on the requested engine name.

## Key Components

### Functions

*   `CreateAlquimiaInterface(const char* const engine_name, AlquimiaInterface* interface, AlquimiaEngineStatus* status)`: Populates an `AlquimiaInterface` structure with function pointers specific to the geochemistry engine identified by `engine_name`.
    *   **Parameters**:
        *   `engine_name` (const char* const): The name of the geochemistry engine to interface with (e.g., "PFloTran", "CrunchFlow"). This is case-insensitive.
        *   `interface` (AlquimiaInterface*): A pointer to an `AlquimiaInterface` structure that will be filled with the appropriate function pointers for the selected engine.
        *   `status` (AlquimiaEngineStatus*): A pointer to an `AlquimiaEngineStatus` structure that will be updated with the outcome of the operation (success or error).
    *   **Behavior**:
        1.  Initializes all function pointers in the `interface` structure to `NULL`.
        2.  Compares `engine_name` (case-insensitively) against known engine names (`kAlquimiaStringPFloTran`, `kAlquimiaStringCrunchFlow`).
        3.  If a match is found and Alquimia was compiled with support for that engine (checked via preprocessor macros `ALQUIMIA_HAVE_PFLOTRAN` or `ALQUIMIA_HAVE_CRUNCHFLOW`), it assigns the corresponding engine-specific functions (e.g., `pflotran_alquimia_setup`, `crunch_alquimia_setup`) to the function pointers in the `interface` structure.
        4.  Sets `status->error` to `kAlquimiaNoError` and provides a success message in `status->message`.
        5.  If the requested engine is not supported (either unknown name or not compiled in), it sets `status->error` to `kAlquimiaErrorInvalidEngine` and provides an appropriate error message in `status->message`.

## Important Variables/Constants

This file primarily uses constants defined elsewhere (see `alquimia_constants.c`) and preprocessor macros to determine engine availability.

*   `kAlquimiaStringPFloTran` (from `alquimia_constants.h`): String constant for "PFloTran".
*   `kAlquimiaStringCrunchFlow` (from `alquimia_constants.h`): String constant for "CrunchFlow".
*   `kAlquimiaNoError` (from `alquimia_constants.h`): Error code for success.
*   `kAlquimiaErrorInvalidEngine` (from `alquimia_constants.h`): Error code for invalid engine.
*   `ALQUIMIA_HAVE_PFLOTRAN` (preprocessor macro): Defined if Alquimia is compiled with PFloTran support.
*   `ALQUIMIA_HAVE_CRUNCHFLOW` (preprocessor macro): Defined if Alquimia is compiled with CrunchFlow support.

## Usage Examples

```c
#include "alquimia/alquimia_interface.h"
#include "alquimia/alquimia_constants.h" // For engine names and status
#include <stdio.h>

int main() {
    AlquimiaInterface chem_interface;
    AlquimiaEngineStatus status;

    // Attempt to create an interface for PFloTran
    CreateAlquimiaInterface(kAlquimiaStringPFloTran, &chem_interface, &status);

    if (status.error == kAlquimiaNoError) {
        printf("Successfully created interface for PFloTran.\n");
        // Now chem_interface.Setup, chem_interface.Shutdown, etc., can be used.
        if (chem_interface.Setup != NULL) {
            printf("Setup function is available.\n");
            // Example: chem_interface.Setup(...);
        }
    } else {
        fprintf(stderr, "Failed to create PFloTran interface: %s\n", status.message);
    }

    // Attempt to create an interface for an unsupported engine
    CreateAlquimiaInterface("NonExistentEngine", &chem_interface, &status);
    if (status.error != kAlquimiaNoError) {
        fprintf(stderr, "Failed as expected for NonExistentEngine: %s\n", status.message);
    }

    return 0;
}
```

## Dependencies and Interactions

*   **Internal Dependencies**:
    *   `alquimia/alquimia_interface.h`: Declares the `AlquimiaInterface` structure and `AlquimiaEngineStatus` structure, and the `CreateAlquimiaInterface` function itself.
    *   `alquimia/pflotran_alquimia_interface.h`: Declares the PFloTran-specific functions (e.g., `pflotran_alquimia_setup`) that are assigned to the interface pointers if PFloTran is selected.
    *   `alquimia/crunch_alquimia_interface.h`: Declares the CrunchFlow-specific functions (e.g., `crunch_alquimia_setup`) that are assigned to the interface pointers if CrunchFlow is selected.
    *   `alquimia/alquimia_util.h`: Provides `AlquimiaCaseInsensitiveStringCompare` for comparing engine names.
    *   `alquimia/alquimia_constants.h`: Provides string constants for engine names (`kAlquimiaStringPFloTran`, `kAlquimiaStringCrunchFlow`) and error codes (`kAlquimiaNoError`, `kAlquimiaErrorInvalidEngine`).
*   **External Libraries**:
    *   `<stdio.h>` (implicitly, for `snprintf` used to format status messages).
*   **Interactions with Other Components**:
    *   This file is a critical entry point for applications using Alquimia. The `CreateAlquimiaInterface` function is typically one of the first Alquimia functions called by a driver or simulation code.
    *   The populated `AlquimiaInterface` structure is then used by the driver code to call the appropriate engine-specific functions for setup, performing reactions, retrieving data, and shutting down the geochemistry engine.
    *   The selection mechanism (based on `engine_name` and compile-time flags) allows Alquimia to be flexible and support multiple geochemistry backends without requiring the driver code to be aware of the specific engine details beyond the initial setup.
```
