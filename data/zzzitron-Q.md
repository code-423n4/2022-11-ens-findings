# QA

Label | Title
---|---
L00 | NameWrapper: Missing `isWrapped` function

## LOW

### L00 NameWrapper: Missing `isWrapped` function

According to the `wrapper/README.md`:

> To check if a name has been wrapped, call `isWrapped()`. This checks:
> 
> - The NameWrapper is the owner in the Registry contract
> - The owner in the NameWrapper is non-zero

However, there is no implementation of the `isWrapped()` function.
