To install Python and pip on a Linux system (e.g., Ubuntu), follow these steps:

1. Update Package List
First, ensure your package list is up-to-date:
```
sudo apt update
```

2. Install Python 3
Install Python 3 using the following command:

```
sudo apt install python3
```

This installs Python 3, which is the recommended version for most modern Python applications.

3. Install pip for Python 3
Install pip, the Python package manager, for Python 3:

```
sudo apt install python3-pip
```

4. Verify Python and pip Installation
After installation, you can verify that Python and pip are installed correctly by checking their versions:
```
python3 --version
pip3 --version
```

This should return the installed versions of Python 3 and pip.

5. Install virtualenv and virtualenvwrapper (Optional but recommended)
If you plan to use virtualenvwrapper for managing virtual environments, you need to install virtualenv and virtualenvwrapper:

```
sudo apt install python3-virtualenv
pip3 install virtualenvwrapper
```

6. Configure virtualenvwrapper (Optional)
Once virtualenvwrapper is installed, you need to set up some environment variables:

Open your .bashrc (or .zshrc for Zsh users) file:

```
nano ~/.bashrc
```

Add the following lines to configure virtualenvwrapper:

```
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export VIRTUALENVWRAPPER_VIRTUALENV=$HOME/.local/bin/virtualenv
source $HOME/.local/bin/virtualenvwrapper.sh
export WORKON_HOME=$HOME/.virtualenvs
```

save change with : CTRL + O and then CTRL X
Apply the changes: 
```
source ~/.bashrc
```

7. Create and Manage Virtual Environments
Now you can create and manage virtual environments using virtualenvwrapper. For example:

Create a new virtual environment:
```
mkvirtualenv myenv
```
Activate the virtual environment:

```
workon myenv
```

Deactivate the virtual environment:
```
deactivate
```
That's it! You should now have Python, pip, and virtualenvwrapper set up on your Linux system, allowing you to manage Python packages and virtual environments.
