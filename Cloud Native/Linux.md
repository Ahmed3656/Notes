## Shells
#### 1. **sh (Bourne Shell)**
- **Origin:** Original Unix shell by Stephen Bourne.
- **Location:** `/bin/sh`
- **Focus:** Portability and simplicity.
- **Features:**
    - Minimal scripting features (no arrays, no functions with local variables).
    - Only supports `[` for conditions (no `[[`).
    - Doesn’t support arithmetic `(( ))` expressions.
    - Limited string manipulation.
    - **Pipeline behavior:** Only returns exit status of the **last command**.
    - Lacks command history and tab completion.
    - Limited POSIX compliance (some older versions).
    - Smallest footprint → suitable for embedded systems.
- **Use Case:** POSIX scripts, system boot scripts.

<hr class="hr-light"/>

#### 2. **bash (Bourne Again Shell)**
- **Origin:** GNU Project, enhanced `sh`.
- **Location:** `/bin/bash`
- **Focus:** Scripting power + interactivity.
- **Features:**
    - Backward-compatible with `sh`.
    - Arrays (indexed, not associative by default).
    - Functions with `local` variables.
    - `[[ ... ]]` for advanced conditions.
    - `(( ))` for arithmetic evaluation.
    - String manipulation (`${var:offset:length}`, `${var/pattern/replacement}`).
    - Associative arrays (from Bash 4.x+).
    - `set -o pipefail`: fails pipeline on **any** command failure (default is last).
    - Brace expansion: `{1..10}`, `{a,b,c}`
    - Command history.
    - Tab and file name autocompletion.
    - Command substitution with `$(...)` or backticks.
    - `**` globbing (recursive with `shopt -s globstar`).
    - Script debugging via `set -x`, `set -e`, etc.
    - Job control (`&`, `fg`, `bg`, `jobs`).
- **Use Case:** Most modern scripts, general Linux use.

<hr class="hr-light"/>

#### 3. **zsh (Z Shell)**
- **Origin:** Paul Falstad.
- **Location:** `/bin/zsh`
- **Focus:** Interactivity + scripting enhancements.
- **Features (all of Bash + more):**
    - Full Bash compatibility.
    - Powerful globbing (`ls **/*.txt`).
    - Named directories with `~name`.
    - Enhanced tab completion (context-aware).
    - Path expansion (`cd ~dev/project/**/models`)
    - Spelling correction (`cd /ect` → `/etc`).
    - `autoload` for lazy loading functions.
    - Plugins + themes (oh-my-zsh, zinit).
    - Prompt customization with segments.
    - Shared history across sessions.
    - Inline syntax highlighting (with plugin).
    - Recursive glob (`**`) is built-in.
    - Floating point arithmetic support.
    - Supports associative arrays.
    - Supports `PIPESTATUS` like Bash.
    - Can emulate other shells (`emulate sh`, etc.).
- **Use Case:** Power user shell, developers, terminal productivity.

<hr class="hr-light"/>

#### 4. **Fish (Friendly Interactive Shell)**
- **Origin:** Axel Liljencrantz (2005).
- **Location:** `/usr/bin/fish`
- **Focus:** User-friendly, modern syntax for daily interactive use.
- **Features:**
	- Syntax is readable and expressive: `set foo bar` instead of `foo=bar`.
	- No `[[` or `$(( ))`; uses consistent `if`, `for`, `switch`, etc.
	- **Autosuggestions** from history and context.
	- **Syntax highlighting** and error messages by default.
	- Tab completion that **learns and adapts**.
	- Universal variables: persist across sessions.
	- Arrays and lists are native: `set fruits apple banana grape`.
	- Web-based configuration tool (`fish_config`).
	- Does not support POSIX shell syntax.
	- No `export` needed for variables (`set -x var value`).
	- Command substitution with `(...)` instead of `$(...)`.
	- Doesn’t support `source ~/.bashrc` — uses `config.fish`.
- **Use case:** 
	- Daily driver shell for developers and power users who want clarity and rich interaction.
	- Not ideal for writing system scripts — fallback to bash/sh for portability.

<hr class="hr-light"/>

#### 5. **ksh (Korn Shell)**
- **Origin:** David Korn (AT&T)
- **Location:** `/bin/ksh`
- **Focus:** Performance and advanced scripting.
- **Features:**
    - Backward-compatible with `sh`.
    - Arrays (indexed).
    - Arithmetic `(( ))` like Bash.
    - Functions with `typeset` for local variables.
    - Co-processes with `|&` and `read -p`.
    - Command substitution via `$(...)`.
    - Associative arrays (in ksh93).
    - Floating point support (in ksh93).
    - Enhanced string manipulation.
    - Built-in math functions.
    - Job control.
    - Supports `PIPESTATUS` array.
    - Globbing not as powerful as zsh.
