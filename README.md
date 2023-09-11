# diffheap

This little utility runs a go build with gcflags set to `-m=2` and processes
the output to make it less dependent on file structure: row/cols are removed,
file names are replaced with `<pkg>.<func>` or `<pkg>.<receiver>.<func>`.

## Example Call

```interactive
open-policy-agent/opa % go build -gcflags="-m=2" ./ast 2>&1 | grep capabilities.go
ast/capabilities.go:56:6: cannot inline CapabilitiesForThisVersion: function too complex: cost 203 exceeds budget 80
ast/capabilities.go:65:25: can inline CapabilitiesForThisVersion.func1 with cost 12 as: func(int, int) bool { return f.Builtins[i].Name < f.Builtins[j].Name }
ast/capabilities.go:59:47: inlining call to capabilities.ABIVersions
ast/capabilities.go:72:14: inlining call to sort.Strings
ast/capabilities.go:63:19: make([]*Builtin, len(Builtins)) escapes to heap:
ast/capabilities.go:63:19:   flow: {heap} = &{storage for make([]*Builtin, len(Builtins))}:
ast/capabilities.go:63:19:     from make([]*Builtin, len(Builtins)) (spill) at ast/capabilities.go:63:19
ast/capabilities.go:63:19:     from f.Builtins = make([]*Builtin, len(Builtins)) (assign) at ast/capabilities.go:63:13
ast/capabilities.go:65:14: f.Builtins escapes to heap:
ast/capabilities.go:65:14:   flow: {heap} = &{storage for f.Builtins}:
ast/capabilities.go:65:14:     from f.Builtins (spill) at ast/capabilities.go:65:14
ast/capabilities.go:65:14:     from sort.Slice(f.Builtins, func literal) (call parameter) at ast/capabilities.go:65:12
ast/capabilities.go:72:14: sort.StringSlice(sort.x) escapes to heap:
ast/capabilities.go:72:14:   flow: {heap} = &{storage for sort.StringSlice(sort.x)}:
ast/capabilities.go:72:14:     from sort.StringSlice(sort.x) (spill) at ast/capabilities.go:72:14
ast/capabilities.go:72:14:     from sort.Sort(sort.StringSlice(sort.x)) (call parameter) at ast/capabilities.go:72:14
ast/capabilities.go:74:23: []string{...} escapes to heap:
ast/capabilities.go:74:23:   flow: {heap} = &{storage for []string{...}}:
ast/capabilities.go:74:23:     from []string{...} (spill) at ast/capabilities.go:74:23
ast/capabilities.go:74:23:     from f.Features = []string{...} (assign) at ast/capabilities.go:74:13
ast/capabilities.go:57:2: CapabilitiesForThisVersion capturing by value: f (addr=false assign=false width=8)
ast/capabilities.go:63:19: make([]*Builtin, len(Builtins)) escapes to heap:
ast/capabilities.go:63:19:   flow: {heap} = &{storage for make([]*Builtin, len(Builtins))}:
ast/capabilities.go:63:19:     from make([]*Builtin, len(Builtins)) (non-constant size) at ast/capabilities.go:63:19
ast/capabilities.go:57:7: &Capabilities{} escapes to heap:
ast/capabilities.go:57:7:   flow: f = &{storage for &Capabilities{}}:
ast/capabilities.go:57:7:     from &Capabilities{} (spill) at ast/capabilities.go:57:7
ast/capabilities.go:57:7:     from f := &Capabilities{} (assign) at ast/capabilities.go:57:4
ast/capabilities.go:57:7:   flow: ~r0 = f:
ast/capabilities.go:57:7:     from return f (return) at ast/capabilities.go:78:2
ast/capabilities.go:57:7: &Capabilities{} escapes to heap
ast/capabilities.go:63:19: make([]*Builtin, len(Builtins)) escapes to heap
ast/capabilities.go:65:14: f.Builtins escapes to heap
ast/capabilities.go:65:25: func literal does not escape
ast/capabilities.go:72:14: sort.StringSlice(sort.x) escapes to heap
ast/capabilities.go:74:23: []string{...} escapes to heap
```

This is the "canonicalized" version, `diffheap`'s output:

