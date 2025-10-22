# Introduction

This NOMAD OASIS deployment (NOMAD Tajine) demonstrates the use of NOMAD for general purposes beyond the material science. In this particular case, we adapt NOMAD for cooking recipes. This deployment was created within two days of the nomad-tajine hackathon; it contains only one dedicated plugin (nomad-tajine-plugin). Shown below are the main steps for the creation of NOMAD-Tajine.

# Creating a plugin from the template

A structure for the plugin was created using [nomad-plugin-template](https://github.com/FAIRmat-NFDI/nomad-plugin-template) as a new repository in FAIRmat-NFDI. A step-by-step process is described in the `README.md` file of the template; we chose to include schema package, app and parser. The resulting repository can be found [here](https://github.com/FAIRmat-NFDI/nomad-tajine-plugin). 

# Writing a schema

The previous step resulted in a plugin which has the required structure and entry points defined, but no actual useful information. The next step was to create a schema that defines structure for processed data - in our case, a cooking recipe. The schema was written under `src/nomad-tajine-plugin/schema_packages/schema_package.py`.

The main class of the schema is, naturally, a `Recipe`. We used NOMAD BaseSections for the definition of classes, as they readily provide a framework for basic required functionality. For example, `Recipe` inherits from `BaseSection` and  `Schema` classes: the former provides handling of name, id, descriptions and automatically fills parts of the result section; the latter allows to use `Recipe` for manually creating ELN entries in the OASIS.

`Recipe` class has multiple quantities (`name`, `authors`, `nutrition_value` etc) for storing simple values, and subsections (`steps`, `tools` etc) for storing more complex data. For each subsection, a corresponding class with its own structure was defined. Normalization method of each class allows to analyze the data, for example to create a list of all ingredients needed for the recipe from known ingredients for each step.

Below you can find example code for one of the classes:

```python
class RecipeStep(ActivityStep):
    duration = Quantity(
        type=float,
        a_eln=ELNAnnotation(
            component=ELNComponentEnum.NumberEditQuantity, defaultDisplayUnit='minute'
        ),
        unit='minute',
    )

    tools = SubSection(
        section_def=Tool,
        description='',
        repeats=True,
    )

    ingredients = SubSection(
        section_def=IngredientAmount,
        description='',
        repeats=True,
    )

    instruction = Quantity(
        type=str, a_eln=ELNAnnotation(component=ELNComponentEnum.StringEditQuantity)
    )
```

# Creating an example upload

# Visualization with a NOMAD app

# Adding information from external sources

# Deploying OASIS

