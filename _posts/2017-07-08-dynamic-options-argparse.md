---
layout: post
title: How to declare dynamic options with argparse
tag:
    - Python
---

While working on a project to generate docker images based on a matrix combinations ([https://github.com/Lothiraldan/docker-jinja2-builder](https://github.com/Lothiraldan/docker-jinja2-builder)), I needed to declare some options defined in a user-defined configuration file.

A matrix in the project will look like:

```yaml
matrix:
  python_version:
    - 2.7
    - 3.4
    - 3.5
    - 3.6
options:
    local_wheel:
        include_file: True
    test_option:
image_name: python_integration_tests_{ID}
image_id: "{python_version}"
```

The tool can be called against this matrix with: `python builder.py .` when the matrix is defined in the current directory.

The options defined in the matrix must be usable from the CLI but are unknown until the user pass the directory. I wanted to being able to do:

```python
python builder.py --test-option .
```

Or:

```python
python builder.py --local_wheel=project.whl .
```

Luckily for me, I'm using argparse and it's powerfull and flexible enought to cope with this need.

# Need

My argparse configuration was very simple before:

```python
parser = argparse.ArgumentParser(
    description='Build multiple docker images with Jinja2')
parser.add_argument(
    'base_path',
    action='store',
    help='A directory where a matrix.yml file and a Dockerfile.jinja2 can be find')

args = parser.parse_args()
print("Args:", args)
```

Passing a directory was working as expected:

```bash
python builder.py .
Args: Namespace(base_path='.')
```

But passing additional options failed:

```bash
python builder.py --local_wheel=project.whl .
usage: builder.py [-h] base_path
builder.py: error: unrecognized arguments: --local_wheel=project.whl
```

# Credits

After trying several solution, I made a quick search in my favorite search engine for [`dynamic options argparse`](https://duckduckgo.com/?q=dynamic+options+argparse&ia=qa) and quickly found [this interesting solution](http://stackoverflow.com/questions/25317795/dynamic-arguments-for-pythons-argparse#25320537).

It was easy enough to modify my code to ignore unknown options with [`parse_known_args`]():

```python
parser = argparse.ArgumentParser(
    description='Build multiple docker images with Jinja2')
parser.add_argument(
    'base_path',
    action='store',
    help='A directory where a matrix.yml file and a Dockerfile.jinja2 can be find')

known_args = parser.parse_known_args()
print("Known args:", known_args)
```

The output of the method changed a bit:

```bash
python builder.py --local_wheel=project.whl .
Known args: (Namespace(base_path='.'), ['--local_wheel=project.whl'])
```

`parse_known_args` returns a tuple of the parsed arguments it known and the rest of the options. Having access now to the base_path, we should parse the matrix file, extract the options and declare the new arguments:

```python
base_path = known_args[0].base_path

with open(join(base_path, 'matrix.yml')) as matrix_file:
    matrix = yaml.load(matrix_file.read())

# Get options for updating the arg parser
options = matrix.get('options', {})
for option_name, option in options.items():
    # Check if it the option is a file
    if option and 'include_file' in option:
        parser.add_argument(
            '--{}'.format(option_name),
            dest=option_name,
            action='store'
        )
    else:
        parser.add_argument(
            '--{}'.format(option_name),
            dest=option_name,
            action='store_true',
        )
```

Finally, we just ask argparse to reparse all arguments using the new options defined:

```python
args = parser.parse_args()
print("Args:", args)
```

```
python builder.py .
Args: Namespace(base_path='.', local_wheel=None, test_option=False)
```

```
python builder.py --local_wheel=project.whl .
Args: Namespace(base_path='.', local_wheel='project.whl', test_option=False)
```

```
python builder.py --test-option .
Args: Namespace(base_path='.', local_wheel=None, test_option=True)
```
