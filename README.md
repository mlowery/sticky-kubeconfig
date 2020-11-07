# sticky-kubeconfig

sticky-kubeconfig is a set of shell scripts that provides a unique kubeconfig per terminal tab or window. What you do to your kubeconfig in one terminal session does not affect other sessions (including setting the current context).

## Components

1. Shell script that makes a copy of the "golden" aka canonical kubeconfig and sets `KUBECONFIG` environment variable to that path. Many tools respect `KUBECONFIG` if set and will use its value unless overridden by the `--kubeconfig` flag.
2. (Optional) Shell script that can tell you when your session-unique kubeconfig has diverged from the golden kubeconfig (so that you can choose to preserve those changes in the golden kubeconfig). This script can be used to decorate prompts to signal a "dirty" kubeconfig. The script can also be run by a human to assist with discarding or preserving the session-unique kubeconfig.

## Benefits

* Prompt decoration in a tab (e.g. `PS1` for Bash) that shows current kubeconfig context and namespace is always accurate, even if you've switched contexts in another tab.
* Saves passing the context each time as a `kubectl` argument.
* Warns when golden kubeconfig (i.e. `~/.kube/config_golden`) no longer matches current tab's copy.
* Works with prompts like [Starship](https://starship.rs/) that [respect](https://starship.rs/config/#kubernetes) `KUBECONFIG`.

## Installing

1. Create your golden kubeconfig. Your golden kubeconfig will be the starting kubeconfig for each terminal session. This can be done like so:

    ```sh
    cat ~/.kube/config | grep -v \"^current-context: \" > ~/.kube/config_golden
    ```
2. Add the following shell snippet to your `~/.bashrc` or `~/.zshrc`:

    ```sh
    export _STICKY_KUBECONFIG_GOLDEN=~/.kube/config_golden

    function _sticky_kubeconfig_init {
        local d=/tmp/sticky_kubeconfig/${TERM_SESSION_ID//:/-}
        mkdir -p $d
        KUBECONFIG=$d/config
        if [[ ! -f $KUBECONFIG ]]; then
            cp $_STICKY_KUBECONFIG_GOLDEN $KUBECONFIG
        fi
        export KUBECONFIG
    }

    _sticky_kubeconfig_init
    ```
3. (Optional) Decorate your shell prompt using the `sk-dirty` shell script. Here's an example using [Starship](https://starship.rs/) that adds an icon to the prompt when the session-unique kubeconfig is dirty:

    ```sh
    [custom.sk-dirty]
    when = 'sk-dirty -q'
    command = "echo 💾"
    ```
4. (Optional) When you see the dirty icon in your prompt. You can optionally choose to set the session-unique kubeconfig as the new golden kubeconfig. To do so, run `sk-dirty` (with no arguments).

## Requirements

* Your terminal emulator must set environment variable `TERM_SESSION_ID`.

## Best Practices

* (Optional) Don't set a `current-context` in your golden kubeconfig. This will keep the prompt decoration hidden (depending on your prompt) until you've set a current context. This reduces "noise" in your prompt if `kubectl` isn't being used.

## Testing

Tested on macOS, ZSH, and iTerm2.
