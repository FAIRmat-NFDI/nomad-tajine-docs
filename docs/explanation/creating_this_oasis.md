# Introduction

This NOMAD OASIS deployment (NOMAD Tajine) demonstrates the use of NOMAD for general purposes beyond the material science. In this particular case, we adapt NOMAD for cooking recipes. This deployment was created within two days of the nomad-tajine hackathon; it contains only one dedicated plugin (nomad-tajine-plugin). Shown below are the main steps for the creation of NOMAD-Tajine.

# Creating a plugin from the template

A structure for the plugin was created using [nomad-plugin-template](https://github.com/FAIRmat-NFDI/nomad-plugin-template) as a new repository in FAIRmat-NFDI. A step-by-step process is described in the `README.md` file of the template; we chose to create schema package, app and parser entry points. The resulting repository can be found [here](https://github.com/FAIRmat-NFDI/nomad-tajine-plugin). 

# Writing a schema

The previous step resulted in a plugin which has the required structure and entry points defined, but no actual useful information. The next step was to create a schema that defines structure for processed data - in our case, a cooking recipe. The schema was written under `src/nomad-tajine-plugin/schema_packages/schema_package.py`.

The main class of the schema is, naturally, a `Recipe`. We used NOMAD BaseSections for the definition of classes, as they readily provide a framework for basic required functionality. For example, `Recipe` inherits from `BaseSection` and  `Schema` classes: the former provides handling of name, id, descriptions and automatically fills parts of the result section; the latter allows to use `Recipe` for manually creating ELN entries in the OASIS.

`Recipe` class has multiple quantities (`name`, `authors`, `nutrition_value` etc) for storing simple values, and subsections (`steps`, `tools` etc) for storing more complex data. For each subsection, a corresponding class with its own structure was defined. Normalization method of each class allows to analyze the data, for example to create a list of all ingredients needed for the recipe from known ingredients of each step.

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

This class describes a single step of the recipe; it inherits from `ActivityStep` base section and adds two quantities (both user editable, as defined by `a_eln` parameter) and two subsections (both repeating, as more than one tool or instrument might be needed for a cooking step). Some quantities (such as `name`) are inherited from the base section and therefore do not have to be defined explicitly. The quantities and/or subsections above might not be sufficient for certain types of cooking steps, so this class can be further specialized, for example, by adding temperature value:

```python
class HeatingCoolingStep(RecipeStep):
    temperature = Quantity(
        type=float,
        default=20.0,
        a_eln=ELNAnnotation(
            component=ELNComponentEnum.NumberEditQuantity, defaultDisplayUnit='celsius'
        ),
        unit='celsius',
    )
```

The plugin require testing as it is being written. Full deployment of an OASIS with the plugin takes significant time (see below), therefore we use [nomad-distro-dev](https://github.com/FAIRmat-NFDI/nomad-distro-dev) for the tests. The instructions for it are provided in the `README.md` file; we only had `nomad-lab[parsing, infrastructure]` and `nomad-tajine-plugin` as dependencies (remove other dependencies to speed up the start-up time of nomad-distro-dev).

# Creating an example upload

We created an example upload for one dish (Moroccan Chicken Tagine), that includes information on preparation of the dish in the `src/nomad-tajine-plugin/example_uploads/example/tajine.archive.yaml` file, which is automatically picked up by NOMAD default parser. The ingredients and tools are separate nomad entries referenced by the recipe, with their own corresponding `.archive.yaml` files. This way, the information for the ingredients is not duplicated and can be used in multiple recipes containing the same ingredient.

# Visualization with a NOMAD app

We also created an example app that allows to search for recipes and visualize the information on all the recipes stored in the OASIS. The app code is written under `src/nomad-tajine-plugin/apps/__init__.py`

# Adding information from external sources

To be added...

# Deploying OASIS

To be added...

