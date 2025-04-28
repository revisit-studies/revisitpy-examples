# Simple HTML Study

Here we will get familiar with the syntax of the reVISitPy package. To do this, we'll recreate the config for the simple html study [here](https://github.com/revisit-studies/study/blob/main/public/demo-html-input/config.json). We'll start by first creating the study metadata and the UI configuration.


```python
import revisitpy as rvt

study_metadata = rvt.studyMetadata(
    title='Passing Data From reVISit to HTML and back',
    version='pilot',
    authors=[
        'The reVISit Team',
        'reVISitPy'
    ],
    date='2025-01-20',
    description="A demo of how to pass visualization data from reVISit to HTML and send user answers from HTML back to the reVISit.",
    organizations=[
        "University of Utah",
        "WPI",
        "University of Toronto"
    ]
)

ui_config = rvt.uiConfig(
    contactEmail="contact@revisit.dev",
    helpTextPath="./assets/help.md",
    logoPath="./assets/revisitLogoSquare.svg",
    withProgressBar=True,
    autoDownloadStudy=False,
    autoDownloadTime=5000,
    sidebar=False
)
```

# Creating Response and Base Components

Next we'll create the response and the base component that we will be using. Note that in the reVISitPy package, the base components work in a slightly different way. Instead of directly adjusting the `"baseComponent"` top level key in the configuration file, the inheritance is handled directly in the library. So, when you inspect a component that has a base, the component will already have all the appropriate properties from the base. Similar to the config, however, a base component in `revisitpy` is defined in the same exact way as a regular component.


```python
response_one = rvt.response(
    id='barChart',
    prompt='Your selected answer:',
    location='belowStimulus',
    type='reactive',
    required=True
)

base_component = rvt.component(
    component_name__='bar-chart',
    type='website',
    response=[response_one],
    path="./assets/bar-chart-interaction.html",
    instructionLocation='aboveStimulus'
)
```

# Generating the regular components

Since we have the base components set up, we can create our two other components which both inherit these bases.


```python
# No need to define the response in this component since it will automatically be initialized as empty.
introduction = rvt.component(
    component_name__='introduction',
    type='markdown',
    path='./assets/introduction.md' # <--- Relative path to asset from this notebook
)

comp_one = rvt.component(
    base__=base_component,
    component_name__='bar-chart-1',
    description="A trial for the user to click the largest bar",
    instruction='Click on the largest bar',
    parameters={
        "barData": [
            0.32,
            0.01,
            1.2,
            1.3,
            0.82,
            0.4,
            0.3
        ]
    }
)

comp_two = rvt.component(
    base__=base_component,
    component_name__='bar-chart-2',
    description="A trial for the user to click the smallest bar",
    instruction='Click on the smallest bar',
    parameters={
        "barData": [
            1.2,
            1.2,
            1.2,
            1.3,
            0.82,
            0.4,
            0.3
        ]
    }
)
```

# Generate The Sequence and Final Config

Now that all the individual pieces are created, we'll create the sequence and then the final configuration file.


```python
sequence = rvt.sequence(
    order='fixed',
    components=[
        introduction,
        comp_one,
        comp_two
    ]
)

# Adding 'components' directly is not necessary since they are already defined in the original sequence.
study = rvt.studyConfig(
    schema='https://raw.githubusercontent.com/revisit-studies/study/v2.0.0/src/parser/StudyConfigSchema.json',
    uiConfig=ui_config,
    studyMetadata=study_metadata,
    sequence=sequence
)

print(study)
```
**Output:**
```output
{
    "$schema": "https://raw.githubusercontent.com/revisit-studies/study/v2.0.0/src/parser/StudyConfigSchema.json",
    "components": {
        "introduction": {
            "path": "./assets/introduction.md",
            "response": [],
            "type": "markdown"
        },
        "bar-chart-1": {
            "description": "A trial for the user to click the largest bar",
            "instruction": "Click on the largest bar",
            "instructionLocation": "aboveStimulus",
            "parameters": {
                "barData": [
                    0.32,
                    0.01,
                    1.2,
                    1.3,
                    0.82,
                    0.4,
                    0.3
                ]
            },
            "path": "./assets/bar-chart-interaction.html",
            "response": [
                {
                    "id": "barChart",
                    "location": "belowStimulus",
                    "prompt": "Your selected answer:",
                    "required": true,
                    "type": "reactive"
                }
            ],
            "type": "website"
        },
        "bar-chart-2": {
            "description": "A trial for the user to click the smallest bar",
            "instruction": "Click on the smallest bar",
            "instructionLocation": "aboveStimulus",
            "parameters": {
                "barData": [
                    1.2,
                    1.2,
                    1.2,
                    1.3,
                    0.82,
                    0.4,
                    0.3
                ]
            },
            "path": "./assets/bar-chart-interaction.html",
            "response": [
                {
                    "id": "barChart",
                    "location": "belowStimulus",
                    "prompt": "Your selected answer:",
                    "required": true,
                    "type": "reactive"
                }
            ],
            "type": "website"
        }
    },
    "sequence": {
        "components": [
            "introduction",
            "bar-chart-1",
            "bar-chart-2"
        ],
        "order": "fixed"
    },
    "studyMetadata": {
        "authors": [
            "The reVISit Team",
            "reVISitPy"
        ],
        "date": "2025-01-20",
        "description": "A demo of how to pass visualization data from reVISit to HTML and send user answers from HTML back to the reVISit.",
        "organizations": [
            "University of Utah",
            "WPI",
            "University of Toronto"
        ],
        "title": "Passing Data From reVISit to HTML and back",
        "version": "pilot"
    },
    "uiConfig": {
        "autoDownloadStudy": false,
        "autoDownloadTime": 5000.0,
        "contactEmail": "contact@revisit.dev",
        "helpTextPath": "./assets/help.md",
        "logoPath": "./assets/revisitLogoSquare.svg",
        "sidebar": false,
        "withProgressBar": true
    }
}
```
# Visualizing the Study

Now that we have the study completed, let's use the widget to visualize this. We'll start by using the `revisitpy-server` package to run a local copy of the reVISit repo.

## Running the Server

Simply import the `revisitpy-server` package and call `rs.serve()`. We set the return value of `rs.serve()` to `process` so that we can terminate the process afterwards if desired.


```python
import revisitpy_server as rs

process = rs.serve()
```
**Output:**
```output
Server is running in the background at http://localhost:8080
```
## Launching The Widget

Launching the widget is also straightforward -- especially when using the `revisitpy-server` package.


```python
w = rvt.widget(study, server=True)
# In your own Jupyter notebook, calling `w` will now display the widget in a fully interactive manner.
# w
```
**Output:**
```output
Copying file from ./assets/introduction.md to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/introduction.md
Copying file from ./assets/bar-chart-interaction.html to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/bar-chart-interaction.html
Copying file from ./assets/bar-chart-interaction.html to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/bar-chart-interaction.html
Copying file from ./assets/help.md to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/help.md
Copying file from ./assets/revisitLogoSquare.svg to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/revisitLogoSquare.svg
```