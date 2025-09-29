---
title: "Switching My Shell Prompt to Starship"
date: 2025-09-23 10:00:00 +0200
categories: [Tech]
tags: [terminal, starship, zsh]
---

Recently I became aware of Starship, which is supposed to be a cross shell prompt. Given that I usually change between BASH on linux and ZSH on macOS I thought this could be a good option. I am / was leveraging the `gnzh` theme for oh-my-zsh and wanted to have that same feeling on Starship but with the 'new' stuff:

- Git branch & status  
- Kubernetes context  
- Terraform workspace  
- Python version  

This is my end configuration:

```toml
add_newline = true

format = """
╭─$username@$hostname $directory$git_branch$git_status$kubernetes$terraform$python
╰─$character """

# Username
[username]
style_user = "green"
show_always = true
format = "[$user]($style)"

# Hostname
[hostname]
ssh_only = false
style = "green"
format = "[@$hostname]($style)"

# Directory
[directory]
style = "blue"
truncation_length = 3
truncate_to_repo = false
format = "[$path]($style) "

# Git branch
[git_branch]
symbol = "🌱 "
style = "cyan"
format = "[$symbol$branch]($style) "

# Git status
[git_status]
style = "red"
format = "[$all_status$ahead_behind]($style) "
conflicted = "="
stashed    = "$"
modified   = "!"
staged     = "+"
untracked  = "?"
renamed    = "»"
deleted    = "✘"
ahead      = "⇡${count}"
behind     = "⇣${count}"

# Kubernetes
[kubernetes]
symbol = "☸️ "
style = "yellow"
format = "[$symbol$context]($style) "

# Terraform
[terraform]
symbol = "💠 "
style = "magenta"
format = "[$symbol$workspace]($style) "

# Python
[python]
symbol = "🐍 "
style = "red"
format = "[$symbol$version]($style) "

# Prompt character (arrow)
[character]
success_symbol = "[➤](bold green)"
error_symbol   = "[➤](bold red)"
```

Overall I find the experience to be the same and it does seem a bit snappier.