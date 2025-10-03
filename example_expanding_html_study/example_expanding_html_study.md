# Enhancing The Simple HTML Study

In the previous example, we went over creating the configuration file shown [here](https://github.com/revisit-studies/study/blob/main/public/demo-html-input/config.json). While that example is still a valid use case for the `revisitpy` package, this may be a configuration that is simpler to write in standard JSON. In this example, we'll go over a use case which more readily requires the `revisitpy` package.

Instead of defining the data for each bar chart directly in the python notebook, suppose we have a CSV file which contains the data for each bar chart. Depending on your specific needs, you may be generating data in R or some other separate Python environment. In these cases, it is extremely useful to have a seamless way to convert raw data into well-defined reVISit components.

Rather than copy this data over into our jupyter notebook, we will use the built-in `data` function to first import the CSV file and then generate components from the data set using the `Sequence` class's `from_data` method.

## Using the Data Parser and `from_data` method

We load in the data using the built-in `data` function. This will generate a list of `DataRows` classes. We can then pass this directly into the `from_data` method of the sequence we create.


```python
import revisitpy as rvt

# Import the data using the build in data parser.
study_data = rvt.data('./assets/data.csv')

# Generate a single component for each element of the dataset.
data_sequence = rvt.sequence(order='random').from_data(study_data)

print('------- Sequence --------')
print(data_sequence)

print('------- Individual Component -------')
print(data_sequence.get_components()[0])
```
**Output:**
```output
------- Sequence --------
{
    "components": [
        "place-holder-component_id:1_b1:0.32_b2:0.01_b3:1.2_b4:1.3_b5:0.82_b6:0.4_b7:0.3",
        "place-holder-component_id:2_b1:1.2_b2:1.2_b3:1.2_b4:1.3_b5:0.82_b6:0.4_b7:0.3",
        "place-holder-component_id:3_b1:0.6_b2:1.1_b3:0.7_b4:0.74_b5:1.3_b6:0.1_b7:0.9",
        "place-holder-component_id:4_b1:1.3_b2:1.1_b3:1.3_b4:0.64_b5:0.31_b6:0.2_b7:0.8",
        "place-holder-component_id:5_b1:0.1_b2:0.1_b3:0.3_b4:0.4_b5:0.5_b6:1.1_b7:0.2",
        "place-holder-component_id:6_b1:0.3_b2:1.1_b3:0.65_b4:0.77_b5:0.4_b6:1.3_b7:1.1"
    ],
    "order": "random"
}
------- Individual Component -------
{
    "meta": {
        "id": 1,
        "b1": 0.32,
        "b2": 0.01,
        "b3": 1.2,
        "b4": 1.3,
        "b5": 0.82,
        "b6": 0.4,
        "b7": 0.3
    },
    "response": [],
    "type": "questionnaire"
}
```
## Transforming The Components

Notice that each component in this sequence we created is not the website component that we'd like. Instead, these are "filler components". The idea of the `from_data` method (or the similarly designed `permute` method) is to generate the _structure_ of the sequence. This method applies each of the data rows to the `meta` attribute of the components. These attributes will be printed out when inspecting the JSON and will appear in the final config. However, in general, the `meta` attribute does not have any tangible effect on the resulting component. This `meta` attribute is used when transforming these filler components into our desired components. Below we inspect the `meta` attribute of the first component in the sequence.


```python
print(data_sequence.get_components()[0].get('meta'))
```
**Output:**
```output
{'id': 1, 'b1': 0.32, 'b2': 0.01, 'b3': 1.2, 'b4': 1.3, 'b5': 0.82, 'b6': 0.4, 'b7': 0.3}
```
Now that we've verified that the metadata attributes are correctly applied to the components, lets transform these filler components to the components we'd like. We start by creating a "component function"


```python
# Define the response and the base component.

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

def bar_chart_component_function(id, b1, b2, b3, b4, b5, b6, b7):
    # Toggle between 'largest' and 'smallest' depending on 
    # if the original ID is an odd or even number.
    largest_smallest_string = 'smallest' if id % 2 == 0 else 'largest'
    
    comp = rvt.component(
        base__=base_component,
        component_name__=f'bar-chart-{id}',
        description=f"A trial for the user to click the {largest_smallest_string} bar",
        instruction=f'Click on the {largest_smallest_string} bar',
        parameters={
            "barData": [b1, b2, b3, b4, b5, b6, b7]
        }
    )
    
    # Always return a component in our component functions.
    return comp
```

The component function defined above has the headers of the CSV file as the parameters to the function. When we use this component function, the `meta` attributes of each filler component will be passed into as arguments to this function and return the resulting component to overwrite the filler component. 

Optionally, we could define this component function with the single parameter `**kwargs`. This will ensure that any argument in the metadata attributes can be passed into the function. However, this would then require you to use syntax like `kwargs.get('b1')` instead of just `b1`. You should consider these options when designing your component function based on your specific needs.

If the component function fails to create the component, the operation will not totally fail. Instead, the original filler component will remain in its place.

Now that we have the function defined, the transformation is simple:


```python
data_sequence.component(bar_chart_component_function)

print(data_sequence)
print(data_sequence.get_components()[0])
```
**Output:**
```output
{
    "components": [
        "bar-chart-6",
        "bar-chart-5",
        "bar-chart-4",
        "bar-chart-3",
        "bar-chart-2",
        "bar-chart-1"
    ],
    "order": "random"
}
{
    "description": "A trial for the user to click the smallest bar",
    "instruction": "Click on the smallest bar",
    "instructionLocation": "aboveStimulus",
    "parameters": {
        "barData": [
            0.3,
            1.1,
            0.65,
            0.77,
            0.4,
            1.3,
            1.1
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
```
## Creating the Final Sequence

Now that we have the new sequence we created, let's redefine our study config and the inputted sequence.


```python

introduction = rvt.component(
    component_name__='introduction',
    type='markdown',
    path='./assets/introduction.md' # <--- Relative path to asset from this notebook
)

sequence = rvt.sequence(order='fixed', components=[introduction]) + data_sequence

print(sequence)

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
    withSidebar=False
)

study = rvt.studyConfig(
    schema='https://raw.githubusercontent.com/revisit-studies/study/v2.0.0/src/parser/StudyConfigSchema.json',
    uiConfig=ui_config,
    studyMetadata=study_metadata,
    sequence=sequence
)
```
**Output:**
```output
{
    "components": [
        "introduction",
        {
            "components": [
                "bar-chart-6",
                "bar-chart-5",
                "bar-chart-4",
                "bar-chart-3",
                "bar-chart-2",
                "bar-chart-1"
            ],
            "order": "random"
        }
    ],
    "order": "fixed"
}
```
Finally, lets preview this in our widget.


```python
import revisitpy_server as rs

process = rs.serve()

w = rvt.widget(study, server=True)

# In your own Jupyter notebook, calling `w` will now display the widget in a fully interactive manner.
# w
```
**Output:**
```output
Server is running in the background at http://localhost:8080
Copying file from ./assets/introduction.md to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/introduction.md
Copying file from ./assets/bar-chart-interaction.html to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/bar-chart-interaction.html
Copying file from ./assets/bar-chart-interaction.html to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/bar-chart-interaction.html
Copying file from ./assets/bar-chart-interaction.html to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/bar-chart-interaction.html
Copying file from ./assets/bar-chart-interaction.html to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/bar-chart-interaction.html
Copying file from ./assets/bar-chart-interaction.html to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/bar-chart-interaction.html
Copying file from ./assets/bar-chart-interaction.html to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/bar-chart-interaction.html
Copying file from ./assets/help.md to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/help.md
Copying file from ./assets/revisitLogoSquare.svg to /Users/bbollen23/revisit-py-examples/.venv/lib/python3.12/site-packages/revisitpy_server/static/__revisit-widget/assets/revisitLogoSquare.svg
```