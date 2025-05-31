# alquimia_containers.F90

## Overview

This Fortran module, `AlquimiaContainers_module`, defines a set of derived types that mirror the C data structures used by Alquimia. These types are crucial for the C-Fortran interoperability, allowing data to be passed between Alquimia's C core and Fortran-based geochemistry engines or client codes. The definitions adhere strictly to the Alquimia API to ensure compatibility, and all types use `bind(c)` to be C-compatible. The module also re-defines some key constants from the C side.

## Key Components

This module primarily defines Fortran derived types and constants. It does not contain executable procedures (functions or subroutines) itself, but rather provides the data structure definitions used by other Fortran interface modules.

### Modules

*   `AlquimiaContainers_module`: The main module containing all type and constant definitions.
    *   **Constants**:
        *   `kAlquimiaMaxStringLength` (integer (c_int), parameter): Maximum string length (512).
        *   `kAlquimiaMaxWordLength` (integer (c_int), parameter): Maximum word length (32).
        *   Error codes (e.g., `kAlquimiaNoError`, `kAlquimiaErrorInvalidEngine`).
        *   Geochemical quantity string identifiers (e.g., `kAlquimiaStringTotalAqueous`, `kAlquimiaStringPH`).
    *   **Derived Types (all `bind(c)`)**:
        *   `AlquimiaVectorDouble`: Represents a dynamic array of doubles.
            *   `size` (integer(c_int)): Number of elements.
            *   `capacity` (integer(c_int)): Allocated capacity.
            *   `data` (type(c_ptr)): C pointer to the double array.
        *   `AlquimiaVectorInt`: Represents a dynamic array of integers.
            *   `size` (integer(c_int)): Number of elements.
            *   `capacity` (integer(c_int)): Allocated capacity.
            *   `data` (type(c_ptr)): C pointer to the integer array.
        *   `AlquimiaVectorString`: Represents a dynamic array of strings.
            *   `size` (integer(c_int)): Number of strings.
            *   `capacity` (integer(c_int)): Allocated capacity for the array of pointers.
            *   `data` (type(c_ptr)): C pointer to an array of C pointers (char**).
        *   `AlquimiaSizes`: Defines the number of various entities in the geochemical problem.
            *   `num_primary` (integer(c_int))
            *   `num_sorbed` (integer(c_int))
            *   `num_minerals` (integer(c_int))
            *   ... (and other counts for different species, sites, auxiliary data, etc.)
        *   `AlquimiaState`: Holds the state variables of the geochemical system.
            *   `water_density` (real(c_double))
            *   `porosity` (real(c_double))
            *   `temperature` (real(c_double))
            *   `aqueous_pressure` (real(c_double))
            *   `total_mobile` (type(AlquimiaVectorDouble)): Total mobile concentrations of primary species.
            *   `total_immobile` (type(AlquimiaVectorDouble)): Total immobile concentrations.
            *   ... (and other vectors for mineral fractions, site densities, gas concentrations, etc.)
        *   `AlquimiaProperties`: Contains material and problem-specific properties.
            *   `volume` (real(c_double))
            *   `saturation` (real(c_double))
            *   `isotherm_kd` (type(AlquimiaVectorDouble))
            *   ... (and other vectors for Freundlich/Langmuir constants, mineral rate constants, etc.)
        *   `AlquimiaAuxiliaryData`: Container for auxiliary integer and double data.
            *   `aux_ints` (type(AlquimiaVectorInt))
            *   `aux_doubles` (type(AlquimiaVectorDouble))
        *   `AlquimiaEngineStatus`: Reports the status of an engine operation.
            *   `error` (integer(c_int)): Error code.
            *   `message` (type(c_ptr)): C pointer to a status message string.
            *   `converged` (logical(c_bool))
            *   ... (and other solver/iteration statistics)
        *   `AlquimiaEngineFunctionality`: Describes capabilities of the geochemistry engine.
            *   `thread_safe` (logical(c_bool))
            *   `temperature_dependent` (logical(c_bool))
            *   ... (and other boolean flags for functionalities)
        *   `AlquimiaProblemMetaData`: Contains metadata about the geochemical problem, like names of species.
            *   `primary_names` (type(AlquimiaVectorString))
            *   `positivity` (type(AlquimiaVectorInt)): Flags for positivity constraints.
            *   `mineral_names` (type(AlquimiaVectorString))
            *   ... (and other string vectors for names of sites, gases, etc.)
        *   `AlquimiaAuxiliaryOutputData`: Holds additional output data from engine calculations.
            *   `pH` (real(c_double))
            *   `aqueous_kinetic_rate` (type(AlquimiaVectorDouble))
            *   `mineral_saturation_index` (type(AlquimiaVectorDouble))
            *   ... (and other vectors for reaction rates, concentrations, activity coefficients, etc.)
        *   `AlquimiaAqueousConstraint`: Defines a single aqueous constraint.
            *   `primary_species_name` (type(c_ptr)): C pointer to string.
            *   `constraint_type` (type(c_ptr)): C pointer to string.
            *   `associated_species` (type(c_ptr)): C pointer to string.
            *   `value` (real(c_double))
        *   `AlquimiaAqueousConstraintVector`: Represents a dynamic array of `AlquimiaAqueousConstraint`. (Note: `data` is `type(c_ptr)`, so it points to an array of C structs).
        *   `AlquimiaMineralConstraint`: Defines a single mineral constraint.
            *   `mineral_name` (type(c_ptr)): C pointer to string.
            *   `volume_fraction` (real(c_double))
            *   `specific_surface_area` (real(c_double))
        *   `AlquimiaMineralConstraintVector`: Represents a dynamic array of `AlquimiaMineralConstraint`. (Note: `data` is `type(c_ptr)`).
        *   `AlquimiaGeochemicalCondition`: Defines a named geochemical condition with sets of aqueous and mineral constraints.
            *   `name` (type(c_ptr)): C pointer to string.
            *   `aqueous_constraints` (type(AlquimiaAqueousConstraintVector))
            *   `mineral_constraints` (type(AlquimiaMineralConstraintVector))

