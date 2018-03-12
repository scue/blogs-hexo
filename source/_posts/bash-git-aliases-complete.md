---
layout: post
date: 2015-05-20 00:00:00
title: "Git Aliases Tab补全"
description: ""
category: 技术
tags: 
  - Bash
  - Git
---

很喜欢zsh的git aliases，也希望带到其他电脑上使用，或Windows上的Git使用；

于是乎研究了一下Git aliases在Bash下的自动补全，发现确实可行，所以分享一下给大家；

创建文件 `~/.bash_aliases.git`，包含以下内容：

    __linux_git_complete_file=/usr/share/bash-completion/completions/git
    __windows_git_complete_file=/etc/bash_completion.git
    __host_os_type=$(uname -sm)
    
    case ${__host_os_type:0:5} in
        Linux)
            __current_git_complete_file=$__linux_git_complete_file
            ;;
        MINGW)
            __current_git_complete_file=$__windows_git_complete_file
            ;;
        *)
            echo "Warning: .bash_alias.git unknow host os type !"
            ;;
    esac
    
    test -e $__current_git_complete_file && source $__current_git_complete_file
    
    unset __host_os_type
    unset __linux_git_complete_file
    unset __windows_git_complete_file
    unset __current_git_complete_file
    
    # Aliases
    alias g='git'
    alias gst='git status'
    alias gd='git diff'
    alias gdc='git diff --cached'
    alias gl='git pull'
    alias gup='git pull --rebase'
    alias gp='git push'
    alias gc='git commit -v'
    alias gc!='git commit -v --amend'
    alias gca='git commit -v -a'
    alias gca!='git commit -v -a --amend'
    alias gcmsg='git commit -m'
    alias gco='git checkout'
    alias gcm='git checkout master'
    alias gr='git remote'
    alias grv='git remote -v'
    alias grmv='git remote rename'
    alias grrm='git remote remove'
    alias grset='git remote set-url'
    alias grup='git remote update'
    alias grbi='git rebase -i'
    alias grbc='git rebase --continue'
    alias grba='git rebase --abort'
    alias gb='git branch'
    alias gba='git branch -a'
    alias gcount='git shortlog -sn'
    alias gcl='git config --list'
    alias gcp='git cherry-pick'
    alias glg='git log --stat --max-count=10'
    alias glgg='git log --graph --max-count=10'
    alias glgga='git log --graph --decorate --all'
    alias glo='git log --oneline --decorate --color'
    alias glog='git log --oneline --decorate --color --graph'
    alias gss='git status -s'
    alias ga='git add'
    alias gm='git merge'
    alias grh='git reset HEAD'
    alias grhh='git reset HEAD --hard'
    alias gclean='git reset --hard && git clean -dfx'
    alias gwc='git whatchanged -p --abbrev-commit --pretty=medium'
    
    #remove the gf alias
    #alias gf='git ls-files | grep'
    
    alias gpoat='git push origin --all && git push origin --tags'
    alias gmt='git mergetool --no-prompt'
    
    alias gg='git gui citool'
    alias gga='git gui citool --amend'
    alias gk='gitk --all --branches'
    
    alias gsts='git stash show --text'
    alias gsta='git stash'
    alias gstp='git stash pop'
    alias gstd='git stash drop'
    
    # Will cd into the top of the current repository
    # or submodule.
    alias grt='cd $(git rev-parse --show-toplevel || echo ".")'
    
    # Git and svn mix
    alias git-svn-dcommit-push='git svn dcommit && git push github master:svntrunk'
    
    alias gsr='git svn rebase'
    alias gsd='git svn dcommit'
    #
    # Will return the current branch name
    # Usage example: git pull origin $(current_branch)
    #
    function current_branch() {
      ref=$(git symbolic-ref HEAD 2> /dev/null) || \
      ref=$(git rev-parse --short HEAD 2> /dev/null) || return
      echo ${ref#refs/heads/}
    }
    
    function current_repository() {
      ref=$(git symbolic-ref HEAD 2> /dev/null) || \
      ref=$(git rev-parse --short HEAD 2> /dev/null) || return
      echo $(git remote -v | cut -d':' -f 2)
    }
    
    # these aliases take advantage of the previous function
    alias ggpull='git pull origin $(current_branch)'
    alias ggpur='git pull --rebase origin $(current_branch)'
    alias ggpush='git push origin $(current_branch)'
    alias ggpnp='git pull origin $(current_branch) && git push origin $(current_branch)'
    
    # these alias ignore changes to file
    alias gignore='git update-index --assume-unchanged'
    alias gunignore='git update-index --no-assume-unchanged'
    # list temporarily ignored files
    alias gignored='git ls-files -v | grep "^[[:lower:]]"'
    
    # Git Aliases AutoComplete
    type __git_complete >/dev/null 2>&1 && {
        __git_complete   g        _git
        __git_complete   gd       _git_diff
        __git_complete   gdc      _git_diff
        __git_complete   gl       _git_pull
        __git_complete   gup      _git_pull
        __git_complete   gp       _git_push
        __git_complete   gc       _git_commit
        __git_complete   gc!      _git_commit
        __git_complete   gca      _git_commit
        __git_complete   gca!     _git_commit
        __git_complete   gcmsg    _git_commit
        __git_complete   gco      _git_checkout
        __git_complete   gcm      _git_checkout
        __git_complete   gr       _git_remote
        __git_complete   grv      _git_remote
        __git_complete   grmv     _git_remote
        __git_complete   grrm     _git_remote
        __git_complete   grset    _git_remote
        __git_complete   grup     _git_remote
        __git_complete   grbi     _git_rebase
        __git_complete   grbc     _git_rebase
        __git_complete   grba     _git_rebase
        __git_complete   gb       _git_branch
        __git_complete   gba      _git_branch
        __git_complete   gcount   _git_shortlog
        __git_complete   gcl      _git_config
        __git_complete   gcp      _git_cherry_pick
        __git_complete   glg      _git_log
        __git_complete   glgg     _git_log
        __git_complete   glgga    _git_log
        __git_complete   glo      _git_log
        __git_complete   glog     _git_log
        __git_complete   gss      _git_log
        __git_complete   ga       _git_add
        __git_complete   gm       _git_merge
        __git_complete   grh      _git_reset
        __git_complete   grhh     _git_reset
        __git_complete   gwc      _git_whatchanged
        __git_complete   gmt      _git_mergetool
        __git_complete   gk       _gitk
        __git_complete   gsts     _git_stash
        __git_complete   gsta     _git_stash
        __git_complete   gstp     _git_stash
        __git_complete   gstd     _git_stash
        __git_complete   gsr      _git_svn
        __git_complete   gsd      _git_svn
    }

然后，再让 `~/.bashrc` 去包含这个文件：

    echo 'test -e ~/.bash_aliases.git && . ~/.bash_aliases.git' >> ~/.bashrc

然后使用类似命令`gd`、`gst`、`ga`和`gc`等命令简写时，会非常顺手，Enjoy！
