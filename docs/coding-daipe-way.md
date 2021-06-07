# Coding the "Daipe way"

Daipe **greatly simplify datalake(house) management**: 

* Tools to simplify & automate table creation, updates and migrations.
* Explicit table schema enforcing for Hive tables, CSVs, ...
* Decorators to write well-maintainable and self-documented function-based notebooks
* Rich configuration options to customize naming standards, paths, and basically anything to match your needs

!!! info "Why function based notebooks?"

    Compared to bare notebooks, the function-based approach brings the **following advantages**: 
    
    1. create and publish auto-generated documentation and lineage of notebooks and pipelines 
    1. write much cleaner notebooks with properly named code blocks
    1. (unit)test specific notebook functions with ease
    1. use YAML to configure your notebooks for given environment (dev/test/prod/...)
    1. utilize pre-configured objects to automate repetitive tasks


There are two main decorators in the Daipe framework - `@transformation()` and `@notebook_function()`.

1. `@transformation()` understands Spark dataframes better and provides you with extra Spark-related functionality like display and duplicate columns checking.
1. `@notebook_function()` should be used for functions and procedures which don't manipulate with a DataFrame e. g. downloading data.

