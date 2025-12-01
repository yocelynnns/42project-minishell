# Minishell

**As beautiful as a shell** - A custom UNIX shell implementation with command execution, piping, redirection, and built-in commands.

## Project Information

| Field | Description |
|-------|-------------|
| **Program Name** | `minishell` |
| **Files** | `Makefile`, `*.h`, `*.c` |
| **Makefile Targets** | `NAME`, `all`, `clean`, `fclean`, `re` |
| **External Functions** | `readline`, signal handlers, file operations, process management |
| **Libft Authorized** | Yes (included in project) |

## 🎯 Project Overview

Minishell is a fully functional UNIX shell that replicates core Bash functionality including:
- **Command execution** with PATH resolution
- **Pipes and redirections** (`|`, `>`, `>>`, `<`, `<<`)
- **Environment variable expansion** (`$VAR`, `$?`)
- **Built-in commands** (`echo`, `cd`, `pwd`, `export`, `unset`, `env`, `exit`)
- **Signal handling** (Ctrl+C, Ctrl+D, Ctrl+\)
- **Command history** with readline

## 🏗️ Architecture & File Structure

```
minishell/
├── Makefile
├── inc/
│   └── minishell.h          # Main header with all structs and prototypes
├── libft/                   # Custom libft implementation
│   ├── Makefile
│   ├── ft_*.c              # Standard libft functions
│   └── libft.h
└── src/
    ├── main.c              # Program entry point and initialization
    ├── shell_loop.c        # Main shell loop and input handling
    ├── signals/            # Signal handling implementation
    │   ├── signals.c
    │   └── signals_helper.c
    ├── parsing/            # Lexing and parsing subsystem
    │   ├── lexing.c        # Tokenization
    │   ├── token.c         # Token management
    │   ├── token_handle*.c # Token processing
    │   ├── expansion.c     # Variable expansion
    │   ├── ast.c           # Abstract Syntax Tree
    │   └── ast_redir.c     # Redirection AST nodes
    ├── execute/            # Command execution
    │   ├── exec.c          # Main execution logic
    │   ├── exec_path.c     # PATH resolution
    │   ├── heredoc.c       # Here document handling
    │   └── exec_*.c        # Execution helpers
    ├── builtins/           # Built-in commands
    │   ├── echo.c
    │   ├── cd.c
    │   ├── pwd.c
    │   ├── export.c
    │   ├── unset.c
    │   ├── env.c
    │   └── exit.c
    ├── env/                # Environment management
    │   ├── env_init.c
    │   ├── env_utils.c
    │   └── env_*.c
    └── utils/              # Utility functions
        ├── free.c          # Memory management
        ├── utils.c
        └── *.c
```

## 🔧 Core Features

### Mandatory Requirements
- ✅ **Interactive Prompt** with readline support
- ✅ **Command History** using readline history
- ✅ **Command Execution** with PATH resolution and absolute/relative paths
- ✅ **Pipes** (`|`) for command chaining
- ✅ **Redirections**:
  - Input (`<`)
  - Output (`>`)
  - Append (`>>`)
  - Here document (`<<`)
- ✅ **Environment Variables** expansion (`$VAR`)
- ✅ **Exit Status** (`$?`) expansion
- ✅ **Signal Handling**:
  - Ctrl+C: New prompt on new line
  - Ctrl+D: Exit shell
  - Ctrl+\: Do nothing
- ✅ **Built-in Commands**:
  - `echo` with `-n` option
  - `cd` with relative/absolute paths
  - `pwd`
  - `export`
  - `unset`
  - `env`
  - `exit`

### Advanced Features
- **Quote Handling**: Single quotes (`'`) and double quotes (`"`)
- **Variable Expansion**: `$VAR` and `$?` in double quotes
- **Memory Management**: No leaks (except readline internals)
- **Error Handling**: Comprehensive error messages and status codes

## 🛠️ Build & Usage

### Compilation
```bash
make
```

### Running the Shell
```bash
./minishell
```

### Example Usage
```bash
# Basic commands
minishell$ ls -la
minishell$ echo "Hello World"

# Pipes and redirections
minishell$ cat file.txt | grep "pattern" > output.txt
minishell$ ls -l | wc -l

# Environment variables
minishell$ echo $PATH
minishell$ export MY_VAR=value
minishell$ echo $MY_VAR

# Here documents
minishell$ cat << EOF
> line 1
> line 2
> EOF

# Built-in commands
minishell$ cd /path/to/directory
minishell$ pwd
minishell$ export NEW_VAR=hello
minishell$ unset NEW_VAR
minishell$ exit
```

## 🔍 Technical Implementation

