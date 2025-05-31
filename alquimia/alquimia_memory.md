# alquimia_memory.c

## Overview

This file provides a suite of memory management utilities specifically tailored for Alquimia's data structures. It includes functions for allocating and freeing memory for various vector types (`AlquimiaVectorDouble`, `AlquimiaVectorInt`, `AlquimiaVectorString`) and complex Alquimia data structures like `AlquimiaState`, `AlquimiaProperties`, `AlquimiaProblemMetaData`, `AlquimiaAuxiliaryData`, `AlquimiaAuxiliaryOutputData`, `AlquimiaEngineStatus`, `AlquimiaGeochemicalCondition`, and the encompassing `AlquimiaData` struct. The allocation strategy for vectors often involves allocating a capacity that is the nearest power of 2 greater than or equal to the requested size, potentially for optimized resizing or memory alignment, though resizing logic isn't present in this file.

## Key Components

### Static Helper Functions

*   `nearest_power_of_2(int n)`: Calculates the smallest power of 2 that is greater than or equal to `n`. Returns 0 if `n` is 0. This is used to determine the initial capacity for vector allocations.

### Vector Allocation/Deallocation Functions

For each vector type (`Double`, `Int`, `String`):

*   `AllocateAlquimiaVector[Type](const int size, AlquimiaVector[Type]* vector)`:
    *   Allocates memory for an `AlquimiaVector[Type]`.
    *   Sets `vector->size` to `size`.
    *   Sets `vector->capacity` to `nearest_power_of_2(size)`.
    *   Allocates `vector->data` using `calloc` with the calculated capacity.
    *   For `AlquimiaVectorString`, it additionally allocates memory for each individual string in the `data` array using `calloc` with `kAlquimiaMaxStringLength`.
    *   Uses `ALQUIMIA_ASSERT` to check for allocation failures.
    *   If `size` is 0, sets members to 0 or `NULL`.
*   `FreeAlquimiaVector[Type](AlquimiaVector[Type]* vector)`:
    *   Frees the memory allocated for `vector->data`.
    *   For `AlquimiaVectorString`, it first iterates through and frees each individual string.
    *   Sets `vector->data` to `NULL` and `vector->size`, `vector->capacity` to 0.

### Alquimia Structure Allocation/Deallocation Functions

For each major Alquimia data structure (e.g., `AlquimiaState`, `AlquimiaProperties`, etc.):

*   `Allocate[StructureName](const AlquimiaSizes* const sizes, [StructureName]* structure)` (or similar parameters like `const int size_name` for `AlquimiaGeochemicalCondition`):
    *   Allocates memory for the members of the given `structure`. This typically involves calling the `AllocateAlquimiaVector[Type]` functions for vector members, using dimensions specified in the `sizes` object or other parameters.
    *   For `AlquimiaEngineStatus`, allocates memory for the `message` string.
    *   For `AlquimiaGeochemicalCondition` and its sub-structures, allocates memory for names and constraint lists.
    *   Initializes some scalar members to default values (e.g., `aux_output->pH = -999.9`).
*   `Free[StructureName]([StructureName]* structure)`:
    *   Frees memory for all dynamically allocated members of the `structure`. This typically involves calling `FreeAlquimiaVector[Type]` for vector members or `free` for simple pointers.
    *   Sets freed pointers to `NULL`.

### Composite Structure Allocation/Deallocation

*   `AllocateAlquimiaData(AlquimiaData* data)`: A convenience function that allocates all substructures within an `AlquimiaData` object (`state`, `properties`, `aux_data`, `meta_data`, `aux_output`) by calling their respective `Allocate` functions. The `data->sizes` member is assumed to be pre-populated.
*   `FreeAlquimiaData(AlquimiaData* data)`: A convenience function that frees all substructures within an `AlquimiaData` object.

## Important Variables/Constants

*   `kAlquimiaMaxStringLength` (from `alquimia_constants.h`): Used in `AllocateAlquimiaVectorString` and `AllocateAlquimiaAqueousConstraint`, `AllocateAlquimiaMineralConstraint`, `AllocateAlquimiaEngineStatus` to determine buffer sizes for strings.
*   `ALQUIMIA_ASSERT` (macro, likely defined in `alquimia_interface.h` or a similar core header): Used for basic assertion checking, particularly to ensure `calloc` does not return `NULL`.

## Usage Examples

