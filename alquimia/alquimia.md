# alquimia.c

## Overview

This file provides core error handling and version reporting functionalities for the Alquimia geochemistry software. It allows setting custom error and abort handlers and printing the Alquimia version.

## Key Components

### Functions

*   `alquimia_error(const char* message, ...)`: Reports a fatal error.
    *   **Parameters**:
        *   `message`: A format string for the error message.
        *   `...`: Variadic arguments for the format string.
    *   **Behavior**: If a custom error handler is set via `alquimia_set_error_handler`, it calls that handler. Otherwise, it prints the formatted error message to standard output and exits the program with a status of -1.
*   `alquimia_set_error_handler(alquimia_error_handler_function handler)`: Sets a custom function to be called when `alquimia_error` is invoked.
    *   **Parameters**:
        *   `handler`: A function pointer of type `alquimia_error_handler_function` which takes a `const char*` (the error message) as an argument.
*   `alquimia_abort(const char* message, ...)`: Reports a condition that requires program termination via abort.
    *   **Parameters**:
        *   `message`: A format string for the abort message.
        *   `...`: Variadic arguments for the format string.
    *   **Behavior**: If a custom abort handler is set via `alquimia_set_abort_function`, it calls that handler. Otherwise, it prints the formatted abort message to standard output and calls `abort()`.
*   `alquimia_set_abort_function(alquimia_abort_function function)`: Sets a custom function to be called when `alquimia_abort` is invoked.
    *   **Parameters**:
        *   `function`: A function pointer of type `alquimia_abort_function` which takes a `const char*` (the abort message) as an argument.
*   `alquimia_version_fprintf(const char* exe_name, FILE* stream)`: Prints the Alquimia version to a specified file stream.
    *   **Parameters**:
        *   `exe_name`: The name of the executable, printed alongside the version.
        *   `stream`: The `FILE*` stream to print the version information to (e.g., `stdout`, `stderr`).
    *   **Behavior**: If the stream is `NULL`, the function does nothing. Otherwise, it prints a string in the format "`exe_name` v`ALQUIMIA_VERSION`".

### Static Functions

*   `default_error_handler(const char* message)`: The default error handler used if no custom handler is set. Prints the message and calls `exit(-1)`.
*   `default_abort_function(const char* message)`: The default abort handler used if no custom handler is set. Prints the message and calls `abort()`.

## Important Variables/Constants

*   `error_handler` (static `alquimia_error_handler_function`): Function pointer storing the current error handler. Initialized to `NULL`.
*   `abort_function` (static `alquimia_abort_function`): Function pointer storing the current abort handler. Initialized to `NULL`.
*   `ALQUIMIA_VERSION` (macro, defined in `alquimia.h`): A preprocessor macro that expands to the current version string of Alquimia.

## Usage Examples

```c
#include "alquimia/alquimia.h"
#include <stdio.h>

// Custom error handler example
void my_custom_error_handler(const char* message) {
    fprintf(stderr, "MY CUSTOM ERROR: %s\n", message);
    // Maybe do some custom cleanup
    exit(1);
}

int main(int argc, char** argv) {
    // Print version information
    alquimia_version_fprintf(argv[0], stdout);

    // Set a custom error handler
    alquimia_set_error_handler(my_custom_error_handler);

    // Trigger an error
    int user_value = -5;
    if (user_value < 0) {
        alquimia_error("User value %d is negative, which is not allowed.", user_value);
    }

    // This part will not be reached if the error is triggered
    printf("Program continues normally.\n");
    return 0;
}
```

## Dependencies and Interactions

*   **Internal Dependencies**:
    *   `alquimia/alquimia.h`: Contains the declarations for the functions and types defined in `alquimia.c`, including `alquimia_error_handler_function`, `alquimia_abort_function`, and the `ALQUIMIA_VERSION` macro.
*   **External Libraries**:
    *   `<stdarg.h>`: Used for handling variadic arguments in `alquimia_error` and `alquimia_abort`.
    *   `<stdio.h>`: (Implicitly via `alquimia.h` for `FILE*`, and used in default handlers) For `printf`, `vsnprintf`, `fprintf`.
    *   `<stdlib.h>`: (Used in default handlers) For `exit`, `abort`.
*   **Interactions with Other Components**:
    *   This file provides fundamental services (error reporting, aborting, versioning) that can be used by any other component within the Alquimia project or by applications linking against Alquimia.
    *   Other parts of the Alquimia library or driver codes will call `alquimia_error` or `alquimia_abort` to handle critical situations.
    *   The behavior of these calls can be customized by the main application by providing alternative handler functions.
```
