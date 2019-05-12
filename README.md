<p align="center">
  <img width="443" alt="logo" src="https://user-images.githubusercontent.com/633843/57529538-84a22780-7378-11e9-9235-312633dc125e.png"><br>
  <b>A low friction command-line interface library for Go (golang)</b><br><br>
  <a href="https://godoc.org/github.com/jpillora/opts" rel="nofollow"><img src="https://camo.githubusercontent.com/42566bdba17f1a0c86c1a1de859d6ab70bde1457/68747470733a2f2f676f646f632e6f72672f6769746875622e636f6d2f6a70696c6c6f72612f6f7074733f7374617475732e737667" alt="GoDoc" data-canonical-src="https://godoc.org/github.com/jpillora/opts?status.svg" style="max-width:100%;"></a> <a href="https://circleci.com/gh/jpillora/opts" rel="nofollow"><img src="https://camo.githubusercontent.com/34202387888c6b05f640653a29bb1e204f5a9e19/68747470733a2f2f636972636c6563692e636f6d2f67682f6a70696c6c6f72612f6f7074732e7376673f7374796c653d736869656c6426636972636c652d746f6b656e3d36396566396336616330643863656263623335346262383563333737656365666637376266623162" alt="CircleCI" data-canonical-src="https://circleci.com/gh/jpillora/opts.svg?style=shield&amp;circle-token=69ef9c6ac0d8cebcb354bb85c377eceff77bfb1b" style="max-width:100%;"></a>
</p>

---

Creating command-line interfaces should be simple:

```go
package main

import (
	"log"

	"github.com/jpillora/opts"
)

func main() {
	config := struct {
		File  string `opts:"help=file to load"`
		Lines int    `opts:"help=number of lines to show"`
	}{}
	opts.Parse(&config)
	log.Printf("%+v", config)
}
```

```sh
$ go build -o my-prog
$ ./my-prog --help

  Usage: my-prog [options]

  Options:
  --file, -f   file to load
  --lines, -l  number of lines to show
  --help, -h

```

```sh
$ ./my-prog -f foo -l 12
{File:foo Lines:42}
```

### Features (with examples)

- Easy to use ([eg-helloworld](example/eg-helloworld/))
- Promotes separation of CLI code and library code ([eg-app](example/eg-app/))
- Automatically generated `--help` text via struct tags ([eg-help](example/eg-help/))
- Default values by modifying the struct prior to `Parse()` ([eg-defaults](example/eg-defaults/))
- Default values from a JSON config file, unmarshalled via your config struct ([eg-config](example/eg-config/))
- Default values from environment, defined by your field names ([eg-env](example/eg-env/))
- Group your flags in the help output ([eg-groups](example/eg-groups/))
- Sub-commands by nesting structs
- Sub-commands by providing child `Opts`
- Infers program name from executable name
- Infers command names from struct or package name
- Define custom flags types via `flag.Value` ([eg-customtypes](example/eg-customtypes/))
- Customizable help text by modifying the default templates ([eg-customhelp](example/eg-customhelp/))
- Built-in shell auto-completion ([eg-complete](example/eg-complete))

Find these examples and more in the [`example/`](./example) directory.

### Package API

See https://godoc.org/github.com/jpillora/opts

