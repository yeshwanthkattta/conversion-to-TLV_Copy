Installation and Debugging Report for EDA Tools on macOS

Author: Yeshwanth Reddy Katta Target OS: macOS (Apple Silicon)

 # 1. Initial Installation Script Creation

A shell script named install_edatools.sh was created to automate the installation of dependencies:

Installed via nano or VS Code.

Made executable using chmod +x install_edatools.sh.

Run with ./install_edatools.sh.

2. Homebrew Dependency Installation

The script installs:

Core build tools: bison, flex, readline, gawk, libffi, graphviz, pkg-config, boost, zlib, tcl-tk, llvm, git, and python@3.13

Warnings showed already-installed packages were up to date.

3. Handling Python (PEP 668 Restriction)

Problem:
error: externally-managed-environment
Cause: Homebrew’s Python is PEP 668 compliant and restricts global pip installs.
Solution:
python3 -m venv ~/.venv_edatools
source ~/.venv_edatools/bin/activate
pip install --upgrade pip
pip install pynput openai

4. Updated Python Dependency Script

## Function to install Python dependencies
install_python_deps() {
    echo "🐍 Checking if inside virtual environment..."

    # $VIRTUAL_ENV is a special environment variable that only exists when inside a venv
    if [ -z "$VIRTUAL_ENV" ]; then
        echo "❌ Not in a Python virtual environment. Please run:"
        echo "   source ~/.venv_edatools/bin/activate"
        echo "   pip install pynput openai"
        exit 1
    else
        echo "✅ Python virtual environment detected: $VIRTUAL_ENV"
        echo "✅ Dependencies already installed"
    fi
}

 # Yosys Build Process:

Initial Error:
fatal error: 'map' file not found (This indicated missing C++ standard headers)
1. Initial Setup & Prerequisites

Attempted: brew tap Homebrew/bundle && brew bundle — Deprecated

Fix: Manually installed:
brew install bison flex readline gawk libffi graphviz pkg-config python@3.11 boost zlib tcl-tk llvm

Required tools: GNU Flex, Bison, Make, Python, and optional tools like readline, libffi, Tcl, zlib, Graphviz, and Xdot.
(Not able to build using homebrew) so tried with Macports

2. MacPorts & Compiler Configuration

MacPorts installation check:

port version

 Output: Version: 2.8.1

Install MacPorts from: https://www.macports.org/install.php

Dependencies install via MacPorts:

sudo port install bison flex readline gawk libffi graphviz \
    pkgconfig python311 boost zlib tcl

Environment Setup:

export PATH="/opt/local/bin:/opt/local/sbin:/Users/yeshwanthreddykatta/anaconda3/bin:$PATH"

Add the above to ~/.zshrc, save, and run source ~/.zshrc

Verify Clang version:
which clang-16
clang-16 --version

4. Yosys Build Configuration

Clone Yosys:

git clone --recurse-submodules https://github.com/YosysHQ/yosys.git
cd yosys

Initialize submodules:

```bash
git submodule update --init --recursive

Clean old builds:

gmake clean
```
Edit Makefile.conf:

CONFIG := clang
CXX := /opt/local/bin/clang++-mp-16
CC := /opt/local/bin/clang-mp-16
LDLIBS += -ltcl
ENABLE_ZLIB := 0
ENABLE_TCL := 1

Final Build Commands:

gmake config-clang
gmake -j$(sysctl -n hw.logicalcpu)
 Yosys built successfully with MacPorts clang-16.


