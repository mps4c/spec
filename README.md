# mps4c - Module / Package System for C

`mps4c` is module/package system for C

## Workspace

Every project, build cache, installed packages, ... will be stored in workspace.

`.workspace.ft` file indicates current directory is root of the workspace.

### Directory Structure

workspace root directory consists of the following:

- `packages/` - directory containing all installed mps4c packages
  - _(directories with the same name as each package)_
    - `build/` - build related files, such as shell scripts
    - `include/` - directory including **PUBLIC** header files
    - `src/` - directory including source files
    - `.package.ft` - indicates current directory is root of the package
    - `dependencies.ft` - list of dependent packages
    - `Makefile` - recipes to build the package
- `projects/` - directory containing your repositories
  - _(any depth of directories, may contain any other files)_
    - `modules/` - directory containing modules in the project (package)
      - _(any depth of directories, **MAY NOT CONTAIN OTHER FILES**)_
        - _(module directory name)_ - [see _Module directory name_ below](#module-directory-name)
          - modules files, depending on [module type](#module-type)
    - `.project.ft` - indicates current directory is root of the project
    - `Makefile` - recipe to build the `package` directory in form above
    - ... and more like `.editorconfig`, `.gitignore`, `compile_flags.txt`, ...
- `tmp/` - temporary directory for all packages/projects
- `.workspace.ft` - indicates current directory is root of the workspace

## `mps4c` command

The `mps4c` command has functions to handle workspaces, projects, ... etc.

### Activating

Some mps4c implementation may require to activate the command.

Way to activate the command depends on the implementation.

### Init workspace

You can init the workspace to existing, empty directory using command below.

```shell
mps4c init
# or
mps4c -C /path/to/workspace/directory init
```

### Clean workspace

`mps4c clean` removes cache for the implementation

(each implementation must have its own cache directory)

`mps4c fclean` removes entire cache and even output files.

### Build flags

Print include directory parameter, like `-I include -I /workspace/tmp/include`

```shell
$CC -o $TARGET $CPPFLAGS $(mps4c include_directories) $CFLAGS -c $SOURCE
```

Print library directory parameter, like `-L ../../lib -L /workspace/tmp/lib`

```shell
$CC -o out.exe $LDFLAGS $(mps4c library_directories) $OBJS $LDLIBS
```

Print libraries to link, in correct topological order, like `$(LDLIBS)`

```shell
$CC -o out.exe $LDFLAGS $OBJS $(mps4c ldlibs)
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

The two parts are [_package name part_](#package-name) which is required and _module name part_ which is optional.

The _Word_ consists of lowercase letters and digits. (`[a-z0-9]`)

The _module name part_ is one or more _Word_ separated by underscore(`_`).

If the package name is `ft_args`, possible module name can be like below:

- `ft_args`
- `ft_args__free`
- `ft_args__get_map`

### Module edition

Edition is exact implementation of source module, while module name is interface.

Some module requires other source module, with no exact implementation specified.

In this case, the module requires module with ANY edition with given module name.

### Header module

Header module has two files:

- `dependencies_lib.ft` - list of library name contains **DIRECTLY** included header (optional)
- `dependencies.ft` - list of **DIRECTLY** included header module (optional)
- _(module name)_`.h` - actual content of the header module (required)

Indirectly included header modules will be loaded automatically. Don't worry!

### Source module

Source module has two files:

- `dependencies_lib.ft` - list of name of libraries which will called (optional)
- `dependencies.ft` - list of included / called(exact) modules (exact) (optional)
- `peer_dependencies.ft` - list of called function modules' name (optional)
- _(module name)_`.c` - actual content of the source module (required)

A source module must have _**ONLY ONE** non-static function_ to keep it searchable.

### Bundle module

Bundle module just makes other dependencies as like only one dependency.

Sometimes, some dependencies may be required in a lot of modules.

To prevent list all dependencies in a lot of modules, you can use the bundle module.

Bundle module has only one file: `dependencies.ft` - list of dependencies to bundle

### Module directory name

Module directory name is concatenated form of three parts separated by dot(`.`).

The first part is module type:

- `h` for header module,
- `s` for source module,
- `b` for bundle module.

The second part is _module name part_ above

The third part is module edition name which is optional.

If the package name is `ft_args`, the directory name can be like below:

- `h./` - include `ft_args.h`
- `s./` - include `ft_args.c`
- `h.internal/` - include `ft_args__internal.h`
- `s.get_map` - include `ft_args__get_map.c`

## Package

Each package has executables, libraries, and source files

### Package type

The package is one of the two below:

- Normal package which including libraries and/or executables
- Mock package for testing purpose which including libraries

If the package name contains [_package name part_](#package-name), then the package is Mock package.

### Package name

Package name is concatenated form of two parts separated by period(`.`).

The two parts are _package name part_ which is required and and _package edition part_.

_package name part_ / _package edition part_ are one or more _Word_ separated by underscore(`_`).

But _package name part_ has extra constraint: can't starts with digit.

If some Mock package is for Normal package `memory`,

Possible names for the Mock package are: `memory.branch`, `memory.leak` and so on.

### Normal package

// TODO:

### Mock package

## Future plans

- remove _package name prefix_ (auto prefix)
- remove dependency file (auto collect dependencies from source code)
- remove directory per module
- remove filename restrictions (auto collect from source file)
