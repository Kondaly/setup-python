# setup-python V2

<p align="left">
  <a href="https://github.com/actions/setup-python"><img alt="GitHub Actions status" src="https://github.com/actions/setup-python/workflows/Main%20workflow/badge.svg"></a>
</p>

This action sets up a Python environment for use in actions by:

- optionally installing and adding to PATH a version of Python that is already installed in the tools cache
- downloading, installing and adding to PATH an available version of Python from GitHub Releases ([actions/python-versions](https://github.com/actions/python-versions/releases)) if a specific version is not available in the tools cache
- failing if a specific version of Python is not preinstalled or available for download
- registering problem matchers for error output

# What's new

- Ability to download, install and set up Python packages from `actions/python-versions` that do not come preinstalled on runners
  - Allows for pinning to a specific patch version of Python without the worry of it ever being removed or changed
- Automatic setup and download of Python packages if using a self-hosted runner
- Support for pre-release versions of Python

# Usage

See [action.yml](action.yml)

Basic:
```yaml
steps:
- uses: actions/checkout@v2
- uses: actions/setup-python@v2
  with:
    python-version: '3.x' # Version range or exact version of a Python version to use, using SemVer's version range syntax
    architecture: 'x64' # optional x64 or x86. Defaults to x64 if not specified
- run: python my_script.py
```

Matrix Testing:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [ '2.x', '3.x', 'pypy2', 'pypy3' ]
    name: Python ${{ matrix.python-version }} sample
    steps:
      - uses: actions/checkout@v2
      - name: Setup python
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
          architecture: x64
      - run: python my_script.py
```

Exclude a specific Python version:
```yaml
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: [2.7, 3.6, 3.7, 3.8, pypy2, pypy3]
        exclude:
          - os: macos-latest
            python-version: 3.8
          - os: windows-latest
            python-version: 3.6
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
      - name: Display Python version
        run: python -c "import sys; print(sys.version)"
```

Download and set up a version of Python that does not come preinstalled on an image:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
        # in this example, there is a newer version already installed, 3.7.7, so the older version will be downloaded
        python-version: [3.5, 3.6, 3.7.4, 3.8]
    steps:
    - uses: actions/checkout@v2
    - uses: actions/setup-python@v2
      with:
        python-version: ${{ matrix.python-version }}
    - run: python my_script.py

```

Download and set up a accurate pre-release version of Python:
```yaml
steps:
- uses: actions/checkout@v2
- uses: actions/setup-python@v2
  with:
    python-version: '3.9.0-beta.4'
- run: python my_script.py
```

Download and set up the latest available version of Python (includes both pre-release and stable versions):
```yaml
steps:
- uses: actions/checkout@v2
- uses: actions/setup-python@v2
  with:
    python-version: '3.9.0-alpha - 3.9.0' # SemVer's version range syntax
- run: python my_script.py
```

# Getting started with Python + Actions

Check out our detailed guide on using [Python with GitHub Actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/using-python-with-github-actions).

# Available versions of Python

`setup-python` is able to configure Python from two sources:

- Preinstalled versions of Python in the tools cache on GitHub-hosted runners
    - For detailed information regarding the available versions of Python that are installed see [Software installed on GitHub-hosted runners](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/software-installed-on-github-hosted-runners).
    - For every minor version of Python, expect only the latest patch to be preinstalled.
    - If `3.8.1` is installed for example, and `3.8.2` is released, expect `3.8.1` to be removed and replaced by `3.8.2` in the tools cache.
    - If the exact patch version doesn't matter to you, specifying just the major and minor version will get you the latest preinstalled patch version. In the previous example, the version spec `3.8` will use the `3.8.2` Python version found in the cache.
- Downloadable Python versions from GitHub Releases ([actions/python-versions](https://github.com/actions/python-versions/releases))
    - All available versions are listed in the [version-manifest.json](https://github.com/actions/python-versions/blob/main/versions-manifest.json) file.
    - If there is a specific version of Python that is not available, you can open an issue here

# Hosted Tool Cache

GitHub hosted runners have a tools cache that comes with a few versions of Python + PyPy already installed. This tools cache helps speed up runs and tool setup by not requiring any new downloads. There is an environment variable called `RUNNER_TOOL_CACHE` on each runner that describes the location of this tools cache and there is where you will find Python and PyPy installed. `setup-python` works by taking a specific version of Python or PyPy in this tools cache and adding it to PATH.

|| Location |
|------|-------|
|**Tool Cache Directory** |`RUNNER_TOOL_CACHE`|
|**Python Tool Cache**|`RUNNER_TOOL_CACHE/Python/*`|
|**PyPy Tool Cache**|`RUNNER_TOOL_CACHE/PyPy/*`|

GitHub virtual environments are setup in [actions/virtual-environments](https://github.com/actions/virtual-environments). During the setup, the available versions of Python and PyPy are automatically downloaded, setup and documented.
- [Tools cache setup for Ubuntu](https://github.com/actions/virtual-environments/blob/master/images/linux/scripts/installers/hosted-tool-cache.sh)
- [Tools cache setup for Windows](https://github.com/actions/virtual-environments/blob/master/images/win/scripts/Installers/Download-ToolCache.ps1)

# Specifying a Python version

If there is a specific version of Python that you need and you don't want to worry about any potential breaking changes due to patch updates (going from `3.7.5` to `3.7.6` for example), you should specify the exact major, minor, and patch version (such as `3.7.5`)
  - The only downside to this is that set up will take a little longer since the exact version will have to be downloaded if the exact version is not already installed on the runner due to more recent versions.
  - MSI installers are used on Windows for this, so runs will take a little longer to set up vs Mac and Linux.

You should specify only a major and minor version if you are okay with the most recent patch version being used.
  - There will be a single patch version already installed on each runner for every minor version of Python that is supported.
  - The patch version that will be preinstalled, will generally be the latest and every time there is a new patch released, the older version that is preinstalled will be replaced.
  - Using the most recent patch version will result in a very quick setup since no downloads will be required since a locally installed version Python on the runner will be used.

# Using `setup-python` with a self hosted runner

Python distributions are only available for the same [environments](https://github.com/actions/virtual-environments#available-environments) that GitHub Actions hosted environments are available for. If you are using an unsupported version of Ubuntu such as `19.04` or another Linux distribution such as Fedora, `setup-python` will not work. If you have a supported self-hosted runner and you would like to use `setup-python`, there are a few extra things you need to make sure are set up so that new versions of Python can be downloaded and configured on your runner.

If you are experiencing problems while configuring Python on your self-hosted runner, turn on [step debugging](https://github.com/actions/toolkit/blob/main/docs/action-debugging.md#step-debug-logs) to see addition logs.

### Windows

- Your runner needs to be running with administrator privileges so that the appropriate directories and files can be set up when downloading and installing a new version of Python for the first time.
- If your runner is configured as a service, make sure the account that is running the service has the appropriate write permissions so that Python can get installed. The default `NT AUTHORITY\NETWORK SERVICE` should be sufficient.
- You need `7zip` installed and added to your `PATH` so that the downloaded versions of Python files can be extracted properly during first-time setup.
- MSI installers are used when setting up Python on Windows. A word of caution as MSI installers update registry settings.
- The 3.8 MSI installer for Windows will not let you install another 3.8 version of Python. If `setup-python` fails for a 3.8 version of Python, make sure any previously installed versions are removed by going to "Apps & Features" in the Settings app.

### Linux

- The Python packages that are downloaded from `actions/python-versions` are originally compiled from source in `/opt/hostedtoolcache/` with the [--enable-shared](https://github.com/actions/python-versions/blob/94f04ae6806c6633c82db94c6406a16e17decd5c/builders/ubuntu-python-builder.psm1#L35) flag, which makes them non-relocatable.
- Create an environment variable called `AGENT_TOOLSDIRECTORY` and set it to `/opt/hostedtoolcache`. This controls where the runner downloads and installs tools.
  - In the same shell that your runner is using, type `export AGENT_TOOLSDIRECTORY=/opt/hostedtoolcache`
  - A more permanent way of setting the environment variable is to create a `.env` file in the same directory as your runner and to add `AGENT_TOOLSDIRECTORY=/opt/hostedtoolcache`. This ensures the variable is always set if your runner is configured as a service.
- Create a directory called `hostedtoolcache` inside `/opt`.
- The user starting the runner must have write permission to the `/opt/hostedtoolcache` directory. It is not possible to start the Linux runner with `sudo` and the `/opt` directory usually requires root privileges to write to. Check the current user and group that the runner belongs to by typing `ls -l` inside the runners root directory.
- The runner can be granted write access to the `/opt/hostedtoolcache` directory using a few techniques:
  - The user starting the runner is the owner, and the owner has write permission
  - The user starting the runner is in the owning group, and the owning group has write permission
  - All users have write permission
- One quick way to grant access is to change the user and group of `/opt/hostedtoolcache` to be the same as the runners using `chown`
    - `sudo chown runner-user:runner-group opt/hostedtoolcache/`
- If your runner is configured as a service and you run into problems, make sure the user that the service is running as is correct. For more information, you can [check the status of your self-hosted runner](https://help.github.com/en/actions/hosting-your-own-runners/configuring-the-self-hosted-runner-application-as-a-service#checking-the-status-of-the-service).

### Mac

- The same setup that applies to `Linux` also applies to `Mac`, just with a different tools cache directory.
- Create a directory called `/Users/runner/hostedtoolcache`
- Set the `AGENT_TOOLSDIRECTORY` environment variable to `/Users/runner/hostedtoolcache`.
- Change the permissions of `/Users/runner/hostedtoolcache` so that the runner has write access.


# Using Python without `setup-python`

`setup-python` helps keep your dependencies explicit and ensures consistent behavior between different runners. If you use `python` in a shell on a GitHub hosted runner without `setup-python` it will default to whatever is in PATH. The default version of Python in PATH vary between runners and can change unexpectedly so we recommend you always use `setup-python`.

# License

The scripts and documentation in this project are released under the [MIT License](LICENSE)

# Contributions

Contributions are welcome! See our [Contributor's Guide](docs/contributors.md)
