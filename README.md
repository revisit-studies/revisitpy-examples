# Running Examples

The simplest way to get started is using the package manager `uv`. If you don't have `uv`, please see [here](https://docs.astral.sh/uv/getting-started/installation/).

Once you have `uv` installed, run the following:

```bash
uv sync
```

This will create the virtual environment. When running a jupyter notebook, use the `ipykernel` that is in the `.venv` directory when prompted.

You can then run the notebooks in the `example_jnd_study` folder as follows:

```bash
uv run jupyter notebook example_jnd_study/example_jnd_study.ipynb
```

# Development

## Creating Markdown files

jupyter nbconvert --to markdown --template=mdoutput example_jnd_study/example_jnd_study.ipynb