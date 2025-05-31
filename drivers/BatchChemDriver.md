# BatchChemDriver.c

## Overview

`BatchChemDriver.c` implements a driver for running batch chemistry simulations using the Alquimia library. It reads simulation parameters, initial conditions, and chemical properties from an input file, initializes a geochemistry engine via Alquimia, runs the simulation over a specified time or number of steps, and collects results. It also provides functionality to retrieve the history of solute concentrations and auxiliary data.

## Key Components

### Main Structures

*   `BatchChemDriverInput`: Stores input parameters read from the configuration file.
    *   Key fields: `description`, `t_min`, `t_max`, `max_steps`, `dt` (timestep), `volume`, `saturation`, `water_density`, `porosity`, `temperature`, `aqueous_pressure`, `chemistry_engine`, `chemistry_input_file`, `cond_name` (initial condition name), `output_type`, `output_file`, arrays for isotherm/ion exchange/surface site parameters.
*   `struct BatchChemDriver`: The main driver structure holding all simulation state, parameters, Alquimia interface objects, and data history.
    *   Key fields: `description`, `t_min`, `t_max`, `dt`, `max_steps`, `verbose`, state variables (`water_density`, `porosity`, etc.), material properties (`volume`, `saturation`), current simulation time and step, Alquimia data structures (`chem_properties`, `chem_state`, `chem_aux_data`, `chem_aux_output`, `chem_sizes`, `chem_metadata`, `chem_cond`), Alquimia interface (`chem`, `chem_engine`, `chem_status`), and history arrays (`history_times`, `chem_state_history`, `chem_aux_output_history`).

### Input Parsing Functions and Macros

*   `ValueIsTrue(const char* value)`: (static) Helper to parse boolean-like strings ("1", "true", "yes", "on").
*   `GetIndexLabel(const char* name, char* label)`: (static) Extracts an index label (e.g., "species_A") from a string like "param[species_A]".
*   `DECLARE_INDEXED_VALUE_PARSER` (macro): Generates parser functions (`IndexNameIndex` and `GetIndexNameValue`) for handling input parameters that are indexed by names (e.g., `isotherm_kd[species_name]`).
    *   `IsothermIndex`, `GetIsothermValue`
    *   `IonExchangeIndex`, `GetIonExchangeValue`
    *   `SurfaceSiteIndex`, `GetSurfaceSiteValue`
*   `ParseInput(void* user, const char* section, const char* name, const char* value)`: (static) Callback function for the `ini_parse` library. Parses key-value pairs from the input INI file and populates the `BatchChemDriverInput` structure.

### Core Driver Functions

*   `BatchChemDriverInput_New(const char* input_file)`: Allocates and initializes a `BatchChemDriverInput` structure by parsing the specified input file. Sets default values and performs basic validation.
*   `BatchChemDriverInput_Free(BatchChemDriverInput* input)`: Frees memory allocated for a `BatchChemDriverInput` structure and its members.
*   `BatchChemDriver_New(BatchChemDriverInput* input)`: Allocates and initializes the main `BatchChemDriver` structure.
    *   Sets up the Alquimia interface (`CreateAlquimiaInterface`).
    *   Initializes the chosen geochemistry engine (`driver->chem.Setup`).
    *   Allocates memory for Alquimia data structures (`AlquimiaState`, `AlquimiaProperties`, etc.).
    *   Retrieves problem metadata from the engine.
    *   Sets up the initial geochemical condition.
    *   Copies relevant parameters from `BatchChemDriverInput` to the driver and Alquimia structures.
*   `BatchChemDriver_Free(BatchChemDriver* driver)`: Frees all resources associated with the `BatchChemDriver`, including Alquimia data structures, history arrays, and shuts down the chemistry engine.
*   `BatchChemDriver_RecordHistory(BatchChemDriver* driver)`: (static) Appends the current simulation time, `AlquimiaState`, and `AlquimiaAuxiliaryOutputData` to the history arrays. Handles reallocation of history arrays if capacity is exceeded.
*   `BatchChemDriver_Initialize(BatchChemDriver* driver)`: (static) Initializes the chemical state of the simulation.
    *   Sets material properties and thermodynamic state in `driver->chem_properties` and `driver->chem_state`.
    *   Invokes the `ProcessCondition` Alquimia function to establish the initial chemical state.
    *   Records the initial state in the history.
*   `BatchChemDriver_Run(BatchChemDriver* driver)`: Executes the batch chemistry simulation loop.
    *   Calls `BatchChemDriver_Initialize`.
    *   Loops while `driver->time < driver->t_max` and `driver->step < driver->max_steps`.
    *   In each step:
        *   Calls `driver->chem.ReactionStepOperatorSplit` to advance the chemical reactions by `dt`.
        *   Calls `driver->chem.GetAuxiliaryOutput` to retrieve auxiliary data (like pH).
        *   Updates `driver->time` and `driver->step`.
        *   Calls `BatchChemDriver_RecordHistory`.
