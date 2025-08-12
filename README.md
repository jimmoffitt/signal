## Automating Signal inbox monitoring and sending messages. 

Python packages? 
* https://github.com/AsamK/signal-cli
* pip install pysignal-cli

Using the Python `subprocess` package to call the signal-cli package. 

Sending a message:

```python
        command = [
            "signal-cli",
            "-u", sender,
            "send",
            "-m", message,
            recipient
        ]
        
        # Run the command and capture its output
        result = subprocess.run(command, check=True, capture_output=True, text=True)
```

Monitoring inbox:

```python
    # The command to start listening for messages
    command = [
        "signal-cli",
        "-u", user_number,
        "receive",
        "--json" # We'll use JSON output for easier parsing
    ]
    
    # Run the command. The `stdout=subprocess.PIPE` captures the output as it happens, allowing us to process messages in real-time.
    process = subprocess.Popen(command, stdout=subprocess.PIPE, text=True, bufsize=1)
```

If you are comfortable using the Subprocess package, then we are on our way...

## Install 

Install and register your Signal number (needs a phone number that can receive SMS or calls):
```bash
brew install signal-cli
signal-cli -u your_phone_number register
signal-cli -u your_phone_number verify YOUR_VERIFICATION_CODE
```

Link devices if you want to use it with an existing account instead of a new registration:
```bash
signal-cli link
```


## Python Script to Send a Message

This script uses pysignal-cli to send a message to a specific recipient. You'll need to replace the placeholders with your registered phone number and the recipient's phone number.


```python
# send_message.py
# This script sends a text message via signal-cli.

import subprocess

# Your registered Signal phone number (e.g., "+15551234567")
# This is the number that the message will be sent FROM.
SENDER_NUMBER = "+15551234567"

# The recipient's Signal phone number or group ID
# (e.g., "+15559876543" or "some_group_id")
RECIPIENT = "+15559876543"

# The message to be sent
MESSAGE = "Hello from an automated Python script!"

def send_signal_message(sender, recipient, message):
    """
    Sends a Signal message using the signal-cli command-line tool.
    
    Args:
        sender (str): The phone number registered with signal-cli.
        recipient (str): The recipient's phone number or group ID.
        message (str): The message text to send.
    """
    try:
        # Construct the command to be executed in the terminal
        command = [
            "signal-cli",
            "-u", sender,
            "send",
            "-m", message,
            recipient
        ]
        
        # Run the command and capture its output
        result = subprocess.run(command, check=True, capture_output=True, text=True)
        
        # Check if the command was successful
        print(f"Message sent successfully! Output:\n{result.stdout}")
        
    except subprocess.CalledProcessError as e:
        # Handle errors if the command fails
        print(f"Error sending message: {e}")
        print(f"Stdout: {e.stdout}")
        print(f"Stderr: {e.stderr}")
    except FileNotFoundError:
        # Handle the case where signal-cli is not in the system's PATH
        print("Error: signal-cli command not found. Make sure it's installed and in your PATH.")

if __name__ == "__main__":
    send_signal_message(SENDER_NUMBER, RECIPIENT, MESSAGE)
```

## Python Script to Display Inbox Contents
This script uses pysignal-cli to start a listening process. It will then display any incoming messages in real-time. This is often the most useful for building a bot or an automated service.

```python
# receive_messages.py
# This script listens for and displays incoming Signal messages in real-time.

import json
import subprocess

# Your registered Signal phone number
# This is the number that will be listening for messages.
MY_PHONE_NUMBER = "+15551234567"

def listen_for_messages(user_number):
    """
    Starts a persistent listen process with signal-cli and prints messages.
    
    This function blocks and listens for new messages. It requires
    signal-cli to be running in the background.
    """
    print(f"Listening for Signal messages on {user_number}...")
    
    # The command to start listening for messages
    command = [
        "signal-cli",
        "-u", user_number,
        "receive",
        "--json" # We'll use JSON output for easier parsing
    ]
    
    # Run the command. The `stdout=subprocess.PIPE` captures the output
    # as it happens, allowing us to process messages in real-time.
    process = subprocess.Popen(command, stdout=subprocess.PIPE, text=True, bufsize=1)

    try:
        # Iterate over the output line by line as it is received
        for line in iter(process.stdout.readline, ''):
            if line:
                try:
                    # Parse the JSON output from signal-cli
                    message_data = json.loads(line.strip())
                    
                    # Extract and print the relevant message details
                    envelope = message_data.get("envelope", {})
                    data_message = envelope.get("dataMessage", {})
                    
                    source = envelope.get("source")
                    timestamp = envelope.get("timestamp")
                    message_body = data_message.get("message")
                    
                    if message_body:
                        print("-" * 50)
                        print(f"Timestamp: {timestamp}")
                        print(f"From: {source}")
                        print(f"Message: {message_body}")
                        print("-" * 50)
                        
                except json.JSONDecodeError:
                    print(f"Could not parse JSON: {line.strip()}")
            
    except KeyboardInterrupt:
        print("\nStopping listener...")
    finally:
        # Terminate the subprocess cleanly when the script exits
        process.terminate()
        process.wait()

if __name__ == "__main__":
    listen_for_messages(MY_PHONE_NUMBER)
```
## Signal Groups

### Sending to Groups

To send a message to a group, you replace the recipient's phone number with the group ID. You can find the group ID by using signal-cli to list your groups or by capturing an incoming message from the group. The command to send a message would look like this:

```bash
signal-cli -u SENDER_NUMBER send -m "Your message here" GROUP_ID
```
The GROUP_ID is a long, alphanumeric string that uniquely identifies the group.

### Group Management Commands

signal-cli also includes other commands for managing groups, such as:

* signal-cli -u your_number updateGroup --name "New Name": To rename a group.
* signal-cli -u your_number addMember --uri "signal-cli-uri-link": To add a new member.
* signal-cli -u your_number leaveGroup: To leave a group.


## The AI debate about real-time listening

ChatGPT makes these claims:

* Receiving messages in real-time requires running signal-cli in daemon mode or polling.

```python
from pysignalclilib import SignalCli

USERNAME = "+1234567890"
signal = SignalCli(username=USERNAME)

# Read messages
messages = signal.receive()
for msg in messages:
    print(f"From: {msg['envelope']['source']}")
    print(f"Message: {msg['envelope']['dataMessage']['message']}")
```
