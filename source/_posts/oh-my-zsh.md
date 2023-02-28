---
title: oh-my-zsh
date: 2023-02-28 13:50:35
tags:
---

# 极简的主题
```
PROMPT="%(?:%{$fg_bold[green]%}➜ :%{$fg_bold[red]%}➜ )"
RPROMPT='%{$fg[cyan]%}%c%{$reset_color%}  $(git_prompt_info)'

ZSH_THEME_GIT_PROMPT_PREFIX="%{$fg_bold[blue]%}git:(%{$fg[red]%}"
ZSH_THEME_GIT_PROMPT_SUFFIX="%{$reset_color%} "
ZSH_THEME_GIT_PROMPT_DIRTY="%{$fg[blue]%}) %{$fg[yellow]%}✗"
ZSH_THEME_GIT_PROMPT_CLEAN="%{$fg[blue]%})"
```
# 存放路径
.oh-my-zsh/custom/themes/zman.zsh-theme

# 配置文件
文件: ~/.zshrc
修改配置项: ZSH_THEME="zman"
