# GitHub Action for pytest

This GitHub Action allows you to run `pytest` and output [GitHub Job Summaries](https://github.blog/2022-05-09-supercharging-github-actions-with-job-summaries/). For creating the job summaries, this action uses [pytest-md](https://github.com/hackebrot/pytest-md) and [pytest-emoji](https://github.com/hackebrot/pytest-emoji).

```yaml
- name: Run pytest
  uses: Quantco/pytest-action@v2
  with:
    verbose: true
    emoji: true
    job-summary: true
    custom-arguments: '-q'
```

You need to have Python as well as `pytest` installed in your pipeline before you can run this action. If `job-summary` is set to `true`, you also need to install `pytest-md`. If `emoji` is set to `true`, you need to install `pytest-emoji`.

If you want to change the time zone of the job summary, you may want to use the [szenius/set-timezone](https://github.com/marketplace/actions/set-timezone) action:
```yaml
- name: Set timezone
  uses: szenius/set-timezone@dd47655c84241eec2ffa0a855959c16c0920c3c4
  with:
    timezone: 'Europe/Berlin'
```

When `job-summary` is set to `true`, the action will output a Job Summary.

![Example Job Summary](https://user-images.githubusercontent.com/29506042/170843320-2bb104c5-5284-4fff-a83c-525da58a1a7f.png)

## Activating `conda` environments

This action uses `bash -l {0}` as the shell to run `pytest` in, 
i.e., the login shell that also sources your `.bash_profile`. 
When using `bash` in GitHub Actions, it doesn't source your `.bash_profile` by default. 
If you want to use a `conda` environment, you need to make sure to add it into your `.bash_profile` s.t. 
the `conda` environment automatically gets activated. 
[mamba-org/provision-with-micromamba](https://github.com/mamba-org/provision-with-micromamba) 
does this automatically for you.

```bash
- uses: mamba-org/provision-with-micromamba@a319a818023c3a5a448a60bd72e0c633408fbccc
  with:
    environment-name: myenv
    channels: conda-forge
    environment-file: false
    extra-specs: |
      python=3.7
      pytest
      pytest-md
      pytest-emoji
      numpy
- uses: Quantco/pytest-action@v2
```

## Example Usage

With [mamba-org/provision-with-micromamba](https://github.com/mamba-org/provision-with-micromamba):

```yaml
name: Run Python tests

on: [push]

jobs:
  build:
    name: Run tests
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.6", "3.7", "3.8", "3.9"]

    steps:
      - uses: szenius/set-timezone@dd47655c84241eec2ffa0a855959c16c0920c3c4
        with:
          timezoneLinux: "Europe/Berlin"
      - uses: actions/checkout@v3
      - name: Set up Conda env
        uses: mamba-org/provision-with-micromamba@a319a818023c3a5a448a60bd72e0c633408fbccc
        with:
          cache-env: true
          extra-specs: |
            python
            pytest
            pytest-md
            pytest-emoji
      - uses: Quantco/pytest-action@v2
        with:
          emoji: false
          verbose: false
          job-summary: true
```

With [actions/setup-python](https://github.com/actions/setup-python):

```yaml
name: Run Python tests

on: [push]

jobs:
  build:
    name: Run tests
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.6", "3.7", "3.8", "3.9"]

    steps:
      - uses: szenius/set-timezone@dd47655c84241eec2ffa0a855959c16c0920c3c4
        with:
          timezoneLinux: "Europe/Berlin"
      - uses: actions/checkout@v3
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v3
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install dependencies
        run: pip install pytest pytest-md pytest-emoji
      - uses: Quantco/pytest-action@v2
        with:
          emoji: false
          verbose: false
          job-summary: true
```
