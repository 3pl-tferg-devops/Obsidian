https://github.com/asdf-community/asdf-kubectl

```zsh
asdf plugin-add kubectl https://github.com/asdf-community/asdf-kubectl.git
```

Install
```zsh
asdf install kubectl 1.26.3
```

Enable autocompletion (add to `~/.zshrc`)
```zsh
vim ~/.zshrc # then add
source <(kubectl completion zsh)
autoload -Uz compinit # only add this if you don't have it elsewhere
compinit
```