## Important Variables/Constants

The module defines several constants that mirror those in `alquimia_constants.c` and `alquimia_constants.h`. These are used for consistency in values like maximum string lengths, error codes, and standard string identifiers for geochemical entities.

*   `kAlquimiaMaxStringLength`, `kAlquimiaMaxWordLength`
*   `kAlquimiaNoError`, `kAlquimiaErrorInvalidEngine`, etc.
*   `kAlquimiaStringTotalAqueous`, `kAlquimiaStringPH`, etc.

## Usage Examples

This module itself is not directly "used" by running procedures within it. Instead, other Fortran modules that implement Alquimia interfaces (e.g., for PFloTran or CrunchFlow) would `use AlquimiaContainers_module` to access these type definitions.

```fortran
module MyGeochemistryInterface_module
  use, intrinsic :: iso_c_binding
  use AlquimiaContainers_module

  implicit none

contains

  subroutine process_geochemical_state(c_state_ptr)
    type(c_ptr), intent(in) :: c_state_ptr
    type(AlquimiaState), pointer :: f_state

    ! Associate the Fortran pointer with the C pointer
    call c_f_pointer(c_state_ptr, f_state)

    ! Now access elements of f_state
    print *, "Porosity from Fortran: ", f_state%porosity
    print *, "Temperature: ", f_state%temperature

    ! Example of accessing a vector (assuming it's populated from C)
    if (f_state%total_mobile%size > 0) then
      real(c_double), pointer :: mobile_data_ptr(:)
      call c_f_pointer(f_state%total_mobile%data, mobile_data_ptr, &
                       shape=[f_state%total_mobile%size])
      print *, "First total mobile concentration: ", mobile_data_ptr(1)
    end if

  end subroutine process_geochemical_state

end module MyGeochemistryInterface_module
```

## Dependencies and Interactions

*   **Internal Dependencies**: None beyond the Fortran standard and `iso_c_binding`.
*   **External Libraries**: None.
*   **Interactions with Other Components**:
    *   This module is fundamental for any Fortran code that needs to interoperate with the Alquimia C library.
    *   Fortran-based geochemistry engines (like PFLOTRAN, CrunchFlow when interfaced via Fortran wrappers) will use these types to map data passed from or to the Alquimia C core.
    *   The C counterparts of these structures are defined in `alquimia_containers.h` (and implemented/used in `alquimia_memory.c`, `alquimia_util.c`, etc.). The `bind(c)` attribute and careful ordering of type members ensure that a Fortran `type(AlquimiaState)` is memory-layout compatible with a C `AlquimiaState` struct.
    *   The `type(c_ptr)` members in vector types and for strings (like `name` in `AlquimiaGeochemicalCondition`) mean that the actual data for these arrays/strings is managed by C. Fortran code would typically use `c_f_pointer` to associate these with Fortran pointers to access the data.
```