- **Use Case:** Enterprise systems, advanced scripts, high-performance shell tasks.

<hr class="hr-light"/>

#### 6. **csh (C Shell)**
- **Origin:** Bill Joy (BSD)
- **Location:** `/bin/csh`
- **Focus:** C-like syntax for programming familiarity.
- **Features:**
    - Syntax resembles C language (`if (x == 1) then ... endif`)
    - Poor scripting support: inconsistent error handling, no `trap`, limited I/O redirection.
    - Lacks functions and arrays.
    - Aliases and history support.
    - Supports job control (`jobs`, `fg`, `bg`)
    - Supports command history (first Unix shell to do so).
    - No arithmetic `(( ))`.
    - Limited string operations.
- **Use Case:** Historical only, discouraged today for scripting.

<hr class="hr-light"/>

#### 7. **tcsh (TENEX C Shell)**
- **Origin:** Extended csh.
- **Location:** `/bin/tcsh`
- **Focus:** Interactive improvements over `csh`.
- **Features:**
    - All of `csh` +:
    - Command-line editing (emacs/vi-style).
    - Tab completion (programmable).
    - Better history and recall.
    - Directory stack.
    - Filename globbing.
- **Use Case:** Users who like `csh` syntax with better interactivity.

<hr class="hr-light"/>

#### 8. **PowerShell (pwsh)**
- **Origin:** Microsoft (2006 Windows, 2016 cross-platform as PowerShell Core)
- **Location:** `/usr/bin/pwsh` (Core version)
- **Focus:** Cross-platform automation, object-oriented shell scripting
- Features:
	- Object pipeline: `|` passes **structured objects**, not text.
	- Full access to .NET APIs (on Windows/Linux/macOS).
	- Cmdlets instead of Unix-like commands: `Get-ChildItem`, `Set-Location`.
	- Native error handling: `try`, `catch`, `finally`.
	- Complex scripting capabilities: parameter binding, modules, functions.
	- Supports structured data out-of-the-box: JSON, XML, CSV.
	- Script extensions: `.ps1`.
	- Cross-platform modules (Azure, GitHub, Docker).
	- Custom prompt with full theming (integrates with Oh My Posh).
	- Parallel execution with `ForEach-Object -Parallel`.
- Use Case:
	- IT automation, DevOps, and scripting across OSes — especially in hybrid environments.
	- Heavier than POSIX shells; not ideal for tiny scripts or minimal systems.

<hr class="hr-light"/>

#### 9. **Nushell (nu)**
- **Origin:** Jonathan Turner (ex-Rust + Microsoft dev), 2019
- **Location:** `/usr/bin/nu`
- **Focus:** Shell with structured data, tables, and modern pipeline composition
- Features:
	- Everything is a **table** — not text or strings.
	- `ls` returns a **typed table**, not a raw list.
	- Commands are composable: `ls | where size > 1mb | sort-by modified`.
	- `open`, `get`, `to json`, `to csv` for file formats (JSON/YAML/TOML).
	- Uses its own DSL syntax (not bash-like): `let x = 42`.
	- Built-in support for web APIs: `http get`, `fetch`, `post`.
	- Clean error messages and helpful hints.
	- Strong typing: numbers, strings, booleans, tables.
	- Plugin architecture is evolving.
	- Extensible via Rust and WebAssembly.
- Use Case:
	- Ideal for **data manipulation, querying**, and modern DevOps-style CLI tasks.
	- Still maturing — not a drop-in replacement for POSIX scripting.

<hr class="hr-light"/>

#### 10. **Elvish**
- **Origin:** Xu Cheng (2016)
- **Location:** `/usr/local/bin/elvish`
- **Focus:** Functional shell with structured pipelines and programming power
- Features:
	- Functional-inspired syntax: `put`, `each`, `use`, etc.
	- Native support for **structured values** (lists, maps).
	- Pipelines pass values (not just strings): `put 1 2 3 | each [x]{ echo $x }`.
	- Namespaces and modular scripting: `use unix` to access system commands.
	- Advanced prompt customization.
	- Mutable and immutable variables.
	- Interactive features: autosuggestions, colors, history search.
	- Script-friendly with `.elv` files.