### Data Structures
```c
// Main shell structure
typedef struct s_minishell {
    t_env       *env;      // Environment variables
    t_token     *token;    // Token list
    t_ast_node  *ast;      // Abstract Syntax Tree
    int         exit;      // Exit status
    char        **env2;    // Environment array
} t_minishell;

// Token types for lexing
typedef enum s_token_type {
    WORD, PIPE, REDIRECT_IN, REDIRECT_OUT, HEREDOC, APPEND
} t_token_type;

// AST node types for parsing
typedef enum s_ast_node_type {
    AST_PIPELINE, AST_COMMAND, AST_REDIRECT, AST_WORD
} t_ast_node_type;
```

### Execution Pipeline
1. **Input Reading**: Readline with signal handling
2. **Lexing**: Tokenize input string into tokens
3. **Parsing**: Build Abstract Syntax Tree from tokens
4. **Expansion**: Handle variable and quote expansion
5. **Execution**: Execute commands with proper redirections
6. **Cleanup**: Free memory and prepare for next command

### Signal Handling
```c
// Global variable for signal tracking
int g_sigint = 0;

void signal_reset_prompt(int signo) {
    g_sigint = 1;
    write(1, "\n", 1);
    rl_replace_line("", 0);
    rl_on_new_line();
    rl_redisplay();
}
```

## 🧪 Testing & Validation

### Basic Functionality Tests
```bash
# Test built-in commands
minishell$ echo "test"
minishell$ pwd
minishell$ cd /tmp && pwd
minishell$ export TEST=value && echo $TEST
minishell$ unset TEST && echo $TEST

# Test pipes and redirections
minishell$ echo "hello" | cat
minishell$ ls > output.txt && cat output.txt
minishell$ cat < output.txt

# Test error handling
minishell$ invalid_command
minishell$ cd /nonexistent
minishell$ exit 42
```

### Signal Testing
```bash
# Test Ctrl+C (should show new prompt)
minishell$ sleep 10
# Press Ctrl+C

# Test Ctrl+D (should exit shell)
minishell$ # Press Ctrl+D
```

### Memory Leak Checking
```bash
# Use valgrind to check for memory leaks
valgrind --leak-check=full --show-leak-kinds=all ./minishell
```

## 🚀 Implementation Highlights

### Robust Parsing System
- **Lexical Analysis**: Tokenizes input with quote handling
- **Syntax Tree**: Builds AST for complex command structures
- **Error Recovery**: Handles syntax errors gracefully

### Advanced Execution
- **Process Management**: Proper fork/exec with signal handling
- **Pipe Chaining**: Multiple pipe support
- **File Descriptor Management**: Proper redirection handling

### Environment Management
- **Variable Storage**: Linked list for efficient lookups
- **Export/Unset**: Proper environment manipulation
- **PATH Resolution**: Efficient command location

## 🎯 Evaluation Guide

### During Defense
1. **Demonstrate Basic Features**: Command execution, pipes, redirections
2. **Show Built-in Commands**: cd, echo, export, etc.
3. **Test Signal Handling**: Ctrl+C, Ctrl+D behavior
4. **Explain Architecture**: Lexing, parsing, execution pipeline
5. **Discuss Memory Management**: Allocation and freeing strategy

### Key Technical Points
- **Process Creation**: fork(), execve(), waitpid() usage
- **File Descriptor Manipulation**: dup2(), pipe() for redirections
- **Signal Handling**: Proper signal masking and restoration
- **Memory Management**: Comprehensive cleanup routines

## ⚠️ Important Notes

- **Global Variable**: Only one global variable allowed for signals
- **Memory Leaks**: readline() may leak, but your code shouldn't
- **Error Codes**: Follow Bash exit status conventions
- **Norm Compliance**: Strict adherence to 42 coding standards
- **Edge Cases**: Handle quotes, empty commands, syntax errors

## 👥 Team Contributions

This project was developed in collaboration with **[@Mess925](https://github.com/Mess925)**.

### Built-in Commands & Environment System
**Implemented by [@Mess925](https://github.com/Mess925)**
- `src/builtins/` `src/env/` `src/signals/`
- `echo`, `cd`, `pwd`, `export`, `unset`, `env`, `exit` commands
- Environment variable management and expansion
- Signal handling (Ctrl+C, Ctrl+D, Ctrl+\)

### Core Shell Infrastructure  
**Implemented by [@yocelynnns](https://github.com/yocelynnns)**
- `src/parsing/` `src/execute/` `src/main.c`, `src/shell_loop.c`
- Lexical analysis and tokenization
- Abstract Syntax Tree (AST) construction
- Command execution and pipeline management
- Redirection handling (`<`, `>`, `>>`, `<<`)
- Main program flow

---

**Pro Tip**: The key to a successful minishell is robust error handling and proper memory management. Test extensively with complex command combinations and edge cases!
