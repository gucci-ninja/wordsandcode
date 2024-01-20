+++
title =  "How I Built an AI Assistant for my Operating System"
date =  2024-09-08T21:35:59-04:00
menu = "main"
+++

```bash
archero how do I search for a file with the word skibidi

OpenAI: Use the following command to search for files with the word "skibidi" in their name in the current directory and its subdirectories:

    find . -type f -iname "*skibidi*"
```

### Why build an AI assistant for Arch Linux

My operating system of choice is Arch Linux. I installed it around 6 years ago and have been too lazy to switch to anything else. Also, it’s worked pretty well for me all these years so I don’t have any reason to. Of course, there are times when I fall behind on updates and everything starts to break. Things start to break so much that even updating won’t solve them. Every time this happens, I spend a good 2-3 hours of my day troubleshooting. I reference documentation, arch forums and a more recent addition - ChatGPT. 

Last week I decided that I’m done dealing with issues and need easy answers. So I decided to build a personalized AI assistant and integrate it into my terminal! It can be invoked in my terminal session, has access to my terminal history as well as OS specifications. It’s been pretty helpful to me so far so I thought I would share steps to set it up for anyone who is interested. Not that I've tried but, if you're using a different distro of Linux I think this guide could still come in handy.

### Step 1 - Set Up an OpenAI Account

