# mps4c - Module / Package System for C

`mps4c` is module/package system for C

## Workspace

`.workspace.ft` file indicates current directory is root of the workspace.

### Directory Structure

workspace root directory consists of the following:

- `packages/` - directory containing all installed mps4c packages
  - _(directories with the same name as each package)_
    - `src/` - directory including source files
    - `build/` - build related files
    - `.package.ft` - indicates current directory is root of the package
    - `dependencies.ft` - list of dependent packages
    - `Makefile` - recipes to build the package
- `projects/` - directory containing your repositories
  - _(any depth of directories, may contain any other files)_
    - `modules/` - directory containing modules in the project (package)
      - _(any depth of directories, **MAY NOT CONTAIN OTHER FILES**)_
        - _(module directory name)_ - [see _Module directory name_ below](#module-directory-name)
    - `.project.ft` - indicates current directory is root of the project
    - `Makefile` - recipe to build the `package` directory in form above
    - ... and more like `.editorconfig`, `.gitignore`, `compile_flags.txt`, ...
- `tmp/` - temporary directory for all packages/projects
- `.workspace.ft` - indicates current directory is root of the workspace

### Init workspace

using mps4c cli

```shell
mps4c init
```

or manually configure

```shell
mkdir -p packages projects tmp
touch .workspace.ft
```

## Module

Each module is _**header module**_, _**source module**_, or _bundle module_.

### Dependency

Some module requires other module to be loaded.

For example, included header module `h.ft_args` is dependency for the source module.

...and included header module `h.ft_args__types` which is included by `h.ft_args`.

The mps4c module system will load recursively all dependencies automatically.

### Peer Dependency

Some function calls other function (source module) but not require exact implementation.

In this case, the exact implementation can be specified by [module edition](#module-edition).

This type of dependency is called a peer dependency.

Peer dependencies will not be automatically loaded while dependencies are loaded.

### Module type

The module is one of the three below:

- Header module (single header file, no [edition](#module-edition) allowed)
- Source module (must have only one function)
- Bundle module (just bundle other modules)

### Module name

Module name is concatenated form of two parts separated by double underscores(`__`).

The two parts are _package name_ which is required and _module name part_ which is optional.

The _Word_ consists of lowercase letters and digits. (`[a-z0-9]`)

The _Word_ can't contain consecutive underscores, or begin with a digit.

The _module name part_ is one or more _Word_ separated by underscore(`_`).

If the package name is `ft_args`, the module name can be `ft_args` or `ft_args__free`.

### Module edition

Edition is exact implementation of source module, while module name is interface.

Some module requires other source module, with no exact implementation specified.

In this case, the module requires module with ANY edition with given module name.

### Header module

Header module has two files:

- `dependencies.ft` - list of **DIRECTLY** included header module
- _(module name)_`.h` - actual content of the header module

Indirectly included header modules will be loaded automatically. Don't worry!

### Source module

Source module has two files:

- `dependencies_lib.ft` - list of name of libraries which will called (optional)
- `dependencies.ft` - list of included module and called&&exact function.
- `peer_dependencies.ft` - list of called function modules' name
- _(module name)_`.c` - actual content of the source module

A source module must have _**ONLY ONE** non-static function_ to keep it searchable.

### Bundle module

Bundle module just makes other dependencies as like only one dependency.

Sometimes, some dependencies may be required in a lot of modules.

To prevent list all dependencies in a lot of modules, you can use the bundle module.

### Module directory name

Module directory name is concatenated form of three parts separated by dot(`.`).

The first part is module type:

- `h` for header module,
- `s` for source module,
- `b` for bundle module.

The second part is _module name part_ above

The third part is module edition name which is optional.

If the package name is `ft_args`, the directory name can be `h.`, `s.alloc.w`, `b.a.w`

- `h./` - include `ft_args.h`
- `s.alloc.w/` - include `ft_args__alloc.c`

## Package

Each package has executables, libraries, and source files

## Future Plans

- remove _package name prefix_ (auto prefix)
- remove dependency file (auto collect dependencies from source code)
- remove directory per module
- remove filename restrictions (auto collect from source file)