```c
#include "alquimia/alquimia_memory.h"
#include "alquimia/alquimia_constants.h" // For kAlquimiaMaxStringLength
#include <stdio.h>
#include <string.h> // For strcpy

int main() {
    AlquimiaSizes sizes;
    sizes.num_primary = 5;
    sizes.num_minerals = 2;
    sizes.num_sorbed = 0;
    sizes.num_surface_sites = 1;
    sizes.num_ion_exchange_sites = 1;
    sizes.num_isotherm_species = 0;
    sizes.num_aqueous_kinetics = 0;
    sizes.num_gases = 0;
    sizes.num_aux_integers = 0;
    sizes.num_aux_doubles = 0;
    sizes.num_aqueous_complexes = 0; // Assuming this is part of sizes for aux_output

    AlquimiaState state;
    AllocateAlquimiaState(&sizes, &state);
    printf("Allocated AlquimiaState. total_mobile size: %d, capacity: %d\n",
           state.total_mobile.size, state.total_mobile.capacity);
    if (state.total_mobile.size > 0) {
        state.total_mobile.data[0] = 1.23;
        printf("state.total_mobile.data[0] = %f\n", state.total_mobile.data[0]);
    }

    AlquimiaProblemMetaData meta_data;
    AllocateAlquimiaProblemMetaData(&sizes, &meta_data);
    if (meta_data.primary_names.size > 0) {
        strncpy(meta_data.primary_names.data[0], "H+", kAlquimiaMaxStringLength -1);
         meta_data.primary_names.data[0][kAlquimiaMaxStringLength-1] = '\0';
        printf("meta_data.primary_names.data[0] = %s\n", meta_data.primary_names.data[0]);
    }


    AlquimiaData comprehensive_data;
    comprehensive_data.sizes = sizes; // Copy sizes information
    AllocateAlquimiaData(&comprehensive_data);
    printf("Allocated AlquimiaData. pH in aux_output: %f\n", comprehensive_data.aux_output.pH);


    // Freeing memory
    FreeAlquimiaState(&state);
    printf("Freed AlquimiaState. total_mobile.data is now %p\n", (void*)state.total_mobile.data);

    FreeAlquimiaProblemMetaData(&meta_data);
    FreeAlquimiaData(&comprehensive_data);
    printf("Freed AlquimiaData.\n");

    AlquimiaEngineStatus engine_status;
    AllocateAlquimiaEngineStatus(&engine_status);
    strncpy(engine_status.message, "All good.", kAlquimiaMaxStringLength -1);
    engine_status.message[kAlquimiaMaxStringLength-1] = '\0';
    printf("Engine status: %s\n", engine_status.message);
    FreeAlquimiaEngineStatus(&engine_status);

    return 0;
}
```

## Dependencies and Interactions

*   **Internal Dependencies**:
    *   `alquimia/alquimia_memory.h`: Declares all the allocation/deallocation functions and the structures they operate on.
    *   `alquimia/alquimia_interface.h`: Potentially for `ALQUIMIA_ASSERT` macro (though its exact definition location might vary). It also declares `AlquimiaEngineStatus`.
    *   `alquimia/alquimia_constants.h`: Provides `kAlquimiaMaxStringLength`.
    *   `alquimia/alquimia_containers.h`: Defines the basic vector structures (`AlquimiaVectorDouble`, etc.) and the more complex Alquimia data containers (`AlquimiaState`, `AlquimiaSizes`, etc.).
*   **External Libraries**:
    *   `<stdlib.h>`: For `calloc` and `free`.
    *   `<stdio.h>`: Used in `AllocateAlquimiaGeochemicalConditionVector` for `fprintf` (debug/logging purposes, might be removed in production).
    *   `<string.h>`: Used in `AllocateAlquimiaProblemMetaData` for `memset`.
*   **Interactions with Other Components**:
    *   This file is fundamental to the operation of Alquimia. Any part of Alquimia that creates or uses its primary data structures will rely on these memory management functions.
    *   Driver codes and geochemistry engine interface layers use these functions to set up the necessary data structures before calling engine-specific computation functions and to clean up memory afterward.
    *   The `AlquimiaSizes` structure plays a crucial role, as it dictates the dimensions of the arrays allocated for various properties within other structures like `AlquimiaState`, `AlquimiaProblemMetaData`, etc. It must be correctly populated before calling allocation functions for these dependent structures.
```
