---
title: How to use
menu_order: 3
---

# How to use

Opening either `*.fs`, `*.fsi` or `*.fsx` files should trigger syntax highlighting and other depending runtime files as well.

### Commands

Refer to [LanguageClient-neovim](https://github.com/autozimu/LanguageClient-neovim) for features provided via Language Server Protocol.

To be added as requested for F#-specific features.

#### `:FSharpLoadWorkspaceAuto`
  - Searches a workspace (`sln` or `fsproj`) and then load it.
  - Equivalent to `FSharp.workspaceMode = sln` in Ionide-VSCode.
  - Automatically called when you open F# files. Can be disabled in settings.
  - The deep level of directory hierarchy to search can also be configured in settings.

#### `:FSharpParseProject <files>+`
  - Loads specified projects (`sln` or `fsproj`).

#### `:FSharpReloadWorkspace`
  - Reloads all the projects currently loaded.
  - Automatically called when you save `.fsproj` files. Can be disabled in settings.

#### `:FSharpUpdateServerConfig`
  - Updates FSAC configuration.
  - See [FsAutoComplete Settings](#fsautocomplete-settings) for details.

#### `:FSharpUpdateFSAC`
  - Downloads the latest build of FsAutoComplete to be used with Ionide-vim.

### Working with F# Interactive

Ionide-vim has an integration with F# Interactive.

FSI is displayed using the builtin `:terminal` feature introduced in Vim 8 / Neovim and can be used like in VSCode.

#### `:FsiShow`
  - Shows a F# Interactive window.

#### `:FsiEval <expr>`
  - Evaluates given expression in FSI.

#### `:FsiEvalBuffer`
  - Sends the content of current file to FSI.

#### `:FsiReset`
  - Resets the current FSI session.

#### `Alt-Enter`
  - When in normal mode, sends the current line to FSI.
  - When in visual mode, sends the selection to FSI.
  - Sending code to FSI opens FSI window but the cursor does not focus to it. Unlike Neovim, Vim doesn't support asynchronous buffer updating so you have to input something (e.g. moving cursor) to see the result. You can change this behavior in settings.

#### `Alt-@`
  - Toggles FSI window. FSI windows shown in different tabpages share the same FSI session.
  - When opened, the cursor automatically focuses to the FSI window (unlike in `Alt-Enter` by default).

You can customize the location of FSI, key mappings, etc. See [the documentation below](#f-interactive-settings).

### Settings

Refer to [LanguageClient-neovim's recommended settings](https://github.com/autozimu/LanguageClient-neovim/wiki/Recommended-Settings#recommended-settings)
for features provided via Language Server Protocol.

To be added as requested for F#-specific features.

#### FsAutoComplete Settings

* Ionide-vim uses `snake_case` for the setting names.
  - For FSAC settings only, `CamelCase` can also be used (as it gets serialized to a F# record).
  - If both `snake_case` and `CamelCase` are specified, the `snake_case` one will be preferred.
* You can change the values at runtime and then notify the changes to FSAC by `:FSharpUpdateServerConfig`.
* Some of the settings may not work in Ionide-vim as it is lacking the corresponding feature of Ionide-VSCode.
* If not specified, the recommended default values described on the FSAC's documentation will be used.
  - If you are using a JSON configuration file though `g:LanguageClient_settingsPath`, the recommended default values will override the settings loaded from it.
  - You can disable this by `let g:fsharp#use_recommended_server_config = 0`.

See [the documentation of FSAC](https://github.com/fsharp/FsAutoComplete#settings)
for the complete list of available settings. Frequently used ones are:

##### Enable/disable automatic calling of `:FSharpLoadWorkspaceAuto` on opening F# files (default: enabled)

~~~.vim
let g:fsharp#automatic_workspace_init = 1 " 0 to disable.
~~~

##### Set the deep level of directory hierarchy when searching for sln/fsprojs (default: `2`)

~~~.vim
let g:fsharp#workspace_mode_peek_deep_level = 2
~~~

##### Ignore specific directories when loading a workspace (default: empty)

~~~.vim
let g:fsharp#exclude_project_directories = ['paket-files']
~~~

##### Enable/disable linter and unused opens/declarations analyzer (default: all enabled)

You may want to bind `LanguageClient#textDocument_codeAction()` to some shortcut key. Refer to their docs.

~~~.vim
" 0 to disable.
let g:fsharp#linter = 1
let g:fsharp#unused_opens_analyzer = 1
let g:fsharp#unused_declarations_analyzer = 1
~~~

#### Editor Settings

##### Enable/disable automatic calling of `:FSharpReloadWorkspace` on saving `fsproj` (default: enabled)

~~~.vim
let g:fsharp#automatic_reload_workspace = 1 " 0 to disable.
~~~

##### Show type signature at cursor position (default: enabled)

~~~.vim
let g:fsharp#show_signature_on_cursor_move = 1 " 0 to disable.
~~~

#### F# Interactive Settings

##### Change the F# Interactive command to be used within Ionide-vim (default: `dotnet fsi`)

If you want to use a .NET Framework FSI instead of .NET Core one, set `g:fsharp#use_sdk_scripts` to `0`.
See: https://github.com/fsharp/FsAutoComplete/pull/466#issue-324869672

~~~.vim
let g:fsharp#fsi_command = "fsharpi"
let g:fsharp#use_sdk_scripts = 0 " for net462 FSI
~~~

##### Set additional runtime arguments passed to FSI (default: `[]` (empty))

Sets additional arguments of the FSI instance Ionide-vim spawns and changes the behavior of FSAC accordingly when editing fsx files.

~~~.vim
let g:fsharp#fsi_extra_parameters = ['--langversion:preview']
~~~

##### Customize how FSI window is opened (default: `botright 10new`)

It must create a new empty window and then focus to it.

See [`:help opening-window`](http://vimdoc.sourceforge.net/htmldoc/windows.html#opening-window) for details.

~~~.vim
let g:fsharp#fsi_window_command = "botright vnew"
~~~

##### Set if sending line/selection to FSI shoule make the cursor focus to FSI window (default: disabled)

If you are using Vim, you might want to enable this to see the result without inputting something.

~~~.vim
let g:fsharp#fsi_focus_on_send = 1 " 0 to not to focus.
~~~

##### Change the key mappings (default: `vscode`)

* `vscode`:     Default. Same as in Ionide-VSCode (`Alt-Enter` to send, `Alt-@` to toggle terminal).
  - `<M-CR>` in Neovim / `<ESC><CR>` in Vim: Sends line/selection to FSI.
  - `<M-@>`  in Neovim / `<ESC>@`    in Vim: Toggles FSI window.
* `vim-fsharp`: Same as in [fsharp/vim-fsharp](https://github.com/fsharp/vim-fsharp#fsharp-interactive). Note that `<leader>` is mapped to backslash by default. See [`:help mapleader`](http://vimdoc.sourceforge.net/htmldoc/map.html#mapleader).
  - `<leader>i` : Sends line/selecion to FSI.
  - `<leader>e` : Toggles FSI window.
* `custom`:     You must set both `g:fsharp#fsi_keymap_send` and `g:fsharp#fsi_keymap_toggle` by yourself.
  - `g:fsharp#fsi_keymap_send`   : Sends line/selection to FSI.
  - `g:fsharp#fsi_keymap_toggle` : Toggles FSI window.
* `none`:       Disables mapping.

~~~.vim
" custom mapping example
let g:fsharp#fsi_keymap = "custom"
let g:fsharp#fsi_keymap_send   = "<C-e>"
let g:fsharp#fsi_keymap_toggle = "<C-@>"
~~~

### Advanced Tips

#### Show tooltips on CursorHold

If you are using neovim 0.4.0 or later, floating windows will be used for tooltips and you might find it convenient to make them appear if the cursor does not move for several seconds.

~~~.vim
if has('nvim') && exists('*nvim_open_win')
  augroup FSharpShowTooltip
    autocmd!
    autocmd CursorHold *.fs,*.fsi,*.fsx call fsharp#showTooltip()
  augroup END
endif
~~~

Note that you can set the delay time to show the tooltip by [`set updatetime=<ms>`](http://vimdoc.sourceforge.net/htmldoc/options.html#'updatetime'). The default delay is 4 seconds, which you may find too slow.