- Use Case:
	- Power users or devs who prefer **functional pipelines** and want deep scripting control.
	- Niche, experimental — not for mass deployment or system-level scripts.

---

#### 11. **Xonsh**
- **Origin:** Scopatz et al. (Python community, ~2015)
- **Location:** `/usr/bin/xonsh`
- **Focus:** Python-powered shell — fusion of shell and full Python
- Features:
	- Use **Python directly** in shell: `echo @(3 * 7)` → `21`.
	- Environment variables as Python objects: `env['PATH']`.
	- Combine shell and Python: `for i in $(ls): print(i)`.
	- Supports both `bash`-like syntax and Python scripting.
	- Prompt customization using Python expressions.
	- Powerful tab completion and syntax highlighting.
	- Native REPL for experimentation.
	- Script files use `.xsh` extension.
- Use Case:
	- Devs who already use Python and want it in their shell.
	- Excellent for automation, data tasks, and hybrid shell scripting.

<hr class="hr-light"/>

#### Summary
- when you need broad compatibility and scripting support —> **zsh**
- when you want a smart, user-friendly shell for daily use —> **fish**
- when you want Python as part of your shell language —> **xonsh**
- when you're automating across platforms using structured data —> **pwsh**
- when you're manipulating JSON, tables, or structured files —> **nushell**
- when you want a functional shell with structured pipelines —> **elvish**
- when working in minimal or embedded environments —> **dash**

---
## Directories (file system)
- /home      —> contains home directories for all **non-root** users
- /root         —> contains the root user home directory
- /bin           —> contains executables for all essential **user** binaries (commands)
- /sbin         —> contains executables for all essential **system** binaries (commands). Only users with **superuser** privilege are allowed to  run/execute these programs
- /lib            —> contains essential shared libraries which are executed from /bin and /sbin
- /usr           —> contains user home directories
- /usr/local  —> contains programs that you install on the computer (third party applications)
- /opt          —> contains optional third party applications
- /boot        —> contains files required for booting
- /etc           —> contains configurations for system-wide applications
- /dev          —> contains devices files (hardware)
- /var           —> variable data that changes frequently during system operation like logs, temp, cache data, runtime data, databases, etc.
- /tmp         —> contains temporary files

---

 ## Commands
##### user@(server/machine/host name):(directory where "~" is home)(privilege where "$" is regular user and "#" is the root user)

- uname -a: displays the operating system and the kernel information
- pwd: stands for "print working directory" where it prints where you are currently standing
- ls: list folders and files found in the current directory
- ls -l: lists everything in the current directory as a list with extra information
- ls -la: lists everything in the current directory as a list with extra information and includes hidden files/directories
- lscpu: displays the CPU's information
- lsmem: displays the memory's information
- cat: read, display, combine, or create text files
- curl: sending or receiving data over the internet (Web requests)
- clear: clears the current terminal
- history: displays all of the commands used previously. It also accepts a number argument if you want a specific amount of commands to only appear
- cd: stands for "change directory" and transfers you to the destined directory
- mkdir: stands for "make directory" where it creates a directory
- touch: creates a file in the current directory
- rm: deletes the selected file
- rm -r: deletes the selected directory even if it isn't empty where "-r" stands for recursively remove
- rm name*: deletes all the files that start with "name"
- rm -r name*: deletes all the directories recursively that start with "name"
- mv: renames the selected file/directory
- cp: copies the selected file to a new file u create/select
- cp -r: copies the selected directory recursively
- adduser: adds a new user
- apt: is the package manager command which allows you to manage packages
	install
	remove
	update
	list
	list --upgradable
- snap: is another package manager which allows you to manage packages
- sudo: grants superuser privilege to run commands like adduser, apt, and snap
- su - user: login as the specified user

---

## Package Manager
In ubuntu the package manager included is APT(Advanced Package Tool) which is the traditional package manager.
APT installs .deb packages from official repositories.
It relies on system libraries and shared dependencies.
APT requires manual updates.
APT is faster as it integrates packages directly with the system.
APT stores packages in /usr/bin/, /usr/lib/, etc.

Snap(Snappy) is another package manager which is a universal package manager.
Snap installs self-contained packages from the Snap Store.
It bundles all required dependencies within the package.
Snap updates packages automatically in the background.
Snap is slower as it runs applications in a sandbox.
Snap stores packages in /var/lib/snapd/ and /snap/.

Summary:
	APT is best for **lightweight, system-integrated applications**.
	Snap is useful for **cross-distro compatibility and the latest software versions**.
- Vim
	
- last