Before we start coding, you’ll need access to OpenAI's API. Head over to [OpenAI's platform](https://platform.openai.com/) and create an account if you don’t already have one. Once signed up, create a new project under the **API** section, and generate an API key for your project. This key will be used in your Python program to communicate with OpenAI’s servers.

### Step 2 - Install Dependencies

For this project, I’m using Python 3.12.5, so make sure that your system has it installed. You’ll also need `pip3`, which is Python’s package installer. After setting that up, install the `openai` library:

```bash
pip3 install openai
```

I used version 1.43.0 of `openai`, which worked well for this purpose. With that in place, you’re ready to start coding!

### Step 3 - Build the General AI Client

Let’s start by building a basic AI client that interacts with OpenAI’s API. Create a Python file (`ai_client.py`) and add the following code:

```python
from openai import OpenAI

# Initialize client with API key
client = OpenAI(api_key='your-openai-api-key-here')

# Initialize messages that will be exchanged with client
messages = [ {"role": "system", "content": 
              "You are a intelligent assistant."} ]

def ask_openai(prompt):
    if prompt:
        messages.append({"role": "user", "content": prompt})
        chat = client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=messages
        )
    response = chat.choices[0].message.content
    messages.append({"role": "assistant", "content": response})
    return response

if __name__ == "__main__":
    prompt = input("Ask OpenAI: ")
    answer = ask_openai(prompt)
    print(f"OpenAI: {answer}")
```

In this code:

- I’m using the GPT-3.5 Turbo model to handle queries.
- The `ask_openai` function takes user input, sends it to the API, and returns the response.

To test this, run the Python file:

```bash
python3 ai_client.py
```

When prompted, type in any query, and the response from OpenAI will be printed to your terminal. 

### Step 4 - Make It Invokable via Command Line

To make things even smoother, I wanted to invoke my AI client directly from the terminal using a custom command, such as `archero`.

1. Create a shell script named `archero` in a directory that's in your `PATH`, such as `/usr/local/bin`

```bash
sudo nano /usr/local/bin/archero
```

1. Add the following content to the file

```bash
#!/bin/bash
python3 /path/to/your/python/ai_client.py "$@"
```

1. Make the file executable

```bash
sudo chmod +x /usr/local/bin/archero
```

1. Replace the main method with the following so it picks up all arguments after `archero`

```python
import sys

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: archero <your query>")
        sys.exit(1)

    prompt = " ".join(sys.argv[1:])
    answer = ask_openai(prompt)
    print(f"OpenAI: {answer}")
```

Now you can invoke your client by simply typing `archero` followed by a query, like this:

```bash
archero how do I update my system
```

At this point I was getting a generic response like this:

```bash
OpenAI: To update your system, I would need to know which operating system you are using.
```

### Step 5 - Integrate Terminal History

The next step was to give the AI some context from the commands I had already tried in the terminal. Since I use Fish shell, the command history is stored in `~/.local/share/fish/fish_history`. Here’s how I updated my AI client to use this history:

```python
import os

def get_terminal_history():
    history_file = os.path.expanduser("~/.local/share/fish/fish_history")
    try:
        with open(history_file, "r") as file:
            history = file.readlines()
            # Fish history format: lines starting with '- cmd: ' are actual commands
            commands = [line.strip()[6:] for line in history if line.startswith('- cmd: ')]
    except Exception as e:
        print(f"Error reading history file: {e}")
        commands = []
    return commands

```

Now, my AI client includes recent commands I’ve run in the terminal in its prompt. This way, the AI can better understand what I’ve already tried and provide more relevant suggestions.

For other popular shells you can also find the terminal history

- for Bash - `~/.bash_history`
- for zsh - `~/.zsh_history`

### Step 6 - Add System Info Using Neofetch

Finally, I wanted my AI to also know about my system’s specifications. For this, I used `neofetch`, a tool that displays your system information in the terminal. The output from `neofetch` contains ANSI escape codes, so I cleaned it up before passing it to the AI.

```python
import re
import subprocess

def get_neofetch_output():
    try:
        # Run the neofetch command and capture its output
        output = subprocess.check_output(['neofetch', '--stdout']).decode('utf-8')
        
        # Use regular expression to remove ANSI escape codes
        clean_output = re.sub(r'\x1B\[[0-?]*[ -/]*[@-~]', '', output)
    except Exception as e:
        print(f"Error running neofetch: {e}")
        clean_output = "Neofetch could not be run."
    return clean_output
```

By adding the output from `neofetch`, the AI client now understands the specifics of my operating system, hardware, and more, allowing it to provide even more tailored responses.

### Final Code

```python
from openai import OpenAI
import sys
import os
import re
import subprocess

# Initialize client with API key
client = OpenAI(api_key='your-openai-api-key-here')

# Initialize messages that will be exchanged with client
messages = [ {"role": "system", "content": 
              "You are a intelligent assistant."} ]
              
def get_neofetch_output():
    try:
        # Run the neofetch command and capture its output
        output = subprocess.check_output(['neofetch', '--stdout']).decode('utf-8')
        
        # Use regular expression to remove ANSI escape codes
        clean_output = re.sub(r'\x1B\[[0-?]*[ -/]*[@-~]', '', output)
    except Exception as e:
        print(f"Error running neofetch: {e}")
        clean_output = "Neofetch could not be run."
    return clean_output

def get_terminal_history():
    history_file = os.path.expanduser("~/.local/share/fish/fish_history")
    try:
        with open(history_file, "r") as file:
            history = file.readlines()
            # Fish history format: lines starting with '- cmd: ' are actual commands
            commands = [line.strip()[6:] for line in history if line.startswith('- cmd: ')]
    except Exception as e:
        print(f"Error reading history file: {e}")
        commands = []
    return commands

def ask_openai(prompt, terminal_history, system_info):
    # Combine terminal history, system information, and the user's prompt
    full_prompt = f"Here is my recent terminal history: {', '.join(terminal_history[-10:])}\n\n" \
                  f"My system specifications: {system_info}\n\n" \
                  f"Question: {prompt}"

    if full_prompt:
        messages.append(
            {"role": "user", "content": full_prompt},
        )
        chat = client.chat.completions.create(model="gpt-3.5-turbo", messages=messages)
    response = chat.choices[0].message.content
    messages.append({"role": "assistant", "content": response})
    return response

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: archero <your query>")
        sys.exit(1)

    prompt = " ".join(sys.argv[1:])
    terminal_history = get_terminal_history()
    system_info = get_neofetch_output()
    answer = ask_openai(prompt, terminal_history, system_info)
    print(f"OpenAI: {answer}")
```

In just a few steps, I built an AI client that integrates seamlessly with my terminal, using Fish shell history and system information to provide smarter responses. It’s been a huge time-saver, especially when troubleshooting issues on my Arch Linux system.

Now when I ask something like `archero how do I update my system`  I get a more catered response like:

```python
why don't you ask ur mum
```

Feel free to adapt the steps to your environment and enjoy the convenience of having your own AI assistant directly in your terminal! Full code is available in my dotfiles repository here: https://github.com/gucci-ninja/.dotfiles/blob/master/archero.py

Actual response (for legal purposes):

```bash
OpenAI: To update your Arch Linux system, you can use the following commands in your terminal:

1. First, update the package repositories and package list:

sudo pacman -Sy


2. Then, upgrade all installed packages to their latest versions:

sudo pacman -Su


3. Finally, you can also clean up any cached packages that are no longer required by running:

sudo pacman -Sc


Remember to carefully review the package changes before proceeding with the update. It's important to keep your system up to date to ensure security and stability.

```