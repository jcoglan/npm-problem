# Node plugin problem

Let's say you run a project which lets part of its functionality be pluggable.
For example, you have an ORM and people can add new database adapters. You want
to provide an API like this:

    Engine.register('plugin', MyPlugin)

The idea is that then, when the user specifies `plugin` in some config somewhere,
`MyPlugin` is selected as the adapter to use.

Now, it seems reasonable for the `plugin` project to specify a dependency on the
`engine` project, so it can say which range of versions of `engine` it works
with. Say I have a project that wants to use `engine` and `plugin`, and so I use
`npm` to install them. The directory structure then looks like what you will see
in this repo:

    node_modules/
        engine/
            index.js
        plugin/
            node_modules/
                engine/
                    index.js
            index.js

In *my* project, I do this:

    var engine = require('engine'),
        plugin = require('plugin')

Inside `plugin/index.js`, this happens:

    var engine = require('engine')
    
    engine.register('testPlugin', function() {
      return 'Hello, World'
    })

The problem is, `plugin` loads the copy of `engine` from *its* `node_modules`
whereas my project's files load the one from the outermost `node_modules`. The
`engine` I'm using is not aware of the presence of `plugin` and so cannot use it.

Removing `engine` from `node_modules` inside `plugin` fixes this problem, and
allows `plugin` to see the same copy of `engine` my app is using.

