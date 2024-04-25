# How paths for handler functions work in serveMux

There are two different types of paths that go serveMux allows, fixed and subtree paths for handler functions.
If a path ends in a trailing `/` or is just `/` Go treats it as a wildcard and matches everything after it e.g.
`/view/` will match `/view/items`, basically `/view/*`.

If the path in the handler does not end with a trailing `/` go will treat it as an absolute path and it will not act as a wildcard and therefore not match anything after the path stated, e.g. `/view`.