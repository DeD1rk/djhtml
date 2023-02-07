# DjHTML

***A pure-Python Django/Jinja template indenter without dependencies.***

DjHTML is a fully automatic template indenter that works with mixed
HTML/CSS/Javascript templates that contain
[Django](https://docs.djangoproject.com/en/stable/ref/templates/language/)
or [Jinja](https://jinja.palletsprojects.com/templates/) template
tags. It works similar to other code-formatting tools such as
[Black](https://github.com/psf/black) and interoperates nicely with
[pre-commit](https://pre-commit.com/).

DjHTML is an _indenter_ and not a _formatter_: it will only add/remove
whitespace at the beginning of lines. It will not insert newlines or
other characters. The goal is to correctly indent already
well-structured templates, not to fix broken ones.

For example, consider the following incorrectly indented template:

```jinja
<!doctype html>
<html>
    <body>
        {% block content %}
        Hello, world!
        {% endblock %}
        <script>
            $(function() {
            console.log('Hi mom!');
            });
        </script>
    </body>
</html>
```

This is what it will look like after processing by DjHTML:

```jinja
<!doctype html>
<html>
    <body>
        {% block content %}
            Hello, world!
        {% endblock %}
        <script>
            $(function() {
                console.log('Hi mom!');
            });
        </script>
    </body>
</html>
```


## Installation

DjHTML is compatible with all operating systems supported by Python.
Install DjHTML with the following command:

    $ pip install djhtml

Note that [Windows still uses legacy code pages for the system
encoding](https://docs.python.org/3/using/windows.html#win-utf8-mode).
It is highly advised to set the environment variable `PYTHONUTF8` to
`1` to avoid issues with indenting UTF-8 files. You can do so with the
[setx](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/setx)
command:

    C:\> setx /m PYTHONUTF8 1


## Usage

After installation you can indent templates using the `djhtml`
command:

    $ djhtml template.html
    reindented template.html
    1 template has been reindented.

An exit status of 0 means that everything went well, regardless of
whether any files were changed. When the option `-c` / `--check` is
used, the exit status is 1 when one or more files would have changed,
but no changes are actually made. The exit status of 123 means that
there was an error while indenting one or more files. All available
options are given by `djthml -h` / `djthml --help`.


## `fmt:off` and `fmt:on`

You can exclude specific lines from being processed with the
`{# fmt:off #}` and `{# fmt:on #}` operators:

```jinja
<div class="
    {# fmt:off #}
      ,-._|\
     /     .\
     \_,--._/
    {# fmt:on #}
"/>
```

Contents inside `<pre> ... </pre>`, `<!-- ... --->`, `/* ... */`, and
`{% comment %} ... {% endcomment %}` tags are also ignored (depending
on the current mode).


## Modes

The indenter operates in one of three different modes:

- DjHTML mode: the default mode. Invoked by using the `djhtml` command
  or the pre-commit hook.

- DjCSS mode. Will be entered when a `<style>` tag is encountered in
  DjHTML mode. It can also be invoked directly with the command
  `djcss`.

- DjJS mode. Will be entered when a `<script>` tag is encountered in
  DjHTML mode. It can also be invoked directly with the command
  `djjs`.


## pre-commit configuration

The best way to use DjHTML is as a [pre-commit](https://pre-commit.com/)
hook, so all your HTML, CSS and JavaScript files will automatically be
indented upon every commit.

First, install pre-commit:

    $ pip install pre-commit
    $ pre-commit install

Then, add the following to your `.pre-commit-config.yaml`:

```yml
repos:
  - repo: https://github.com/rtts/djhtml
    rev: 'main'  # replace with the latest tag on GitHub
    hooks:
      - id: djhtml
      - id: djcss
      - id: djjs
```

Now run `pre-commit autoupdate` to automatically replace `main` with
the latest tag on GitHub,
[as recommended by pre-commit](https://pre-commit.com/#using-the-latest-version-for-a-repository).

If you want to override a command-line option, for example to change
the default tabwidth, you change the `entry` point of these hooks:

```yml
    hooks:
      - id: djhtml
        # Use a tabwidth of 2 for HTML files
        entry: djhtml --tabwidth 2
      - id: djcss
      - id: djjs
```

If you want to limit the files these hooks operate on, you can use
[pre-commit mechanisms for filtering](https://pre-commit.com/#filtering-files-with-types).
For example:

```yml
    hooks:
      - id: djhtml
        # Indent only HTML files in template directories
        files: .*/templates/.*\.html$
      - id: djcss
        # Run this hook only on SCSS files (CSS and SCSS is the default)
        types: [scss]
      - id: djjs
        # Exclude JavaScript files in vendor directories
        exclude: .*/vendor/.*
```

Now when you run `git commit` you will see something like the
following output:

    $ git commit

    DjHTML...................................................................Failed
    - hook id: djhtml
    - files were modified by this hook

    reindented template.html
    1 template has been reindented.

To inspect the changes that were made, use `git diff`. If you are
happy with the changes, you can commit them normally. If you are not
happy, please do the following:

1. Run `SKIP=djhtml git commit` to commit anyway, skipping the
   `djhtml` hook.

2. Consider [opening an issue](https://github.com/rtts/djhtml/issues)
   with the relevant part of the input file that was incorrectly
   formatted, and an example of how it should have been formatted.

Your feedback for improving DjHTML is very welcome!

## Development

Use your preferred system for setting up a virtualenv, docker environment,
or whatever else, then run the following:

```sh
python -m pip install -e '.[dev]'
pre-commit install --install-hooks
```

Tests can then be run quickly in that environment:

```sh
python -m unittest discover -v
```

Or testing in all available supported environments and linting can be run
with [`nox`](https://nox.thea.codes):

```sh
nox
```