*   `BatchChemDriver_GetSoluteAndAuxData(BatchChemDriver* driver, double* time, AlquimiaVectorString* var_names, AlquimiaVectorDouble* var_data)`: Retrieves the simulation history.
    *   Populates `var_names` with names of all tracked variables (time, total mobile concentrations, mineral volume fractions, pH, etc.).
    *   Populates `var_data` with the corresponding data from the history arrays for each time step. The data is flattened into a 1D array where columns are variables and rows are time steps.

## Important Variables/Constants

*   `BATCH_CHEM_INPUT_MAX` (from `BatchChemDriver.h`): Maximum number of entries for indexed input arrays (e.g., max isotherm species).
*   Inline helper functions `Min(double a, double b)` and `Max(double a, double b)`.

## Usage Examples

The `BatchChemDriver` is typically compiled into an executable that takes an input INI file as a command-line argument.

**Conceptual main function (not in this file, but shows usage):**
```c
#include "BatchChemDriver.h"
#include "DriverOutput.h" // For writing output
#include <stdio.h>

int main(int argc, char* argv[]) {
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <input_file.ini>\n", argv[0]);
        return 1;
    }

    BatchChemDriverInput* input = BatchChemDriverInput_New(argv[1]);
    if (!input) {
        // Error already printed by Alquimia
        return 1;
    }

    BatchChemDriver* driver = BatchChemDriver_New(input);
    if (!driver) {
        BatchChemDriverInput_Free(input);
        return 1;
    }

    int status = BatchChemDriver_Run(driver);
    if (status != 0) {
        fprintf(stderr, "BatchChemDriver_Run failed.\n");
    }

    // Example: Retrieve and print some data (actual output handled by DriverOutput)
    AlquimiaVectorString var_names;
    AlquimiaVectorDouble var_data;
    double time_dummy; // The time array itself is part of var_data

    AllocateAlquimiaVectorString(0, &var_names); // Initialize empty
    AllocateAlquimiaVectorDouble(0, &var_data);  // Initialize empty

    BatchChemDriver_GetSoluteAndAuxData(driver, &time_dummy, &var_names, &var_data);

    printf("Retrieved %d variables over %d time steps.\n",
           var_names.size,
           (var_names.size > 0 ? var_data.size / var_names.size : 0) );

    // Outputting data would typically use functions from DriverOutput.c
    // Example: WriteGnuplotOutput(driver, input->output_file);

    FreeAlquimiaVectorString(&var_names);
    FreeAlquimiaVectorDouble(&var_data);

    BatchChemDriver_Free(driver);
    BatchChemDriverInput_Free(input);

    return status;
}
```

## Dependencies and Interactions

*   **Internal Dependencies**:
    *   `BatchChemDriver.h`: Header file for this driver, defines `BatchChemDriverInput` and `struct BatchChemDriver`, and function prototypes.
    *   `alquimia/alquimia_interface.h`: For `AlquimiaInterface`, `AlquimiaEngineStatus`, `CreateAlquimiaInterface`, and Alquimia function pointer types.
    *   `alquimia/alquimia_memory.h`: For Alquimia data structure allocation/deallocation functions (`AllocateAlquimiaState`, `FreeAlquimiaProblemMetaData`, etc.) and `AlquimiaStringDup`.
    *   `alquimia/alquimia_util.h`: For `AlquimiaCaseInsensitiveStringCompare`, `CopyAlquimiaState`, `CopyAlquimiaAuxiliaryOutputData`.
    *   `ini.h`: For the `ini_parse` function used to read INI configuration files.
*   **External Libraries**:
    *   `<limits.h>`, `<float.h>`: For `INT_MAX`, `FLT_MAX`.
    *   `<stdio.h>`: For `printf`, `snprintf`, `memcpy`, `strcat`, `strlen`. (Some of these might be via included Alquimia headers).
    *   `<stdlib.h>`: For `malloc`, `free`, `atof`, `atoi`.
    *   `<string.h>`: For `strcmp`, `strncmp`, `strstr`, `memset`.
*   **Interactions with Other Components**:
    *   **Alquimia Library**: This driver is a client of the Alquimia library. It uses Alquimia's interface to set up and run geochemical simulations.
    *   **Geochemistry Engines (PFLOTRAN, CrunchFlow, etc.)**: Interacts indirectly with these engines through the Alquimia API. The specific engine used is determined by the input file.
    *   `DriverOutput.c`: (Not directly called in this file but designed to be used with it) `BatchChemDriver_GetSoluteAndAuxData` provides data that can be formatted and written to files by functions in `DriverOutput.c`.
    *   `batch_chem.c` (executable's main): The `main` function in `batch_chem.c` would typically create and run an instance of `BatchChemDriver`.
```
