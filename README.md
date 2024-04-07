# pretty 🪶

a simple pretty printer for arbitrary data structures in [Zig](https://ziglang.org/)⚡️

designed to inspect deeply-nested and recursive tree structures.

current version: [v0.9.4](https://github.com/timfayz/pretty/tags) (tested on `0.12.0-dev.3539+23f729aec`)

## demo

simplified version of [`src/demo.zig`](https://github.com/timfayz/pretty/blob/main/src/demo.zig):

```zig
// main.zig
const std = @import("std");
const pretty = @import("pretty.zig");
const alloc = std.heap.page_allocator;

pub fn main() !void {
    const array = [3]u8{ 4, 5, 6 };

    // method 1: print with default options
    try pretty.print(alloc, array, .{});

    // method 2: customize your print
    try pretty.print(alloc, array, .{
        .array_show_item_idx = false,
        .inline_mode = true,
    });

    // method 3: don't print, get a string!
    var out = try pretty.dump(alloc, array, .{});
    defer alloc.free(out);
    std.debug.print("{s}..\n", .{out[0..5]});
}
```

output:

```
$ zig run main.zig
[3]u8
  [0]: 4
  [1]: 5
  [2]: 6

[3]u8{ 4, 5, 6 }

[3]u8..
```

## installation

there are three ways to use pretty:

- `git clone https://github.com/timfayz/pretty`, `cd pretty` and `zig build run`, or
- [download latest version](https://github.com/timfayz/pretty/blob/main/src/pretty.zig) of `pretty.zig` and `@import` it directly inside your project, or
- use [zig build system](https://ziglang.org/learn/build-system/) and `build.zig.zon` (highly undocumented):

```bash
$ cd your_project
$ ls
src/
build.zig     # should be present (otherwise use `zig init` to generate it)
build.zig.zon # optional (otherwise will be generated by the next command)
$ zig fetch --save https://github.com/timfayz/pretty/archive/v0.9.4.tar.gz
$ cat build.zig.zon
..
  .pretty = .{
      .url = "https://github.com/timfayz/pretty/archive/v0.9.4.tar.gz",
      .hash = "..long_hash..",
  },
..
```

edit `build.zig`:

```zig
// find this:
const exe = b.addExecutable(.{
    .name = "your_app",
    .root_source_file = .{ .path = "src/main.zig" },
    .target = target,
    .optimize = optimize,
});

// add the following:
const pretty = b.dependency("pretty", .{ .target = target, .optimize = optimize });
exe.root_module.addImport("pretty", pretty.module("pretty"));
..
```

done, build or run your project:

```shell
$ zig build # or zig build run
```

## api

pretty offers four main functions:

1. `pretty.print` – print formatted string to stdout.
1. `pretty.printInline` – print formatted string with `.inline_mode` switched on.
1. `pretty.dump` – generate formatted string and return it as `[]u8` slice.
1. `pretty.dumpAsList` – generate formatted string and return it as `std.ArrayList` interface.

## options

pretty output can be customized with the following options (default values in the option code):

#### generic printing options

- activate single line printing mode:

```
inline_mode: bool = false
```

- limit the printing depth:

```
max_depth: u8 = 10
```

- specify depths to include or exclude from the output:

```
filter_depths: Filter(usize) = .{ .exclude = &.{} }
```

- indentation size for multi-line printing mode:

```
tab_size: u8 = 2
```

- add an extra empty line at the end of the output (to stack up multiple prints):

```
empty_line_at_end: bool = false
```

- indicate empty output with a message (otherwise leave as `""`):

```
indicate_empty_output: bool = true
```

- specify a custom format string (eg. `"pre{s}post"`) to surround the resulting output:

```
fmt: []const u8 = ""
```

#### generic type printing options

- display type tags (ie. `std.builtin.TypeId`, such as `.Union`, `.Int`):

```
show_type_tags: bool = false
```

- display type names:

```
show_type_names: bool = true
```

- limit the length of type names (0 does not limit):

```
type_name_max_len: usize = 60
```

- specify level of folding brackets in type names with `..` (0 does not fold):

```
type_name_fold_brackets: usize = 1
```

- do not fold brackets for function signatures:

```
type_name_fold_except_fn: bool = true
```

#### generic value printing options

- display values:

```
show_vals: bool = true
```

- display empty values:

```
show_empty_vals: bool = true
```

#### pointer printing options

- follow pointers instead of printing their address:

```
ptr_deref: bool = true
```

- reduce duplicating depths when dereferencing pointers:

```
ptr_skip_dup_unfold: bool = true
```

#### optional printing options

- reduce duplicating depths when unfolding optional types:

```
optional_skip_dup_unfold: bool = true
```

#### struct and unions printing options

- display struct fields:

```
struct_show_field_names: bool = true
```

- treat empty structs as having `(empty)` value:

```
struct_show_empty: bool = true
```

- inline primitive type values to save vertical space:

```
struct_inline_prim_types: bool = true
```

- limit the number of fields in the output (0 does not limit):

```
struct_max_len: usize = 15
```

- specify field names to include or exclude from the output:

```
filter_field_names: Filter([]const u8) = .{ .exclude = &.{} }
```

- specify field type tags to include or exclude from the output:

```
filter_field_type_tags: Filter(std.builtin.TypeId) = .{ .exclude = &.{} }
```

- specify field types to include or exclude from the output:

```
filter_field_types: Filter(type) = .{ .exclude = &.{} }
```

#### array and slice printing options

- limit the number of items in the output (0 does not limit):

```
array_max_len: usize = 20
```

- display item indices:

```
array_show_item_idx: bool = true
```

- inline primitive type values to save vertical space:

```
array_inline_prim_types: bool = true
```

- display primitive type names:

```
array_hide_prim_types: bool = true
```

#### primitive types options

- specify type tags to treat as primitives:

```
prim_type_tags: Filter(std.builtin.TypeId) = .{ .include = &.{
    .Int,
    .ComptimeInt,
    .Float,
    .ComptimeFloat,
    .Void,
    .Bool,
} }
```

- specify concrete types to treat as primitives:

```
prim_types: Filter(type) = .{ .include = &.{} }
```

#### string printing options

- limit the length of strings (0 does not limit):

```
str_max_len: usize = 80
```

- treat `[]u8` as `"string"`.

```
slice_u8_is_str: bool = true
```

- treat `[:0]u8` as `"string"`.

```
slice_u8z_is_str: bool = true
```

- treat `[n]u8` as `"string"`.

```
array_u8_is_str: bool = false
```

- treat `[n:0]u8` as `"string"`.

```
array_u8z_is_str: bool = true
```

## examples

derived from [`src/tests.zig`](https://github.com/timfayz/pretty/blob/main/src/tests.zig):

```zig
const value: struct {
    field1: bool = true,
    field2: u8 = 42,
    field3: f32 = 1.1,
} = .{};
```

```zig
try pretty.printInline(alloc, value, .{});
```

```
file.main__struct_1472{ .field1: bool = true, .field2: u8 = 42, .field3: f32 = 1.10000002e+00 }
```

```zig
try pretty.print(alloc, value, .{});
```

```
file.main__struct_1652
  .field1: bool => true
  .field2: u8 => 42
  .field3: f32 => 1.10000002e+00
```

use `filter_*` options to "query" or cut off unnecessary output:

```zig
try pretty.print(alloc, value, .{
    .filter_depths = .{ .exclude = &.{ 0, 2 } },
});
```

```
.field1: bool
.field2: u8
.field3: f32
```

```zig
try pretty.print(alloc, value, .{
      .filter_depths = .{ .include = &.{2} },
});
```

```
true
42
1.10000002e+00
```

```zig
try pretty.print(alloc, value, .{
    .filter_depths = .{ .exclude = &.{0} },
    .filter_field_names = .{ .include = &.{"field2"} },
});
```

```
.field2: u8 => 42
```

## contributing ❤️

please feel free to:

- open an issue and describe the change you'd like to see.
- fork the repository and submit your pull request.

## license

all codebase are belong to MIT.
