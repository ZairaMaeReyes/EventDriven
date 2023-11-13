# Event Driven Programming

**Activity:** Real-time chat-app using Python sockets and Tkinter
***
**_Coding server.py_**

This server allows multiple clients to connect and exchange messages. Here's a brief overview of my code: 

1. Server Initialization (**'__ init __'**):

* The server initializes a socket (server_socket) for communication.
* The create_listening_server method is responsible for setting up the server socket, binding it to a specific IP and port, and starting to listen for incoming connections.

```python
def __init__(self):
    self.server_socket = None
    self.create_listening_server()
```
  
2. Listening for Connections (**'create_listening_server'**):

* The server socket is configured to allow immediate restart and to listen for incoming connections.
* It binds to the local IP address '127.0.0.1' and port 10319.
* It then enters a loop to continuously listen for incoming connections.
* When a connection is established, it calls receive_messages_in_a_new_thread to handle the new client in a separate thread.

```python
def create_listening_server(self):
    self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    local_ip = '127.0.0.1'
    local_port = 10319
    self.server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    self.server_socket.bind((local_ip, local_port))
    print("Listening for incoming messages..")
    self.server_socket.listen(5)
    self.receive_messages_in_a_new_thread()
```
3. Handling Clients (**'receive_messages_in_a_new_thread'**):

* This method runs in a loop, accepting new client connections.
* For each new client, it creates a new thread (t) to handle incoming messages from that client and starts the thread.
* The **'receive_messages'** method is called in the new thread to handle communication with the client.

```python
def receive_messages_in_a_new_thread(self):
    while True:
        client = so, (ip, port) = self.server_socket.accept()
        self.add_to_clients_list(client)
        print('Connected to ', ip, ':', str(port))
        t = threading.Thread(target=self.receive_messages, args=(so,))
        t.start()
```

4. Receiving Messages (**'receive_messages'**):

* This method runs in a loop, continuously receiving messages from a specific client (so).
* The received message is decoded from bytes to a UTF-8 string.
* The last received message is stored in the **'last_received_message'** attribute.
* The message is then broadcasted to all connected clients (except the sender).

```python
def receive_messages(self, so):
    while True:
        incoming_buffer = so.recv(256)
        if not incoming_buffer:
            break
        self.last_received_message = incoming_buffer.decode('utf-8')
        self.broadcast_to_all_clients(so)
    so.close()
```

5. Broadcasting Messages (**'broadcast_to_all_clients'**):

* This method sends the last received message to all clients except the sender.

```python
def broadcast_to_all_clients(self, senders_socket):
    for client in self.clients_list:
        socket, (ip, port) = client
        if socket is not senders_socket:
            socket.sendall(self.last_received_message.encode('utf-8'))
```

6. Adding Clients (**'add_to_clients_list'**):

* This method adds a new client to the **'clients_list'** if it is not already present.

```python
def add_to_clients_list(self, client):
    if client not in self.clients_list:
        self.clients_list.append(client)
```

7. Main Block (**'__ main __'**):

* The server is instantiated and starts running in the main block.

```python
if __name__ == "__main__":
    ChatServer()
```
***
**_Coding client.py_**

This code implements a simple chat application using Python's tkinter for the graphical user interface (GUI) and sockets for network communication. The chat application allows users to join a chat room, send messages, and receive messages from others in real-time. Here's the brief overview of my code:

1. Socket Initialization:

* The **'initialize_socket'** method sets up a TCP socket and connects to a remote server (IP address '127.0.0.1' and port 10319).

```python
    def initialize_socket(self):
        # Initialize the client socket and connect to the remote server.
        self.client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        remote_ip = '127.0.0.1'
        remote_port = 10319
        self.client_socket.connect((remote_ip, remote_port))
```

2. GUI Initialization:

* The **'initialize_gui'** method sets up the main window, title, and calls other methods to display the chat box, name section, and entry box.

```python
    def initialize_gui(self):
        # Set up the GUI window, including chat box, name section, and entry box.
        self.root.title("Chat Room")
        self.root.resizable(0, 0)
        self.display_chat_box()
        self.display_name_section()
        self.display_chat_entry_box()
```

3. Message Receiving Thread:

* The **'listen_for_incoming_messages_in_a_thread'** method creates a separate thread to continuously listen for incoming messages from the server using the **'receive_message_from_server'** method.

```python
    def listen_for_incoming_messages_in_a_thread(self):
        # Create a thread to continuously receive messages from the server.
        thread = threading.Thread(target=self.receive_message_from_server, args=(self.client_socket,))
        thread.start()
```

4. Message Receiving:

* The **'receive_message_from_server'** method runs in a separate thread and continuously receives messages from the server. It decodes the received bytes and updates the chat transcript area accordingly.

