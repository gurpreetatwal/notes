# How to configure Tern

The ternjs docs don't do the best job of telling a user what things go in the `.tern-project` file. Here are some resources that do:

- [atom-ternjs](https://atom.io/packages/atom-ternjs)
- [StackOverflow](https://stackoverflow.com/a/41377689)

## Summary

The two main configuration stanzas are the `"libs" `and `"plugins"` arrays.

### Default Config
The default configuration is available here as of today: https://github.com/ternjs/tern/blob/master/bin/tern#L48

### Libs
`"libs"` provide [JSON type definitions](http://ternjs.net/doc/manual.html#typedef) for common libraries

The most up to date list of "libs" is available here: https://github.com/ternjs/tern/tree/master/defs and includes
- browser
- chai
- ecmascript
- jquery
- react
- underscore


### Plugins
`"plugins"` provide extra functionality to tern

The most up to date list of "plugins" is available here: https://github.com/ternjs/tern/tree/master/plugin and includes
- angular
- commonjs
- complete_strings
- doc_comment
- es_modules
- modules
- node
- node_resolve
- requirejs
- webpack

The best place to get a description of what each plugin does is actually the tern website: http://ternjs.net/doc/manual.html#plugins

Some plugins also take in configuration options. The tern website explains most of the options, but it's not always up to date, the best thing to do is to open the plugin's source file and search for `registerPlugin`. The second argument to the function are the options used from `.tern-project`.
