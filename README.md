# TMUX Setup

To get the right setup for TMUX, you first need to run the following commands:

```
brew install tmux
brew install reattach-to-user-namespace

```

Your `~/.tmux.conf` should look like this:

```
unbind C-b
set -g prefix C-s
bind-key -r C-s send-prefix

bind-key r source-file ~/.tmux.conf \; display-message "~/.tmux.conf reloaded"

bind-key -n C-h select-pane -L
bind-key -n C-j select-pane -D
bind-key -n C-k select-pane -U
bind-key -n C-l select-pane -R

set-option -g default-terminal "screen-256color"
set-option -g status-keys "emacs"

set-option -g status-bg "#666666"
set-option -g status-fg "#aaaaaa"

set-option -g status-left-length 50
set-option -g status-right " #(date '+%a, %b %d - %I:%M') "

bind-key - split-window -v -c '#{pane_current_path}'
bind-key \ split-window -h -c '#{pane_current_path}'

bind -n S-Left resize-pane -L 2
bind -n S-Right resize-pane -R 2
bind -n S-Down resize-pane -D 1
bind -n S-Up resize-pane -U 1

bind -n C-Left resize-pane -L 10
bind -n C-Right resize-pane -R 10
bind -n C-Down resize-pane -D 5
bind -n C-Up resize-pane -U 5

bind c new-window -c "#{pane_current_path}"

set -g base-index 1
set -g renumber-windows on

bind-key b break-pane -d

bind-key C-j choose-tree

# Use vim keybindings in copy mode
setw -g mode-keys vi

# Setup 'v' to begin selection as in Vim
bind-key -T copy-mode-vi v send -X begin-selection
bind-key -T copy-mode-vi y send -X copy-pipe-and-cancel "reattach-to-user-namespace pbcopy"

# Update default binding of `Enter` to also use copy-pipe
unbind -T copy-mode-vi Enter
bind-key -T copy-mode-vi Enter send -X copy-pipe-and-cancel "reattach-to-user-namespace pbcopy"
```
Once you've done this, you'll need to run `C-b source-file ~/.tmux.conf` once.

You'll also want to add the following to a new file called `/usr/local/bin/tat`, and to run `chmod 755 /usr/local/bin/tat` afterwards to change permissions.

```
#!/bin/sh
#
# Attach or create tmux session named the same as current directory.

session_name="$(basename "$PWD" | tr . -)"

session_exists() {
  tmux list-sessions | sed -E 's/:.*$//' | grep -q "^$session_name$"
}

not_in_tmux() {
  [ -z "$TMUX" ]
}

if not_in_tmux; then
  tmux new-session -As "$session_name"
else
  if ! session_exists; then
    (TMUX='' tmux new-session -Ad -s "$session_name")
  fi
  tmux switch-client -t "$session_name"
fi
```
