# alquimia_util.c

## Overview

This file provides a collection of utility functions for working with Alquimia data structures. These utilities include string manipulation (duplication, case-insensitive comparison), searching (finding an index from a name in a string vector), deep copying of Alquimia's various data containers, resizing vectors, and printing/logging the content of these containers for debugging purposes.

## Key Components

### String Utilities

*   `AlquimiaStringDup(const char* str)`: Duplicates a string using `malloc` and `strcpy`.
    *   **Returns**: A pointer to the newly allocated string. The caller is responsible for freeing this memory.
*   `AlquimiaCaseInsensitiveStringCompare(const char* const str1, const char* const str2)`: Compares two strings for equality, ignoring case.
    *   **Returns**: `true` if the strings are equal (case-insensitively), `false` otherwise.

### Search Utilities

*   `AlquimiaFindIndexFromName(const char* const name, const AlquimiaVectorString* const names, int* index)`: Searches for a string (`name`) within an `AlquimiaVectorString` (`names`).
    *   **Parameters**:
        *   `name`: The string to search for.
        *   `names`: The vector of strings to search within.
        *   `index`: An output parameter; if the name is found, this is set to the index of the name in the vector. Otherwise, it's set to -1.
    *   **Behavior**: Uses `strncmp` for comparison, up to `kAlquimiaMaxStringLength`.

### Copy Utilities

This group of functions performs deep copies of various Alquimia data structures. For structures containing vectors, the underlying vector data is also copied. For string vectors or structures containing strings, new memory is allocated for the copied strings.

*   `CopyAlquimiaVectorDouble(const AlquimiaVectorDouble* const source, AlquimiaVectorDouble* destination)`
*   `CopyAlquimiaVectorInt(const AlquimiaVectorInt* const source, AlquimiaVectorInt* destination)`
*   `CopyAlquimiaVectorString(const AlquimiaVectorString* const source, AlquimiaVectorString* destination)`
*   `CopyAlquimiaSizes(const AlquimiaSizes* const source, AlquimiaSizes* destination)`: Uses `memcpy` for a shallow copy as `AlquimiaSizes` contains only primitive types.
*   `CopyAlquimiaProblemMetaData(const AlquimiaProblemMetaData* const source, AlquimiaProblemMetaData* destination)`
*   `CopyAlquimiaProperties(const AlquimiaProperties* const source, AlquimiaProperties* destination)`
*   `CopyAlquimiaEngineFunctionality(const AlquimiaEngineFunctionality* const source, AlquimiaEngineFunctionality* destination)`: Uses `memcpy` for a shallow copy.
*   `CopyAlquimiaState(const AlquimiaState* const source, AlquimiaState* destination)`
*   `CopyAlquimiaAuxiliaryData(const AlquimiaAuxiliaryData* const source, AlquimiaAuxiliaryData* destination)`
*   `CopyAlquimiaAuxiliaryOutputData(const AlquimiaAuxiliaryOutputData* const source, AlquimiaAuxiliaryOutputData* destination)`
*   `CopyAlquimiaGeochemicalCondition(const AlquimiaGeochemicalCondition* const source, AlquimiaGeochemicalCondition* destination)`
*   `CopyAlquimiaGeochemicalConditionVector(const AlquimiaGeochemicalConditionVector* source, AlquimiaGeochemicalConditionVector* destination)`
*   `CopyAlquimiaAqueousConstraint(const AlquimiaAqueousConstraint* const source, AlquimiaAqueousConstraint* destination)`
*   `CopyAlquimiaAqueousConstraintVector(const AlquimiaAqueousConstraintVector* const source, AlquimiaAqueousConstraintVector* destination)`
*   `CopyAlquimiaMineralConstraint(const AlquimiaMineralConstraint* const source, AlquimiaMineralConstraint* destination)`
*   `CopyAlquimiaMineralConstraintVector(const AlquimiaMineralConstraintVector* const source, AlquimiaMineralConstraintVector* destination)`
    *   **Behavior**: For vector types, these functions typically call the corresponding `ResizeAlquimiaVector[Type]` function on the destination and then copy the data (using `memcpy` for primitive types or `AlquimiaStringDup` for strings). For complex structures, they copy scalar fields directly and call the appropriate `CopyAlquimiaVector[Type]` or other copy functions for member structures/vectors.

