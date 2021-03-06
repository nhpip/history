# history 

Saves shell history and optionally variable bindings between shell sessions.

Allows the user to display history, and re-issue historic commands, made much
easier since the variable bindings are saved.

Some screen shots can be found here: [Images](https://github.com/nhpip/history/wiki)

For ease History can be enabled in `~/.iex.exs` for example:

    Code.append_path("~/github/history/_build/dev/lib/iex_history/ebin")
    History.initialize(history_limit: 200, scope: :local, show_date: true, colors: [index: :red])

Of course `Code.append_path` may not be required depending on how the project is imported.

The following options can be set:

    [
      scope: :local,
      history_limit: :infinity,
      hide_history_commands: true,
      prepend_identifiers: true,
      command_display_width: int,
      save_invalid_results: false,
      key_buffer_history: true,
      show_date: true,
      save_bindings: true,
      colors: [
        index: :red,
        date: :green,
        command: :yellow,
        label: :red,
        variable: :green
      ]
    ]
    
`:hide_history_commands` This will prevent all calls to History.* from been saved.

NOTE: `History.x/1` is always hidden. Scope of `:global` will only hide them from output, otherwise they will not be saved.

`:save_invalid_results` If set to false, the default, commands that were evaluated incorrectly will not be saved.

`:key_buffer_history` If set to true will allow the user to scroll up `(ctrl+u)` or down `(ctrl+k)` through history. Unlike the standard up/down arrow history this is command based not line based. So pasting of a large structure will only require 1 up or down. This mechanism also saves commands that were not properly evaluated; however there is a buffer limit of 75 lines, although this can be changed by updating `@history_buffer_size` in `events_server.ex`. This will also not duplicate back to back identical commands.

`:prepend_identifiers`  If this is enabled it will prepend identifiers when a call to `x = History(val)` is issued.

For example:
```
    enabled:
        iex> time = Time.utc_now().second
        14
        iex> new_time = History.x(1)
        22

        iex> new_time
        22                  # New time is assigned to variable time
        iex> time
        13                  # However, the original date variable is unchanged

        iex> History.h()
        1: 2021-09-01 17:13:13: time = Time.utc_now().second
        2: 2021-09-01 17:13:22: new_time =  time = Time.utc_now().second    # We see the binding to new_time

      disabled:
        iex> time = Time.utc_now().second
        43
        iex> new_time = History.x(1)
        50

        iex> new_time       # New time is assigned to variable time
        50
        iex> time
        50                  # However, this time the original time variable has also changed

        iex> History.h
        1: 2021-09-01 17:17:43: time = Time.utc_now().second
        2: 2021-09-01 17:17:50: time = Time.utc_now().second      # We do not see the binding to new_time
```

`:scope` can be one of `:local, :global` or a `node()` name

If `scope` is `:local` (the default) history will be active on all shells, even those that are remotely connected, but the history for each shell will be unique

If `scope` is `node()` (e.g. `:mgr@localhost`) history will only be active on that shell

If `scope` is `:global` history will be shared between all shells. However the saving of variable bindings will be disabled along with the date/time in history

Furthermore, if a `scope` of `:global` is selected following kernel option must be set, either directly as VM options or via an environment variable:
```
    export ERL_AFLAGS="-kernel shell_history enabled"

    --erl "-kernel shell_history enabled"
```
A word about aliases. Rather than using something like `alias History, as: H`, please use `History.alias(H)` instead.

## Functions

### History.h()
Displays the entire history.
```
    iex> History.h()
    1: 2021-09-01 17:29:27: time = Time.utc_now().second
    2: 2021-09-01 17:29:31: time = Time.utc_now().second
    3: 2021-09-01 17:29:36: time
    4: 2021-09-01 17:29:41: new_time
    5: 2021-09-01 17:50:10: Process.info self
    6: 2021-09-01 17:50:33: r = o
    7: 2021-09-01 17:52:36: Process.get(:iex_history)
```

### History.h(val)
If the argument is a string it displays the history that contain or match entirely the passed argument. 
If the argument is a positive integer it displays the command at that index.
If the argument is a negative number it displays the history that many items from the end.
```
    iex> History.h(2)
    2: 2021-09-01 17:29:31: time = Time.utc_now().second
    
    iex> History.h("Applic")
    34: 2021-09-01 18:10:39: Application.put_env(:kernel, :shell_history, :disabled)
    41: 2021-09-01 18:11:30: Application.get_env(:kernel, :shell_history)
    48: 2021-09-01 18:14:02: Application.put_env(:kernel, :shell_history, 0)
    101: 2021-09-01 19:01:15: :rpc.call(:erlang.node(Process.group_leader()), Application, :put_env, [:kernel, :shell_history, :disabled])
    103: 2021-09-01 19:01:30: :rpc.call(:erlang.node(Process.group_leader()), Application, :put_env, [:kernel, :shell_history, :enabled])
    
    iex> History.h(-3)
    5: 2021-09-01 17:50:10: Process.info self
    6: 2021-09-01 17:50:33: r = o
    7: 2021-09-01 17:52:36: Process.get(:iex_history)
```

### History.h(start, stop)
Specify a range, the atoms :start and :stop can also be used.

### History.x(idx)
Invokes the command at index 'i'.
```
    iex> History.h(114)
    114: 2021-09-01 19:30:14: Enum.count([1, 2, 3])
    
    iex> History.x(114)
    3
```

### History.c(idx)
Copies the command at index 'i' and pastes it to the shell.
```
    iex> History.h(114)
    114: 2021-09-01 19:30:14: Enum.count([1, 2, 3])
    
    iex> History.c(114)
    :ok
    iex> Enum.count([1, 2, 3])
``` 

### History.initialize(opts)
Initializes the History app. Takes the following parameters:
```
      [
        scope: :local,
        history_limit: :infinity,
        prepend_identifiers: true,
        show_date: true,
        save_invalid_results: false,
        key_buffer_history: true,
        save_bindings: true,
        colors: [
          index: :red,
          date: :green,
          command: :yellow,
          label: :red,
          variable: :green
        ]
      ]
```

### History.state()
Displays the current state:
```
    History version 4.0 is 
    enabled:
      Current history is 199 commands in size.
      Current bindings are 153 variables in size.
```

### History.clear()
Clears the history and bindings. If scope is  :global the IEx session needs restarting for the changes to take effect.

### History.clear_history(range)
Clears the history only, if no argument all history is cleared, else history from 1 to value is cleared

### History.clear_bindings()
Clears bindings only

### History.unbind(vars)
Unbinds a variable or list of variables, varibales should be expressed as atoms

### History.stop_clear()
Clears the history and bindings then stops the service. If scope is :global the IEx session needs restarting for the changes to take effect.

### History.configuration()
Displays the current conifuration

### History.save_config(filename)
Saves the configuration to filename

### History.load_config(filename)
Loads the configuration from filename. 
NOTE: All changes may not be applied, to do this specify the filename in `History.initialize/1` instead of a config keyword list

### History.configure/2
Allows the following options to be changed, but not saved:
```
    :show_date
    :history_limit
    :hide_history_commands,
    :prepend_identifiers,
    :command_display_width,
    :save_invalid_results,
    :key_buffer_history,
    :save_bindings,
    :colors
 ```   
Examples:
```
    History.configure(:colors, [index: :blue])
    History.configure(:prepend_identifiers, true)
```

### History.get_bindings()
Displays the current shell bindings.

### History.is_enabled?()
Returns true or false is History is enabled

### History.alias/1
If you want to setup an alias like `alias History, as: H`, rather than using `alias/2` from the shell, please use this function instead. So to create an alias of `H` use `History.alias(H)`. This allows aliased functions to be handled correctly.
