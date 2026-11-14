# Minishell

A small Unix shell written in C, modelled on bash. It reads a command line, parses it into a syntax tree and runs it, with pipes, redirections, logical operators and its own builtins.

This is a team project from the 42 curriculum, done at 1337 School.

## Features

- A prompt with line editing and history (readline)
- Runs programs found through `PATH`, or given by a relative or absolute path
- Pipes: `cmd1 | cmd2 | cmd3`
- Redirections: `<`, `>`, `>>`, and here-documents with `<<`
- Logical operators `&&` and `||`, with parentheses to group commands
- Wildcard `*` for file names in the current directory
- Variable expansion: `$VAR` and `$?` (the exit status of the last command)
- Quotes: single quotes keep the text as it is, double quotes still expand variables
- Builtins: `echo` (with `-n`), `cd`, `pwd`, `export`, `unset`, `env`, `exit`
- Signals: `Ctrl-C` gives a new prompt, `Ctrl-D` leaves the shell, `Ctrl-\` does nothing at the prompt
- Exit statuses like bash: 127 for a command that does not exist, 126 for one that cannot be executed, 2 for a syntax error

There is no `;` operator: one line holds one command, or several joined with `|`, `&&` and `||`.

## Requirements

- `cc` and `make`
- The readline library with its headers (`libreadline-dev` on Debian and Ubuntu)
- `bash`, which the Makefile uses to show progress while compiling

## Build and run

```sh
git clone https://github.com/mfk-ayoub/Minishell.git
cd Minishell
make
./minishell
```

| Command | What it does |
| --- | --- |
| `make` | Builds `minishell` (object files go in `bin/`) |
| `make clean` | Removes the object files |
| `make fclean` | Removes the object files and `minishell` |
| `make re` | Rebuilds from scratch |

## Examples

```
(minishell)-> echo hello | tr a-z A-Z
HELLO
(minishell)-> echo one > out.txt && cat out.txt
one
(minishell)-> false || echo fallback
fallback
(minishell)-> (echo a && echo b) | wc -l
2
(minishell)-> echo *.txt
a.txt b.txt out.txt
(minishell)-> export NAME=ayoub
(minishell)-> echo "hi $NAME" 'and $NAME'
hi ayoub and $NAME
(minishell)-> cat << EOF
> line by $USER
> EOF
line by ayoub
(minishell)-> nosuchcmd
minishell: nosuchcmd: command not found
(minishell)-> echo $?
127
(minishell)-> echo a | | b
syntax error
```

## How it works

Each line goes through the same steps:

1. **Syntax check.** Unclosed quotes, misplaced operators and unbalanced parentheses stop the line here with `syntax error`.
2. **Tokenizing.** The line is cut into tokens, and each token gets a type: command, option, pipe, `&&`, `||`, redirection, file, delimiter, parenthesis.
3. **Expansion.** Variables are replaced by their values, then `*` patterns by the matching file names.
4. **Here-documents.** The text of every `<<` is read before anything runs.
5. **Quote removal.** Quotes have done their job and are dropped.
6. **Syntax tree.** The tokens become a tree: operators are the nodes, commands are the leaves, parentheses become subtrees.
7. **Execution.** The tree is walked. Each command is run in a child process with `fork` and `execve`, pipes and redirections are set up with `pipe` and `dup2`, and `&&` and `||` decide from the exit status whether the right side runs. Builtins that change the shell itself, like `cd`, `export` and `exit`, run without a child process.

## Layout

```
.
├── Makefile
├── includes/        headers: structures, token types, prototypes
├── utils/           main loop and initialisation
├── lparsing/        parsing
│   ├── syntax_error/
│   ├── tokenizer/
│   ├── expander/    $VAR and $?
│   ├── wildcards/
│   ├── heredoc/
│   ├── export/
│   ├── linked_list/ the token list
│   └── tree/        building the syntax tree
├── executor/        running the tree: commands, pipes, redirections, && and ||
├── bulutins/        echo, cd, pwd, env, unset, exit
├── signal/          Ctrl-C, Ctrl-D, Ctrl-\
└── lib/libft/       our C helper library
```

## Authors

- **zelkalai**: parsing (tokenizer, expansion, wildcards, here-documents, syntax tree)
- **ayel-mou** (Ayoub Elmouafik): execution (commands, pipes, redirections, builtins, signals)