### Resize Utilities

These functions adjust the size of Alquimia vectors. If the new size is larger than the current capacity, they reallocate memory for the vector's data array, typically by doubling the capacity until it's sufficient.

*   `ResizeAlquimiaVectorDouble(AlquimiaVectorDouble* vec, int new_size)`
*   `ResizeAlquimiaVectorInt(AlquimiaVectorInt* vec, int new_size)`
*   `ResizeAlquimiaVectorString(AlquimiaVectorString* vec, int new_size)`
*   `ResizeAlquimiaGeochemicalConditionVector(AlquimiaGeochemicalConditionVector* vec, int new_size)`
*   `ResizeAlquimiaAqueousConstraintVector(AlquimiaAqueousConstraintVector* vec, int new_size)`
*   `ResizeAlquimiaMineralConstraintVector(AlquimiaMineralConstraintVector* vec, int new_size)`
    *   **Behavior**: If the vector's current size (`vec->size`) is 0, it calls the corresponding `AllocateAlquimiaVector[Type]` function. If `new_size` exceeds `vec->capacity`, `realloc` is used to expand `vec->data`. The `vec->size` is then updated to `new_size`. Note: For `ResizeAlquimiaVectorString`, if expanding an existing non-empty vector, the newly added `char*` elements in `vec->data` are not initialized to point to allocated strings; this would need to be handled separately if those new string elements are to be used.

### Print Utilities

These functions print the contents of Alquimia data structures to a specified `FILE` stream, primarily for debugging.

*   `PrintAlquimiaVectorDouble(const char* const name, const AlquimiaVectorDouble* const vector, FILE* file)`
*   `PrintAlquimiaVectorInt(const char* const name, const AlquimiaVectorInt* const vector, FILE* file)`
*   `PrintAlquimiaVectorString(const char* const name, const AlquimiaVectorString* const vector, FILE* file)`
*   `PrintAlquimiaData(const AlquimiaData* const data, FILE* file)`: Prints a comprehensive overview of the `AlquimiaData` struct, calling other print functions for its members.
*   `PrintAlquimiaSizes(const AlquimiaSizes* const sizes, FILE* file)`
*   `PrintAlquimiaEngineFunctionality(const AlquimiaEngineFunctionality* const functionality, FILE* file)`
*   `PrintAlquimiaProblemMetaData(const AlquimiaProblemMetaData* const meta_data, FILE* file)`
*   `PrintAlquimiaProperties(const AlquimiaProperties* const mat_prop, FILE* file)`
*   `PrintAlquimiaState(const AlquimiaState* const state, FILE* file)`
*   `PrintAlquimiaAuxiliaryData(const AlquimiaAuxiliaryData* const aux_data, FILE* file)`
*   `PrintAlquimiaAuxiliaryOutputData(const AlquimiaAuxiliaryOutputData* const aux_output, FILE* file)`
*   `PrintAlquimiaGeochemicalConditionVector(const AlquimiaGeochemicalConditionVector* const condition_list, FILE* file)`
*   `PrintAlquimiaGeochemicalCondition(const AlquimiaGeochemicalCondition* const condition, FILE* file)`
*   `PrintAlquimiaAqueousConstraint(const AlquimiaAqueousConstraint* const constraint, FILE* file)`
*   `PrintAlquimiaMineralConstraint(const AlquimiaMineralConstraint* const constraint, FILE* file)`

## Important Variables/Constants

*   `kAlquimiaMaxStringLength` (from `alquimia_constants.h`): Used by `AlquimiaFindIndexFromName` as the maximum number of characters to compare with `strncmp`.

## Usage Examples

