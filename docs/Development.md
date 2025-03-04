# Development #

This page has summary information about developing the PyPDF library.

[TOC]

## History ##

This project, `fpdf2` is a _fork_ of the `PyFPDF` project, which can be found
[on GitHub at reingart/pyfpdf](https://github.com/reingart/pyfpdf).

Its aim is to keep the library up to date, to fulfill the goals of its
[original roadmap](https://github.com/reingart/pyfpdf/wiki/Roadmap) and provide
a general overhaul of the codebase to address technical debt keeping features from being
added and bugs to be eradicated.

More on `PyFPDF`:

> This project started as Python fork of the [FPDF](http://fpdf.org/) PHP library. 
> Later, code for native reading TTF fonts was added. FPDF has not been updated since
> 2011. See also the [TCPDF](http://www.tcpdf.org/) library.
> 
> Until 2015 the code was developed at [Google Code](https://code.google.com/p/pyfpdf/).
> Now the main repository is at [Github](https://github.com/reingart/pyfpdf).
> 
> You can also view the
> [old repository](https://github.com/reingart/pyfpdf_googlecode),
> [old issues](https://github.com/reingart/pyfpdf_googlecode/issues), and 
> [old wiki](https://github.com/reingart/pyfpdf_googlecode/tree/wiki).


## Usage ##

- [PyPI download stats](https://pypistats.org/packages/fpdf2)
- packages using `fpdf2` can be listed using [GitHub Dependency graph: Dependents](https://github.com/PyFPDF/fpdf2/network/dependents),
[Wheelodex](https://www.wheelodex.org/projects/fpdf2/rdepends/) or [Watchman Pypi](http://www.watchman-pypi.com).
Some are also listed on [its libraries.io page](https://libraries.io/pypi/fpdf2).


## Repository structure ##

  * `[.github]` - GitHub Actions configuration
  * `[docs]` - documentation folder
  * `[fpdf]` - library source
  * `[scripts]` - manipulate this repository
  * `[test]` - tests
  * `[tutorial]` - tutorials (see also [Tutorial](Tutorial.md))
  * `README.md` - Github and PyPI ReadMe
  * `CHANGELOG.md` - details of each release content
  * `LICENSE` - code license information
  * `CODEOWNERS` - define individuals or teams responsible for code in this repository
  * `CONTRIBUTORS.md` - the people who helped build this library ❤️
  * `setup.cfg`, `setup.py`, `MANIFEST.in` - packaging configuration to publish [a package on Pypi](https://pypi.org/project/fpdf2/)
  * `mkdocs.yml` - configuration for [MkDocs](https://www.mkdocs.org/)
  * `tox.ini` - configuration for [Tox](https://tox.readthedocs.io/en/latest/)
  * `.pylintrc` - configuration for [Pylint](http://pylint.pycqa.org/en/latest/)

## Code auto-formatting ##

We use [black](https://github.com/psf/black) as a code prettifier.
This _"uncomprimising Python code formatter"_ must be installed
in your development environment (`pip install black`) in order to
auto-format source code before any commit.

## Linting ##

We use [pylint](https://github.com/PyCQA/pylint/) as a static code analyzer
to detect potential issues in the code.

In case of special "false positive" cases,
checks can be disabled locally with `#pylint disable=XXX` code comments,
or globally through the `.pylintrc` file.

## Pre-commit hook ##
If you use a UNIX system, you can place the following shell code
in `.git/hooks/pre-commit` in order to always invoke `black` & `pylint`
before every commit:

```shell
#!/bin/bash
git_cached_names() { git diff --cached --name-only --diff-filter=ACM; }
if grep -IRF generate=True $(git_cached_names | grep 'test.*\.py$'); then
    echo '`generate=True` left remaining in a call to assert_pdf_equal'
    exit 1
fi
modified_py_files=$(git_cached_names | grep '\.py$')
modified_fpdf_files=$(git_cached_names | grep '^fpdf.*\.py$')
# If any Python files were modified, format them:
if [ -n "$modified_py_files" ]; then
    if ! black --check $modified_py_files; then
        black $modified_py_files
        exit 1
    fi
    # If fpdf/ files were modified, lint them:
    [[ $modified_fpdf_files == "" ]] || pylint $modified_fpdf_files
fi
```

It will abort the commit if `pylint` found issues
or `black` detect non-properly formatted code.
In the later case though, it will auto-format your code
and you will just have to run `git commit -a` again.

## Testing ##

To run tests, ``cd`` into the repository, install the dependencies using
``pip install -r test/requirements.txt``,  and run `pytest`.

Alternatively, you can use [Tox](https://tox.readthedocs.io/en/latest/).
It is self-documented in the `tox.ini` file in the repository.
To run tests for all versions of Python, simply run `tox`.
If you do not want to run tests for all versions of python, run `tox -e py39`
(or your version of Python).

When a unit test generates a PDF, it is recommended to use the `assert_pdf_equal`
utility function in order to validate the output.
It relies on the very handy [qpdf](https://github.com/qpdf/qpdf) CLI program
to generate a PDF that is easy to compare: annotated, strictly formatted,
with uncompressed internal streams.
You will need to have its binary in your `$PATH`,
otherwise `assert_pdf_equal` will fall back to hash-based comparison.

All generated PDF files (including those processed by `qpdf`) will be stored in
`/tmp/pytest-of-USERNAME/pytest-current/NAME_OF_TEST/`. By default, three
last test runs will be saved and then automatically deleted, so you can
check the output in case of a failed test.

In order to generate a "reference" PDF file, simply call `assert_pdf_equal`
once with `generate=True`.

Be sure to see the example tests in the `test/` folder in general.

## GitHub pipeline ##

A [GitHub Actions](https://help.github.com/en/actions/reference) pipeline
is executed on every commit on the `master` branch,
and for every _Pull Request_.

It performs all validation steps detailed above: code checking with `black`,
linting with `pylint`, unit tests...
_Pull Requests_ submitted must pass all those checks in order to be approved.
Ask maintainers through comments if some errors in the pipeline seem obscure to you.

### Release checklist ###

1. complete `CHANGELOG.md` and add the version & date of the new release
2. bump `FPDF_VERSION` in `fpdf/fpdf.py`
3. `git commit` & `git push`
4. check that [the GitHub Actions succeed](https://github.com/PyFPDF/fpdf2/actions), and that [a new release appears on Pypi](https://pypi.org/project/fpdf2/#history)
5. perform a [GitHub release](https://github.com/PyFPDF/fpdf2/releases), taking the description from the `CHANGELOG.md`.
It will create a new `git` tag.

## Documentation ##

The standalone documentation is in the `docs` subfolder,
written in [Markdown](https://daringfireball.net/projects/markdown/).
Building instructions are contained in the configuration file `mkdocs.yml`
and also in `.github/workflows/continuous-integration-workflow.yml`.

Additional documentation is generated from inline comments, and is available
in the project [home page](https://pyfpdf.github.io/fpdf2/fpdf/).

After being committed to the master branch, code documentation is automatically uploaded to
[GitHub Pages](https://pyfpdf.github.io/fpdf2/).

There is a useful one-page example Python module with docstrings illustrating how to document code:
[pdoc3 example_pkg](https://github.com/pdoc3/pdoc/blob/master/pdoc/test/example_pkg/__init__.py).

To preview the Markdown documentation, launch a local rendering server with:

    mkdocs serve

To preview the API documentation, launch a local rendering server with:

    pdoc --html -o public/ fpdf --http :

## See also ##
[Project Home](index.md), [Frequently asked questions](FAQ.md), 
[Unicode](Unicode.md).
