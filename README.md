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

|name|description|default|
|---|---|---|
|`auth`|Redefine authentication key given in by Redefine in the onboarding process|-|
|`testing-framework`|The [testing framework](https://docs.redefine.dev/integrations/supported-technologies) to optimize.|-|
|`command`|Redefine CLI command, one of "[`start`](https://docs.redefine.dev/configuration/start-command)", "[`verify`](https://docs.redefine.dev/welcome-to-redefine/quick-start#verify)", "[`get session_id`](https://docs.redefine.dev/configuration/remote-workers#configure-a-shared-session-id)", "[`get session_check`](https://docs.redefine.dev/configuration/remote-workers#session-check-command)".|`start`|
|`mode`|Redefine execution mode, on of "[`discover`](https://docs.redefine.dev/configuration/selection-modes/discover)", "[`prioritize`](https://docs.redefine.dev/configuration/selection-modes/prioritize)", "[`fail-fast`](https://docs.redefine.dev/configuration/selection-modes/fail-fast)", "[`optimize`](https://docs.redefine.dev/configuration/selection-modes/optimize)".|`discover`|
|`python-venv-path`|Python virtual environment to install Redefine in.|-|
|`time-limit`|The [time limit](https://docs.redefine.dev/configuration/time-limit) for running Redefine.|-|
|`min-accuracy`|The [minimum accuracy](https://docs.redefine.dev/configuration/minumum-accuracy) for Redefine to achieve.|-|
|`confidence-level`|The [Confidence Level](https://docs.redefine.dev/configuration/configuration-parameters#confidence) required for the run.|-|
|`session-id`|The [session ID](https://docs.redefine.dev/configuration/remote-workers#configure-a-shared-session-id) matching between the orchestrator and the workers|-|
|`config-args`|[Additional arguments](https://docs.redefine.dev/configuration/configuration-parameters) to pass to the `config set` command in format of key1=value1 key2="long value2" ...|-|
|`args`|Additional arguments to pass to the redefine command.|-|

## Examples
### Example usage for Github Actions

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
    branches: [ "main" ]

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
    branches: [ "main" ]

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

### Example usage for Github Actions with multiple workers
```yaml
name: Redefine

# Controls when the workflow will run
on:
  # Triggers the workflow on pull request events where "main" is the target (base) branch
  pull_request:
    branches: [ "main" ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # The orchestrator job, which performs the prediction and provides the workers with the optimized list
  orchestrator:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
    # outputs to pass information between jobs
    outputs:
      session-id: ${{ steps.redefine-session-id.outputs.session-id }}
      optimized-test-list: ${{ steps.redefine.outputs.tests }}

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so follow-up steps can access it
      - uses: actions/checkout@v3

      # Installs Python with poetry cache
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.11

      - name: Redefine Start
        id: redefine-session-id
        uses: redefinedev/redefine-action@main
        with:
            # Redefine authentication key given by Redefine in the subscription process, stored as a secret.
            auth: ${{ secrets.REDEFINE_AUTH }}
            # Redefine command
            command: "get session_id"

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
            # The session id previously generated
            session-id: ${{ steps.redefine-session-id.outputs.session-id }}
            # Use the --dry-run argument to only get the optimized list of tests
            args: --dry-run --output-path=./optimized-test-list.txt

      # Execute pytest to collect the optimized test list
      - name: Run pytest dry run
        run: |
          # Running pytest will not execute tests, instead outputting the optimized test list to file
          pytest tests/

          # Output the optimized list to the job's output to be available in the worker job
          echo "tests=$(cat ./optimized-test-list.txt) >> $GITHUB_OUTPUT


  # The worker jobs, executing the tests based on the optimized list above
  worker:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
    # This job depends on the orchestrator to finish
    needs: orchestrator
    # Matrix to give each worker an index
    strategy:
      matrix:
        # Running 8 workers
        index: [1,2,3,4,5,6,7,8]
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
            mode: worker
            # The *same* session id as used in the orchestrator and all workers
            session-id: ${{ needs.orchestrator.outputs.session-id }}

      # Execute pytest tests with Redefine
      - name: Run pytest
        run: |
          # Use the optimized list from the previous job
          TEST_LIST = ${{ needs.orchestrator.outputs.optimized-test-list }}
          # Prep num of workers and index for round-robin split algorithm.
          NUM_WORKERS = ${{ strategy.job-total }}
          # Can also use ${{ matrix.index }} instead of ${{ strategy.job-index }}
          INDEX = ${{ strategy.job-index }}

          # Split the tests in a round-robin algorithm according to the index, and concat with spaces for pytest
          TESTS=$(echo "$TEST_LIST" | awk "NR % $NUM_WORKERS == $INDEX" | tr '\n' ' ')

          # Execute the tests
          pytest -n auto $TESTS
```

Please note that for a test list larger than 1MB, artifacts need to be used to pass the test list between jobs instead of job output. For more details see [job output](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idoutputs)

<!-- ### Example usage for Github Actions optimized PR with feedback from main -->
