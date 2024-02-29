# Writing it all to the files

Now, we will see how the `write_module` method of the `write_morpho` method (actually called `write_python` to use `gen.py` as is) writes all the functions to the `.c`, `.h` and `.md` files.

The full structure, although long, is pretty straightforward:

```python
# GenApi.py
class API:
    ...
    def write_python(self):
        """
        This method is actually `write_morpho`, but is named `write_python` so that
        we can run `gen.py` directly without modification.
        """

        method_names = [] # List to collect method names

        def collect_list_of_outputs(oargs):
            """
            Collect outputs in a Morpho list
            """

        def return_list_of_outputs():
            """
            Return the list of outputs
            """
        ...
        def write_module(module, c_namespace):

            # Collect all the inputs
            iargs = ...
            oargs = ...

            # Colelct all the outputs
            #
            # Create the Morpho function name
            # Write help for this function to the file
            fnamemorpho = ...
            method_names.append(fnamemorpho)
            # Write function definition to the C-file
            # Write checks for number and type of input arguments

            # Capture all the inputs
            for i,iarg in enumerate(iargs):
                self.fwrite(f, iarg.capture_input(i))

            # Initialize the outputs
            for i,oarg in enumerate(oargs):
                self.fwrite(f, oarg.init_output())

            # Create the C function call
            # Write the function call, accounting for direct return values if any.

            # Capture and return output
            # Write the return statement
            if len(oargs)==0 and not rtype: ## If there's nothing to return, return MORPHO_NIL
                self.fwrite(f, INDENTL1 + "return MORPHO_NIL;\n")
            elif len(oargs)==0 and rtype: ## If there are no outputs, but the function returns something, return the value
                self.fwrite(f, INDENTL1 + f"return {rtype.cToMorphoConverter}(outval);\n")
            for oarg in oargs: ## If there are outputs, we need to first capture them as Morpho values.
                self.fwrite(f, oarg.capture_output())
            if (len(oargs)==1): ## If there's only one output, return it
                self.fwrite(f, oargs[0].return_output())
            elif (len(oargs)>1): ## If there are multiple outputs, collect them in a Morpho List and return it
                collect_list_of_outputs(oargs)
                return_list_of_outputs()

        # Recursively write all the submodules
        for m in module.submodules:
            write_module(m, c_namespace)


    # We will first write to the C and MD files simultaneously, and collect the function names, etc for writing the Header file
    # Simultanously open the C file and the help file as we need to add the methods one by one
    with open("../src/" + f"{EXTENSION_NAME}.c", "w") as f, \
         open("../share/help/" + f"{EXTENSION_NAME}.md", "w") as hlp:

         # Write header-material to the files

         # Write all the modules

         # Write the footer for the C file
         # Write the initialize method
         # Add all the functions
         for method in method_names:
             self.fwrite(f, INDENTL1 + f"builtin_addfunction({method.upper()}_FUNCTION, {method}, BUILTIN_FLAGSEMPTY);\n")
         # Add the error definitions.
         self.fwrite(f, cmorpho_error_def)
         self.fwrite(f, "\n}\n")
         # Write the finalize method
         self.fwrite(f, cmorpho_footer)

         # Now, write the header file
         with open("../src/" + f"{EXTENSION_NAME}.h", "w") as fh:
             # Write the header-material for the header file
             # This currently also defines the Errors.
             for method in method_names:
                 # Define all the function names

        # And that's it!
```
