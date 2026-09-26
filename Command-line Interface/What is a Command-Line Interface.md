There are two ways through which anyone can interact with a computer:
- Via direct commands (Also called CLI)
- Or by using a graphical interface

A Command Line Interface (CLI) is a text-based user interface that allows users to interact with computer systems, software, or applications by typing commands into a console or terminal window. 
**In short, it is a way of interacting with the computer.**

Unlike graphical user interfaces (GUIs), which use visual elements like icons and menus, a CLI relies exclusively on textual input and output.


---
## Key Features of CLI

- Users type commands at a prompt using a keyboard, and the system executes those commands, displaying results or error messages as text.

- CLIs are highly efficient for experienced users, enabling quick navigation, file management, program execution, and system configuration.
 
- They support automation through scripting, allowing users to save sequences of commands in script files for repeated or complex tasks.

- The interface is usually provided by a program called a shell or command-line interpreter (e.g., Bash in Linux, Command Prompt or PowerShell in Windows.

---
## How CLI Works

- The CLI displays a prompt indicating readiness for input.
![[Pasted image 20250521143504.png]]

- Users enter a command, often followed by parameters or options, and press Enter.

- The command-line interpreter processes the input, executes the command, and returns output or feedback.

- Commands can be chained, redirected, or piped to perform more advanced operations (e.g., sending output from one command as input to another.

---
## CLI vs. GUI

| Feature        | CLI                                    | GUI                         |
| -------------- | -------------------------------------- | --------------------------- |
| Input Method   | Typed text commands                    | Mouse clicks, icons, menus  |
| Resource Use   | Low                                    | Higher                      |
| Learning Curve | Steeper (requires memorizing commands) | Easier for beginners        |
| Automation     | Excellent (supports scripting)         | Limited                     |
| Speed          | Fast for experienced users             | Slower for repetitive tasks |

---
## Common Uses
- System administration and configuration
- Software development and deployment
- File management and automation
- Accessing advanced features not available in GUIs

## Examples
- Windows: Command Prompt, PowerShell
- Linux/Unix: Bash, Zsh, Sh
- macOS: Terminal (Bash or Zsh)

---
## Summary
A CLI is a powerful, efficient way to control computers and software by entering text commands, favored by developers and system administrators for its speed, flexibility, and automation capabilities, despite having a steeper learning curve compared to GUIs.

---
**Terminal:** This is a program (or, historically, a physical device) that provides a _text-based user interface_ where you can type and view text. Modern examples include GNOME Terminal, Windows Terminal, and iTerm2

**CLI (Command Line Interface):** This refers to the _interface_ where you enter text commands for the computer to process. The CLI is the method or mode of interacting with the system via typed commands, as opposed to using a graphical interface
## Bash, CLI, and TUI: How They Relate
- **Bash** is a **shell**—a specific program that interprets and executes commands you type.
- **CLI** (Command Line Interface) is a **type of user interface** where you interact with the system by typing commands (Bash is one example of a shell that provides a CLI)[6](https://superuser.com/questions/487750/whats-the-difference-between-the-terms-shell-and-bash)[13](https://github.com/trinib/Linux-Bash-Commands).
- **TUI** (Text-based User Interface) is a broader term for any user interface that uses text instead of graphics. This can include both simple command lines (CLIs) and more complex, interactive text menus, dashboards, or applications that use keyboard navigation and sometimes mouse input

When you run the command:
```bash
bash myFile.sh
```
here’s what happens:

- **The Bash Shell is Invoked:** The command starts a new instance of the Bash shell (Bourne Again SHell), which is a command-line interpreter[2](https://phoenixnap.com/kb/run-bash-script)[3](https://www.freecodecamp.org/news/bash-scripting-tutorial-linux-shell-script-and-command-line-for-beginners/)[6](https://docs.vultr.com/how-to-create-a-bash-script-and-execute-it).
- **The Script is Read and Executed:** Bash reads the contents of `myFile.sh` line by line and executes each command in the script as if you were typing them directly into the terminal[1](https://stackoverflow.com/questions/2177932/how-do-i-execute-a-bash-script-in-terminal)[3](https://www.freecodecamp.org/news/bash-scripting-tutorial-linux-shell-script-and-command-line-for-beginners/)[6](https://docs.vultr.com/how-to-create-a-bash-script-and-execute-it)[7](https://superuser.com/questions/795837/how-to-execute-a-command-within-a-bash-script).
- **No Need for Executable Permissions:** When you run a script this way (by explicitly calling `bash`), the script file itself does not need to have executable permissions. Bash simply reads and executes the file[1](https://stackoverflow.com/questions/2177932/how-do-i-execute-a-bash-script-in-terminal)[2](https://phoenixnap.com/kb/run-bash-script).
    
- **Script Runs as a Separate Process:** The script runs as a separate process. Any commands or programs started by the script are also run as separate processes[4](https://serverfault.com/questions/985650/is-running-a-script-a-process).
    
- **Shebang Line:** If your script starts with a shebang line (e.g., `#!/bin/bash`), it is ignored in this case because you are explicitly specifying the interpreter (`bash`). The shebang is only used when you run the script directly (like `./myFile.sh`)[3](https://www.freecodecamp.org/news/bash-scripting-tutorial-linux-shell-script-and-command-line-for-beginners/)[6](https://docs.vultr.com/how-to-create-a-bash-script-and-execute-it).
    
- **Arguments:** You can pass arguments to the script after the filename, and they will be available inside the script as `$1`, `$2`, etc.[2](https://phoenixnap.com/kb/run-bash-script)[3](https://www.freecodecamp.org/news/bash-scripting-tutorial-linux-shell-script-and-command-line-for-beginners/).
    