```c
#include "alquimia/alquimia_util.h"
#include "alquimia/alquimia_memory.h" // For allocation examples
#include <stdio.h>

int main() {
    // String duplication
    char* original_str = "TestString";
    char* duplicated_str = AlquimiaStringDup(original_str);
    printf("Original: %s, Duplicated: %s\n", original_str, duplicated_str);
    free(duplicated_str);

    // Case-insensitive compare
    printf("Compare 'test' and 'TEST': %s\n",
           AlquimiaCaseInsensitiveStringCompare("test", "TEST") ? "Equal" : "Not Equal");
    printf("Compare 'test' and 'TESA': %s\n",
           AlquimiaCaseInsensitiveStringCompare("test", "TESA") ? "Equal" : "Not Equal");

    // Find index from name
    AlquimiaVectorString names_vec;
    AllocateAlquimiaVectorString(2, &names_vec);
    strcpy(names_vec.data[0], "SpeciesA");
    strcpy(names_vec.data[1], "SpeciesB");
    int index;
    AlquimiaFindIndexFromName("SpeciesB", &names_vec, &index);
    printf("Index of 'SpeciesB': %d\n", index);
    AlquimiaFindIndexFromName("SpeciesC", &names_vec, &index);
    printf("Index of 'SpeciesC': %d\n", index);
    FreeAlquimiaVectorString(&names_vec);

    // Copying and Printing (Example for AlquimiaState)
    AlquimiaSizes sizes;
    sizes.num_primary = 1;
    // ... (initialize other fields of sizes as needed for a complete example) ...
    sizes.num_sorbed=0; sizes.num_surface_sites=0; sizes.num_ion_exchange_sites=0;
    sizes.num_minerals=0; sizes.num_gases=0;

    AlquimiaState state_orig, state_copy;
    AllocateAlquimiaState(&sizes, &state_orig);
    if (state_orig.total_mobile.size > 0) state_orig.total_mobile.data[0] = 1.23;
    state_orig.temperature = 25.0;

    AllocateAlquimiaState(&sizes, &state_copy); // Destination needs to be allocated
    CopyAlquimiaState(&state_orig, &state_copy);

    printf("Original State:\n");
    PrintAlquimiaState(&state_orig, stdout);
    printf("Copied State:\n");
    PrintAlquimiaState(&state_copy, stdout);

    FreeAlquimiaState(&state_orig);
    FreeAlquimiaState(&state_copy);

    return 0;
}
```

## Dependencies and Interactions

*   **Internal Dependencies**:
    *   `alquimia/alquimia_util.h`: Declares the utility functions.
    *   `alquimia/alquimia_memory.h`: Provides allocation (`AllocateAlquimiaVector[Type]`) and deallocation functions, which are used by some copy and resize utilities if the destination vector is initially empty or needs to be cleared.
    *   `alquimia/alquimia_interface.h`: Defines some of the structures (like `AlquimiaEngineFunctionality`) that have copy and print utilities.
    *   `alquimia/alquimia_constants.h`: Provides `kAlquimiaMaxStringLength`.
    *   The file implicitly depends on the definitions of all Alquimia data structures (`AlquimiaVectorDouble`, `AlquimiaState`, etc.) which are typically found in `alquimia_containers.h`.
*   **External Libraries**:
    *   `<string.h>`: For `strlen`, `strcpy`, `strncmp`, `memcpy`.
    *   `<ctype.h>`: For `tolower`.
    *   `<stdlib.h>`: For `malloc`, `realloc`, `free`.
    *   `<stdio.h>`: For `FILE` and `fprintf` used in print utilities.
*   **Interactions with Other Components**:
    *   These utilities are broadly used throughout Alquimia and by client applications for common tasks:
        *   Managing string resources.
        *   Deep copying state or metadata for various purposes (e.g., storing history, passing data to different components).
        *   Dynamically adjusting vector sizes.
        *   Debugging and logging the state of Alquimia data structures.
    *   The copy functions are crucial for maintaining data integrity and preventing shallow copy pitfalls when dealing with complex, dynamically allocated structures.
    *   Resize functions provide flexibility in handling data arrays whose sizes might not be known at compile time or might change during execution.
```
