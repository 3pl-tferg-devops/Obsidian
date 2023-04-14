[[how-to-install-zsh-with-apt]]

[[change-your-default-shell-in-linux]]

On the Mac
```sh
vim ~/.ssh/config
Host *
    SendEnv TERM
```

On the raspberry pi
```sh
echo 'export TERM=xterm' >> ~/.zshrc
```

