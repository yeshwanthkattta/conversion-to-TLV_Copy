# Installation and Debugging Report for EDA Tools on macOS (Apple Silicon)

Author: **Yeshwanth Reddy Katta**  
Target OS: **macOS (Apple Silicon)**  

---

## Table of Contents

1. [Initial Installation Script Creation](#initial-installation-script-creation)  
2. [Homebrew Dependency Installation](#homebrew-dependency-installation)  
3. [Handling Python (PEP 668 Restriction)](#handling-python-pep-668-restriction)  
4. [Updated Python Dependency Script](#updated-python-dependency-script)  
5. [Yosys Build Process](#yosys-build-process)  

---

## Initial Installation Script Creation

1. **Shell script creation**  
   - A shell script named `install_edatools.sh ` was created to automate the installation of dependencies.  
   - The script was created using either `nano` or Visual Studio Code.

2. **Making the script executable**  
   ```bash
   chmod +x install_edatools.sh
   
3. ** Running the script**
```bash
./install_edatools.sh 
```
**Homebrew Dependency Installation**

In the script, the following packages are installed via Homebrew:

Core build tools: bison, flex, readline, gawk, libffi, graphviz, pkg-config, boost, zlib, tcl-tk, llvm, git, and python@3.13

Note: You may encounter warnings showing some packages are already installed or up to date, which is normal if you already had them.

**Handling Python (PEP 668 Restriction)**
Problem:
When installing Python packages globally with Homebrew’s Python, an error appears:

error: externally-managed-environment

**This is due to Python’s PEP 668 compliance, which restricts global pip installs in an externally managed environment.**

Solution:

Create a virtual environment:

```bash
python3 -m venv ~/.venv_edatools
```
Activate the virtual environment:

```bash
source ~/.venv_edatools/bin/activate
``
Upgrade pip and install your required packages inside the virtual environment:

```bash
pip install --upgrade pip
pip install pynput openai 
```
Updated Python Dependency Script
Below is an example of how the Python dependency check might be included in your install_edatools.sh script (or in a separate shell script):


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
Use this function within your main script if you want to ensure you’re inside the Python virtual environment before installing dependencies.

## Yosys Build Process
Initial Error
When attempting to build Yosys on macOS (Apple Silicon), the following error was encountered:

nginx

fatal error: 'map' file not found
This typically indicates missing C++ standard headers or compiler configuration issues.

1. Initial Setup & Prerequisites
Attempted using Homebrew commands:
```bash
brew tap Homebrew/bundle && brew bundle
```
This was found to be deprecated, leading to manual installation of each required tool.

Manually installed via Homebrew:
```bash
brew install bison flex readline gawk libffi graphviz pkg-config python@3.11 boost zlib tcl-tk llvm
```
Required tools:

GNU Flex and Bison (for parsing)

Make (or gmake), Python, Clang/GCC

Optional (but often needed): readline, libffi, Tcl, zlib, Graphviz, Xdot

**2. MacPorts & Compiler Configuration**
When Homebrew-based builds still failed, MacPorts was tried as an alternative.

Check MacPorts installation

```bash
port version
```
Should output a version (e.g., Version: 2.8.1).
If missing, install MacPorts from MacPorts.org.

**Install dependencies via MacPorts**

```bash

sudo port install bison flex readline gawk libffi graphviz \pkgconfig python311 boost zlib tcl
```
Update environment

```bash
export PATH="/opt/local/bin:/opt/local/sbin:/Users/yeshwanthreddykatta/anaconda3/bin:$PATH"
```
Add the above line to your ~/.zshrc (or appropriate shell configuration file), then:
```bash
source ~/.zshrc
```
Verify Clang version

```bash
which clang-16
```
clang-16 --version
Ensure you have Clang 16 (or a compatible version) installed via MacPorts or otherwise.

3. Yosys Build Configuration
Clone Yosys repository

```bash
git clone --recurse-submodules https://github.com/YosysHQ/yosys.git
cd yosys
```
Initialize submodules

```bash
git submodule update --init --recursive
```
Clean old builds

```bash
gmake clean
```
Edit Makefile.conf
```bash


CONFIG := clang
CXX := /opt/local/bin/clang++-mp-16
CC := /opt/local/bin/clang-mp-16
CXXFLAGS += -I/opt/local/include
LINKFLAGS += -L/opt/local/lib
LIBS += -ltcl -ltommath -lz
ENABLE_TCL := 1
ENABLE_ZLIB := 1


```
Adjust paths and flags as needed if your setup differs.

Build Yosys

```bash
gmake config-clang
gmake -j$(sysctl -n hw.logicalcpu)
```
or 
```bash
gmake
```
After the build completes successfully, you should have a working yosys binary.
Steps to Install and Test sby and eqy on macOS
*** 1. Install SymbiYosys (sby)**8
  SymbiYosys is a wrapper around Yosys, smtbmc, and model checkers like Yices/Z3.
a. Clone SymbiYosys:
```bash
sudo port install z3
git clone https://github.com/YosysHQ/sby.git
cd sby
```
b. Install it locally:
```bash
sudo make
```
C.Check installation:
```bash
sby --version
```
1: Add Yosys to your PATH
If Yosys is  locally, so its binary is probably located in the yosys folder. Let’s add that directory to your PATH so you can call yosys from anywhere.
```bash
nano ~/.zshrc
```
 add the below line at end of the script
 ```bash
export PATH="$HOME/yosys:$PATH"
```
 Save and apply
 ```bash
source ~/.zshrc
```

Testing SymbiYosys using a basic design
Create a new folder (optional but recommended):
```bash
mkdir -p ~/SymbiYosys/my_project
cd ~/SymbiYosys/my_project
```
Create and save my_design.v:
```bash
nano my_counter.v
```

Paste the below code:(save and exit (Ctrl+O, Enter, then Ctrl+X).)
```bash
module top(input clk, input rst, output reg [3:0] counter);

    always @(posedge clk) begin
        if (rst)
            counter <= 0;
        else
            counter <= counter + 1;
    end

    // Simple assertion: counter should never overflow (not realistic, just for demo)
    always @(posedge clk)
        if (counter == 4'b1111)
            assert(0);

endmodule
```

Create and save my_counter.sby:
```bash
nano my_counter.sby
```
Paste this config:
```bash
[options]
mode bmc
depth 20

[engines]
smtbmc z3

[script]
read -formal my_counter.v
prep -top top

[files]
my_counter.v
```
Run SymbiYosys:
```bash
sby -f my_design.sby
```
You will get a message at end :
failed assertion top._witness_.check_assert_my_design_v_13_5 at my_design.v:13.13-13.22

This means:

There is an assertion at line 13 of our Verilog file

Steps to install EQY:
. Clone the EQY repository

```bash
xgit clone https://github.com/YosysHQ/eqy.git
cd eqy
```
Build and and install 

add this at top of the Make file:
```bash
YOSYS_SRC := /Users/yeshwanthreddykatta/yosys

CXX := clang++
CXXFLAGS := -fPIC -std=c++17 -Wall -I$(YOSYS_SRC) -I$(YOSYS_SRC)/kernel \
  -DYOSYS_NAMESPACE_BEGIN="namespace Yosys {" \
  -DYOSYS_NAMESPACE_END="}"
```
 Build and install :
 ```bash
make
sudo make install
```

Final Notes
Using MacPorts for dependencies and clang-16 proved more reliable on Apple Silicon.
