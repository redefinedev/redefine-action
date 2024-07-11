[![Maintained by Redefine.dev](https://img.shields.io/badge/maintained%20by-Redefine.dev-blueviolet)](https://bit.ly/40XeRzi)

# Redefine GitHub action

This GitHub Action installs, configures &amp; runs [Redefine](https://bit.ly/3RjVflV) to optimize CI execution time and resources.

## Usage

```yaml
- name: Redefine GitHub Action
    id: redefine
    uses: redefinedev/redefine-action@main
    with:
        # Redefine authentication key given by Redefine in the subscription process, stored as a secret.
        auth: ${{ secrets.REDEFINE_AUTH }}
        # Python virtual environment path
        python-venv-path: .venv/
        # Testing framework to optimize
        testing-framework: pytest
        # Redefine mode as described in https://docs.redefine.dev/configuration/selection-modes
        mode: fail-fast
        # The maximum time to run tests with Redefine optimization
        time-limit: 300
```

## Inputs

### Parameters

name|description|default|
|---|---|---|
|`auth`|Redefine authentication key given in by Redefine in the onboarding process|-|
|`testing-framework`|The [testing framework](https://docs.redefine.dev/integrations/supported-technologies) to optimize.|-|
|`command`|Redefine CLI command, one of "[`install`](https://docs.redefine.dev/configuration/install-command)", "[`verify`](https://docs.redefine.dev/welcome-to-redefine/quick-start#verify)", "[`get session_id`](https://docs.redefine.dev/configuration/remote-workers#configure-a-shared-session-id)", "[`get session_check`](https://docs.redefine.dev/configuration/remote-workers#session-check-command)".|`install`|
|`mode`|Redefine execution mode, on of "[`discover`](https://docs.redefine.dev/configuration/selection-modes/discover)", "[`prioritize`](https://docs.redefine.dev/configuration/selection-modes/prioritize)", "[`fail-fast`](https://docs.redefine.dev/configuration/selection-modes/fail-fast)", "[`optimize`](https://docs.redefine.dev/configuration/selection-modes/optimize)".|`discover`|
|`python-venv-path`|Python virtual environment to install Redefine in.|-|
|`time-limit`|The [time limit](https://docs.redefine.dev/configuration/configuration-parameters#time-limit) for running Redefine.|-|
|`min-accuracy`|The [minimum accuracy](https://docs.redefine.dev/configuration/configuration-parameters#minumum-accuracy) for Redefine to achieve.|-|
|`confidence`|The [Confidence](https://docs.redefine.dev/configuration/configuration-parameters#confidence) level for Redefine's prediction.|-|
|`session-id`|The [session ID](https://docs.redefine.dev/configuration/remote-workers#configure-a-shared-session-id) matching between the orchestrator and the workers|-|
|`config-args`|[Additional arguments](https://docs.redefine.dev/configuration/configuration-parameters) to pass to the `config set` command in format of key1=value1 key2="long value2" ...|-|
|`split`|The number of machines to split the tests between, when using [Redefine Parallel](https://docs.redefine.dev/configuration/parallel-test-execution/redefine-parallel)|-|
|`group`|The machine index when using [Redefine Parallel](https://docs.redefine.dev/configuration/parallel-test-execution/redefine-parallel)|-|
|`args`|Additional arguments to pass to the redefine command.|-|                                                                                                           | -          |
## Examples

### Example usage for Github Actions

```yaml
name: Redefine

# Controls when the workflow will run
on:
  # Triggers the workflow on pull request events where "main" is the target (base) branch
  pull_request:
    branches: ["main"]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "pytest"
  pytest:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so follow-up steps can access it
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.11

      # Install python dependencies
      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Redefine Start
        id: redefine
        uses: redefinedev/redefine-action@main
        with:
          # Redefine authentication key given by Redefine in the subscription process, stored as a secret.
          auth: ${{ secrets.REDEFINE_AUTH }}
          # Python virtual environment path
          python-venv-path: .venv/
          # Testing framework to optimize
          testing-framework: pytest
          # Redefine mode as described in https://docs.redefine.dev/configuration/selection-modes
          mode: fail-fast
          # The maximum time to run tests with Redefine optimization
          time-limit: 300

      # Execute pytest tests with Redefine
      - name: Run pytest
        run: pytest -n auto tests/
```

Note that this example uses the latest version (`main`). It is recommended to use this version instead of a static one to avoid deprecation.

### Example usage for Github Actions with pipenv

```yaml
name: Redefine

# Controls when the workflow will run
on:
  # Triggers the workflow on pull request events where "main" is the target (base) branch
  pull_request:
    branches: ["main"]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "pytest"
  pytest:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so follow-up steps can access it
      - uses: actions/checkout@v3

      # Installs Python with pipenv cache
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.11
          cache: "pipenv"
          cache-dependency-path: "Pipfile.lock"

      # Installs pipenv and other dependencies
      - name: Install pipenv
        run: |
          python -m pip install --no-cache-dir --upgrade pipenv
          pipenv install --dev -v

      # Get VENV path
      - name: Get venv path
        id: get-venv
        run: |
          echo "venv_path=$(pipenv --venv)" >> $GITHUB_OUTPUT

      - name: Redefine Start
        id: redefine
        uses: redefinedev/redefine-action@main
        with:
          # Redefine authentication key given by Redefine in the subscription process, stored as a secret.
          auth: ${{ secrets.REDEFINE_AUTH }}
          # Python virtual environment path
          python-venv-path: ${{ steps.get-venv.outputs.venv_path }}
          # Testing framework to optimize
          testing-framework: pytest
          # Redefine mode as described in https://docs.redefine.dev/configuration/selection-modes
          mode: fail-fast
          # The maximum time to run tests with Redefine optimization
          time-limit: 300

      # Execute pytest tests with Redefine
      - name: Run pytest
        run: pytest -n auto tests/
```

### Example usage for Github Actions with poetry

```yaml
name: Redefine

# Controls when the workflow will run
on:
  # Triggers the workflow on pull request events where "main" is the target (base) branch
  pull_request:
    branches: ["main"]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "pytest"
  pytest:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so follow-up steps can access it
      - uses: actions/checkout@v3

      # Installs Python with poetry cache
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.11
          cache: "poetry"
          cache-dependency-path: "poetry.lock"

      # Installs poetry and other dependencies
      - name: Install poetry
        run: |
          curl https://install.python-poetry.org | python3 -
          echo "$HOME/.local/bin" >> $GITHUB_PATH
          poetry install

      # Get VENV path
      - name: Get venv path
        id: get-venv
        run: |
          echo "venv_path=$(poetry env info -p)" >> $GITHUB_OUTPUT

      - name: Redefine Start
        id: redefine
        uses: redefinedev/redefine-action@main
        with:
          # Redefine authentication key given by Redefine in the subscription process, stored as a secret.
          auth: ${{ secrets.REDEFINE_AUTH }}
          # Python virtual environment path
          python-venv-path: ${{ steps.get-venv.outputs.venv_path }}
          # Testing framework to optimize
          testing-framework: pytest
          # Redefine mode as described in https://docs.redefine.dev/configuration/selection-modes
          mode: fail-fast
          # The maximum time to run tests with Redefine optimization
          time-limit: 300

      # Execute pytest tests with Redefine
      - name: Run pytest
        run: pytest -n auto tests/
```

### Example usage for Github Actions with multiple machines in parallel

```yaml
name: Redefine

# Controls when the workflow will run
on:
  # Triggers the workflow on pull request events where "main" is the target (base) branch
  pull_request:
    branches: [ "main" ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "pytest"
  pytest:
    # Steps represent a sequence of tasks that will be executed as part of the job
    strategy:
      # Create a uniques index for each machine
      matrix:
        group: [ 0, 1, 2 ]
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so follow-up steps can access it
      - uses: actions/checkout@v3

      # Installs Python with poetry cache
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.11


      - name: Redefine Start
        id: redefine
        uses: redefinedev/redefine-action@main
        with:
            # Redefine authentication key given by Redefine in the subscription process, stored as a secret.
            auth: ${{ secrets.REDEFINE_AUTH }}
            # Testing framework to optimize
            testing-framework: pytest
            # Redefine mode as described in https://docs.redefine.dev/configuration/selection-modes
            mode: optimize
            # The maximum time to run tests with Redefine optimization
            time-limit: 300
            # Create a unique session ID for the workers to use
            # The session ID should be unique for each run
            session-id: ${{ github.run_id }}_${{ github.run_attempt }}
            # number of machines to split the tests, in this example 3
            split: 3
            # the machine "index"
            group: ${{ matrix.group }}


      # Execute pytest tests with Redefine
      # Redefine will distribute the tests between the machines
      - name: Run pytest dry run
        run: |
          pytest tests/

<!-- ### Example usage for Github Actions optimized PR with feedback from main -->
```
