# Packages and the Go Tool

## Import Paths
1. the Go language specification doesn’t define the meaning of these strings or how to determine a package’s import path, but leaves these issues to the tools.

## The Package Declaration
1. Conventionally, the package name is the last segment of the import path
2. a package defining a command (an executable Go program) always has the name `main`, regardless of the package’s import path. This is a signal to `go build` that it must invoke the linker to make an executable file.
3. some files in the directory may have the suffix _test on their package name if the file name ends with _test.go. Such a directory may define two packages: the usual one, plus another one called an external test package. The _test suffix signals to go test that it must build both packages, and it indicates which files belong to each package. External test packages are used to avoid cycles in the import graph arising from dependencies of the test.
4. some tools for dependency management append version number suffixes to package import paths, such as `gopkg.in/yaml.v2`. The package name excludes the suffix, so in this case it would be just `yaml`.

## Import Declarations
1. Imported packages may be grouped by introducing blank lines; such groupings usually indi- cate different domains. The order is not significant, but by convention the lines of each group aresortedalphabetically.
2. If we need to import two packages whose names are the same into a third package, the import declaration must specify an alternative name for at least one of them to avoid a conflict. This is called a renaming import. The alternative name affects only the importing file.

## Blank Imports
1. On occasion we must import a package merely for the side effects of doing so: evaluation of the initializer expressions of its package-level variables and execution of its init functions. To suppress the ‘‘unused import’’ error we would otherwise encounter, we must use a renaming import in which the alternative name is _, the blank identifier. This is known as a blank import. 

## Packages and Naming
1. When creating a package, keep its name short, but not so short as to be cryptic. Be descriptive and unambiguous where possible. For example, don’t name a utility package util when a name such as imageutil or ioutil is specific yet still concise.
2. Package names usually take the singular form.

## The Go Tool
1. If you specify the -u flag, go get will ensure that all packages it visits, including dependencies, are updated to their latest version before being built and installed. Without that flag, packages that already exist locally will not be updated.
2. The `go install` command is very similar to `go build`, except that it saves the compiled code for each package and command instead of throwing it away. Compiled packages are saved beneath the `$GOPATH/pkg` directory corresponding to the `src` directory in which the source resides, and command executables are saved in the `$GOPATH/bin` directory. Thereafter,`go build` and `go install` do not run the compiler for those packages and commands if they have not changed, making subsequent builds much faster. For convenience, `go build -i` installs the packages that are dependencies of the build target.
3. Each declaration of an exported package member and the package declaration itself should be immediately preceded by a comment explaining its purpose and usage. Go doc comments are always complete sentences, and the first sentence is usually a summary that starts with the name being declared. Function parameters and other identifiers are men- tioned without quotation or markup.
4. A comment immediately preceding a package declaration is considered the doc comment for the package as a whole. There must be only one, though it may appear in any file. Longer package comments may warrant a file of their own; fmt’s is over 300 lines. This file is usually called `doc.go`.
5. The `go doc` tool prints the declaration and doc comment of the entity specified on the command line, which may be a package or a package method, or a method.
6. You can also run an instance of godoc in your workspace if you want to browse your own packages: `godoc -http :8000`
7. `go build` tool treats a package specially if its import path contains a path segment named `internal`. Such packages are called internal packages. An internal package may be imported only by another package that is inside the tree rooted at the parent of the internal directory.
8. An argument to go list may contain the ‘‘...’’ wildcard, which matches any substring of a package’s import path. `go list ...xml...` list all packages related to a particular topic, e.g. xml. The `-f` flag lets users customize the output format using the template language of package text/template.