[![GoDoc](https://godoc.org/github.com/jpillora/opts?status.svg)](https://godoc.org/github.com/jpillora/opts)

### Struct Tag API

**opts** tries to set sane defaults so, for the most part, you'll get the desired behaviour by simply providing a configuration struct.

However, you can customise this behaviour by providing the `opts` struct
tag with a series of settings in the form of **`key=value`**:

```
`opts:"key=value,key=value,..."`
```

Where **`key`** must be one of:

- `-` (dash) - Like `json:"-"`, the dash character will cause opts to ignore the struct field. Unexported fields are always ignored.

- `name` - Name is used to display the field in the help text. By default, the flag name is infered by converting the struct field name to lowercase and adding dashes between words.

- `short` - One or two letters to be used a flag's "short" name. By default, the first letter of `name` will be used. It will remain unset if there is a duplicate short name. Only valid when `mode` is `flag`.

- `help` - The help text used to describe the field. It will be displayed next to the flag name in the help output.

	**Note:** `help` is the only setting that can also be set as a stand-alone struct tag (i.e. `help:"my text goes here"`). You must use the stand-alone tag if you wish to use a comma `,` in your help string.

- `group` - The name of the flag group to store the field. Defining this field will create a new group of flags in the help text (will appear as "`<group>` options"). The default flag group is the empty string (which will appear as "Options").

	**Note:** `embedded` `struct`s implicitly set the group name to be the name of the struct field. You can explicitly unset the group name with `group=`.

- `env` - An environent variable to use as the field's **default** value. It can always be overridden by providing the appropriate flag.

	For example, `opts:"env=FOO"`. It can also be infered using the field name with simply `opts:"env"`. You can enable inference on all flags with the `opts.Opts` method `UseEnv()`.

- `mode` - The **opts** mode assigned to the field. All fields will be given a `mode`. Where the `mode` **`value`** must be one of:

	* `flag` - The field will be treated as a flag: an optional, named, configurable field. Set using `./program --<flag-name> <flag-value>`. The struct field must be a [*flag-value*](#flag-values) type. `flag` is the default `mode` for any [*flag-value*](#flag-values).

	* `arg` - The field will be treated as an argument: a required, positional, unamed, configurable field. Set using `./program <argument-value>`. The struct field must be a [*flag-value*](#flag-values) type.

	* `embedded` - A special mode which causes the fields of struct to be used in the current struct. Useful if you want to split your command-line options across multiple files (default for `struct` fields). The struct field must be a `struct`. `embedded` is the default `mode` for a `struct`.

	* `cmd` - A inline command, shorthand for `.AddCommmand(opts.New(&field))`, which also implies the struct field must be a `struct`.

	* `cmdname` - A special mode which will assume the name of the selected command. The struct field must be a `string`.

- `min` - A minimum length of a slice. Only valid when `mode` is `flag` or `arg`, and when the field type is a slice.

#### flag-values:

In general an opts _flag-value_ type aims to be any type that can be get and set using a `string`. Currently, `opts` supports the following types:

- `string`
- `bool`
- `int{8-64}`
- `uint{8-64}`
- `float{32-64}`
- `time.Duration`
- [`flag.Value`](https://golang.org/pkg/flag/#Value)
- `encoding.TextMarshaler` and `encoding.TextUnmarshaler`
	- `time.Time`
- `encoding.BinaryMarshaler` and `encoding.BinaryUnmarshaler`
	- `url.URL`

In addition, `flag`s and `arg`s can also be a slice of any _flag-value_ type. Slices allow multiple flags/args. For example, a struct field `Foo []int` could be set with `--foo 1 --foo 2` and would result in `[]int{1,2}`.

### Help text

By default, **opts** attempts to output well-formatted help text when the user provides the `--help` (`-h`) flag. The [examples/](./example) directory shows various combinations of this default help text, resulting from using various features above.

Simple additions can be made by using the following methods:

* `Summary(text)` - Adds a summary line below the usage text
* `Version(text)` - Adds a version line below all options
* `Author(text)` - Adds an author line below all options

Simple formatting modifications can be made by using:

* `PadWidth(int)`  TODO

Advanced modifications be made customising the underlying [Go templates](https://golang.org/pkg/text/template/) found here [node_help.go#DefaultTemplates](https://github.com/jpillora/opts/blob/master/node_help.go#L55). You can edit each template by ID (map key) using the following methods:

* `DocSet(id, template)` TODO

### Other projects

Other related projects which infer flags from struct tags:

- https://github.com/alexflint/go-arg
- https://github.com/jessevdk/go-flags

#### MIT License

Copyright © 2019 &lt;dev@jpillora.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
