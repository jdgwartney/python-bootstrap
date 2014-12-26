python-bootstrap
================

Python script that creates a python environment from existing environment. This permits the cloning of a python environment and ability to add/upgrade python modules using PIP without disturbing the global python environment.

Creates 2.X environment but can be configured to clone a python 3.x evironment as well.

Script was created to make the install experience better for Boundary Plugins that use python. The script only uses the Python standard library so it should be portable across all environments including Windows.

Prerequisites
-------------
- Python 2.6.6 or later
- Network Access - require to download virtualenv distribution

Caveats
--------
- If you attempt to uninstall a module using PIP using the new python environment make sure that the `PYTHONPATH` environment variable is unset or an error will ocurr indicating that you cannot uninstall a module in an external environment.

### Cloning an Python Environment

Creation of a new Python in environment is simple, just run the following:
```bash
$ bootstrap.py
```

### Installing new modules in the new python environment

Run the following to add a module in the cloned python environment:

```bash
$ pyenv/bin/pip <module name>
```

### Including in a Boundary Plugin
To ease installation of Boundary Plugins that use Python and third-party modules `bootstrap.py` can be called during the post-extract phase to create a new python environment and other dependent third-party modules

#### Creating a new python environment
Include the `bootstrap.py` script in your plugin's GitHub repository and then modify `plugin.json` to:

- To call the `boostrap.py` python script using `postExtract`
- Ignore the python distribution `pyenv`, otherwise the meter will overwrite
- Use the cloned environment

1. Add the following to the `plugin.json` to execute a `postExtract` script:
```json
"postExtract" : "python post-extract.py",
```
2. Add the following to the `plugin.json` to ignore the new environment:
```json
"ignore" : "pyenv",
```
3. Modify `plugin.json` to use new python environment when running a python plugin script:
```json
"command" : "pyenv/bin/python plugin.py",
```

#### Installing other dependent modules declaratively
`bootstrap.py` by default looks for the existence of a file named `requirements.txt` which includes additional modules to install using PIP.

Content for requirements file can be generated from an existing python environment via the following:
```bash
$ pip freeze
boto==2.34.0
virtualenv==12.0.4
ystockquote==0.2.4
```

Include the output in the `requirements.txt` file and place in the same directory as `bootstrap.py`:
```bash
boto==2.34.0
virtualenv==12.0.4
ystockquote==0.2.4
```

After the environment is cloned, the other modules in `requirments.txt` will be installed using PIP.

### Using within in your own Post-Extract script:

The following is an example of how to use `boostrap` in your own post-extract python script:
```python
#!/usr/bin/env python
# Import the Bootstrap class
from bootstrap import Bootstrap

if __name__ == "__main__":
  # Create a new instance of Boostrap and call `setup` to create a python environment
  bootstrap = Bootstrap()
  # call setup() to create a new python environment
  bootstrap.setup()
```

The `Boostrap` constructor contains the following variables that can be overrident via its constructor:

- version - version of `virtualenv` module to install, default `12.04`
- base - base url to download the virtualenv module, default `http://pypi.python.org/packages/source/v/virtualenv`
- python - initial python interpreter to call to clone the environment, default `python2`
- env - path to the new python environment, defaults to `pyenv`
- requirements - path to requirements file, defaults to `requirements.txt`

Example:
```python
bootstrap = Bootstrap(env="mypython") # Override directory name of the new environment
```