```python
    def receive_message_from_server(self, so):
        # Receive messages from the server and update the chat transcript area accordingly.
        while True:
            buffer = so.recv(256)
            if not buffer:
                break
            message = buffer.decode('utf-8')

            if "joined" in message:
                user = message.split(":")[1]
                message = user + " has joined"
                self.chat_transcript_area.insert('end', message + '\n')
                self.chat_transcript_area.yview(END)
            else:
                self.chat_transcript_area.insert('end', message + '\n')
                self.chat_transcript_area.yview(END)

        so.close()
```

5. GUI Components:

* The **'display_name_section'**, **'display_chat_box'**, and **'display_chat_entry_box'** methods create and display various components of the GUI, such as the name entry, chat box, and message entry.

```python
    def display_name_section(self):
        # Set up the name section with an entry for the user's name and a button to join the chat.
        frame = Frame()
        Label(frame, text='Enter your name:', font=("Helvetica", 16)).pack(side='left', padx=10)
        self.name_widget = Entry(frame, width=50, borderwidth=2)
        self.name_widget.pack(side='left', anchor='e')
        self.join_button = Button(frame, text="Join", width=10, command=self.on_join).pack(side='left')
        frame.pack(side='top', anchor='nw')
```

```python
    def display_chat_box(self):
        # Set up the chat box with a scrollable area for displaying messages.
        frame = Frame()
        Label(frame, text='Chat Box:', font=("Serif", 12)).pack(side='top', anchor='w')
        self.chat_transcript_area = Text(frame, width=60, height=10, font=("Serif", 12))
        scrollbar = Scrollbar(frame, command=self.chat_transcript_area.yview, orient=VERTICAL)
        self.chat_transcript_area.config(yscrollcommand=scrollbar.set)
        self.chat_transcript_area.bind('<KeyPress>', lambda e: 'break')
        self.chat_transcript_area.pack(side='left', padx=10)
        scrollbar.pack(side='right', fill='y')
        frame.pack(side='top')
```

```python
    def display_chat_entry_box(self):
        # Set up the entry box for users to type and send messages.
        frame = Frame()
        Label(frame, text='Enter message:', font=("Serif", 12)).pack(side='top', anchor='w')
        self.enter_text_widget = Text(frame, width=60, height=3, font=("Serif", 12))
        self.enter_text_widget.pack(side='left', pady=15)
        self.enter_text_widget.bind('<Return>', self.on_enter_key_pressed)
        frame.pack(side='top')
```

6. Event Handling:

* The **'on_join'** method is called when the "Join" button is pressed. It sends a message to the server indicating that a user has joined.
* The **'on_enter_key_pressed'** method is called when the "Enter" key is pressed in the message entry box. It sends the entered message to the server.
* The **'clear_text'** this method clears the text entry box.

```python
    def on_join(self):
        # Handle the user joining the chat.
        if len(self.name_widget.get()) == 0:
            messagebox.showerror("Enter your name", "Enter your name to send a message")
            return
        self.name_widget.config(state='disabled')
        self.client_socket.send(("joined:" + self.name_widget.get()).encode('utf-8'))
```

```python
    def on_enter_key_pressed(self, event):
        # Handle the user pressing the Enter key.
        if len(self.name_widget.get()) == 0:
            messagebox.showerror("Enter your name", "Enter your name to send a message")
            return
        self.send_chat()
        self.clear_text()
```

```python
    def clear_text(self):
        # Clear the text entry box.
        self.enter_text_widget.delete(1.0, 'end')
```

7. Sending Messages:

* The **'send_chat'** method is responsible for sending messages to the server. It gets the sender's name and message from the GUI components, updates the chat transcript, and sends the message to the server.

```python
    def send_chat(self):
        # Send the user's chat message to the server.
        senders_name = self.name_widget.get().strip() + ": "
        data = self.enter_text_widget.get(1.0, 'end').strip()
        message = (senders_name + data).encode('utf-8')
        self.chat_transcript_area.insert('end', message.decode('utf-8') + '\n')
        self.chat_transcript_area.yview(END)
        self.client_socket.send(message)
        self.enter_text_widget.delete(1.0, 'end')
        return 'break'
```

8. Closing the Window:

* The **'on_close_window'** method is called when the user attempts to close the window. It prompts the user with a confirmation dialog and closes the application if the user chooses to quit.

```python
    def on_close_window(self):
        # Handle the window close event.
        if messagebox.askokcancel("Quit", "Do you want to quit?"):
            self.root.destroy()
            self.client_socket.close()
            exit(0)
```

9. Main Execution:

* The main part of the code creates an instance of the Tkinter **'Tk'** class, initializes the GUI, sets up a protocol to handle window closure, and starts the Tkinter main loop.

```python
# Main function to start the GUI.
if __name__ == '__main__':
    root = Tk()
    gui = GUI(root)
    root.protocol("WM_DELETE_WINDOW", gui.on_close_window)
    root.mainloop()
```
