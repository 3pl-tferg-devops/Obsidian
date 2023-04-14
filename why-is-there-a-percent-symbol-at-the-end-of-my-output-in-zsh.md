When you run a command in the terminal, the shell normally prints the output on a new line after the command completes. 

However, if the output does not end with a newline character, the next shell prompt will appear immediately after the output without a new line in between.

Zsh adds a % symbol to indicate there was no new line at the end of the output and adds a new line so the shell prompt doesn't appear immediately after  the output. 

This symbol can be ignored.