```interactive
open-policy-agent/opa % diffheap ./ast | grep CapabilitiesForThisVersion        
ast:CapabilitiesForThisVersion: cannot inline CapabilitiesForThisVersion: function too complex: cost 203 exceeds budget 80
ast:CapabilitiesForThisVersion: can inline CapabilitiesForThisVersion.func1 with cost 12 as: func(int, int) bool { return f.Builtins[i].Name < f.Builtins[j].Name }
ast:CapabilitiesForThisVersion: inlining call to capabilities.ABIVersions
ast:CapabilitiesForThisVersion: inlining call to sort.Strings
ast:CapabilitiesForThisVersion: make([]*Builtin, len(Builtins)) escapes to heap:
ast:CapabilitiesForThisVersion:   flow: {heap} = &{storage for make([]*Builtin, len(Builtins))}:
ast:CapabilitiesForThisVersion:     from make([]*Builtin, len(Builtins)) (spill) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion:     from f.Builtins = make([]*Builtin, len(Builtins)) (assign) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion: f.Builtins escapes to heap:
ast:CapabilitiesForThisVersion:   flow: {heap} = &{storage for f.Builtins}:
ast:CapabilitiesForThisVersion:     from f.Builtins (spill) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion:     from sort.Slice(f.Builtins, func literal) (call parameter) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion: sort.StringSlice(sort.x) escapes to heap:
ast:CapabilitiesForThisVersion:   flow: {heap} = &{storage for sort.StringSlice(sort.x)}:
ast:CapabilitiesForThisVersion:     from sort.StringSlice(sort.x) (spill) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion:     from sort.Sort(sort.StringSlice(sort.x)) (call parameter) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion: []string{...} escapes to heap:
ast:CapabilitiesForThisVersion:   flow: {heap} = &{storage for []string{...}}:
ast:CapabilitiesForThisVersion:     from []string{...} (spill) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion:     from f.Features = []string{...} (assign) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion: CapabilitiesForThisVersion capturing by value: f (addr=false assign=false width=8)
ast:CapabilitiesForThisVersion: make([]*Builtin, len(Builtins)) escapes to heap:
ast:CapabilitiesForThisVersion:   flow: {heap} = &{storage for make([]*Builtin, len(Builtins))}:
ast:CapabilitiesForThisVersion:     from make([]*Builtin, len(Builtins)) (non-constant size) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion: &Capabilities{} escapes to heap:
ast:CapabilitiesForThisVersion:   flow: f = &{storage for &Capabilities{}}:
ast:CapabilitiesForThisVersion:     from &Capabilities{} (spill) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion:     from f := &Capabilities{} (assign) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion:   flow: ~r0 = f:
ast:CapabilitiesForThisVersion:     from return f (return) at ast:CapabilitiesForThisVersion
ast:CapabilitiesForThisVersion: &Capabilities{} escapes to heap
ast:CapabilitiesForThisVersion: make([]*Builtin, len(Builtins)) escapes to heap
ast:CapabilitiesForThisVersion: f.Builtins escapes to heap
ast:CapabilitiesForThisVersion: func literal does not escape
ast:CapabilitiesForThisVersion: sort.StringSlice(sort.x) escapes to heap
ast:CapabilitiesForThisVersion: []string{...} escapes to heap
```

When run on two different commits, the output of `diffheap` should allow for easier
manual inspection via `diff`.


### Example Use in GitHub Actions

To get automatic comments with diffs of the heap escape analysis, this can be paired
with [`int128/diff-action`](https://github.com/int128/diff-action), for example.

```yaml
  heapdiff:
    name: diff heap output
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
        with:
          path: branch
      - uses: actions/checkout@v4
        with:
          path: main
          ref: ${{ github.event.pull_request.base.ref }}
      - uses: actions/setup-go@v4
      - name: Set up diffheap util
        run: go install github.com/srenatus/diffheap@latest
      - run: diffheap ./pkg/vm > ../branch.out
        working-directory: branch
      - run: diffheap ./pkg/vm > ../main.out
        working-directory: main
      - uses: int128/diff-action@v1
        with:
          base: main.out
          head: branch.out
          comment-header: "escape analysis differences"
          comment-footer: "This is informational. I does not indicate a problem with the code change."
```
