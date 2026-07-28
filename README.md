# Internet Relay Chat Application

A lightweight, console-based **Internet Relay Chat (IRC) application** developed in Python as a for-credit academic project under **CS594 - Internetworking Protocols** at **Portland State University**.

The project implements a basic client-server chat system using **TCP socket programming** and **I/O multiplexing with Python's `select()` module**. It supports multiple concurrent clients, public chat rooms, room switching, private direct messaging, user management, and command-based interaction.

---

## Overview

The application follows a client-server architecture in which multiple chat clients connect to a central server over TCP.

The server is responsible for:

* Accepting incoming client connections
* Managing connected users
* Maintaining chat rooms
* Routing messages between users
* Handling room membership
* Supporting private conversations
* Processing user commands

The client provides a simple command-line interface through which users can interact with the chat server.

The project uses Python's `select()` mechanism to monitor multiple sockets and standard input, allowing the server and clients to handle communication without creating a dedicated thread for every connection.

---

## Features

### Multi-Client Chat

Multiple clients can connect to the same server and communicate through shared chat rooms.

The server maintains a list of active connections and uses `select()` to monitor sockets for incoming activity.

### User Registration

When a client connects, the server welcomes the user and prompts them to enter a username.

Usernames are maintained by the server and used to identify participants in rooms and direct messages.

### Chat Rooms

Users can:

* Create a new room by joining a previously nonexistent room
* Join an existing room
* View available rooms
* Leave a room
* Switch between rooms they have joined

Each room maintains a list of participating users.

### Room-Based Messaging

Messages sent by a user within a room are broadcast to other members of that room.

Messages include the active room name and the sender's username.

### Private Direct Messaging

Users can initiate a private conversation with another connected user using the direct-message command.

The application creates a dedicated chat context containing the two participants.

### Room Listing

Users can request a list of available public rooms.

The room listing displays:

* Room name
* Number of users
* Usernames of current participants

Direct-message rooms are excluded from the public room listing.

### Command Guide

The application provides an interactive command guide describing the supported commands and their syntax.

### Input Validation and Error Handling

The application defines custom exceptions for various invalid operations, including:

* Invalid commands
* Invalid users
* Invalid room switching
* Attempting to leave a room the user has not joined
* Attempting to send messages without being in a room

The server provides feedback to clients when invalid commands or operations are detected.

### Graceful Exit

Users can exit the application using the `$exit` command.

The server removes the user from the active room and connection management structures before closing the session.

---

## Supported Commands

| Command               | Description                                                     |
| --------------------- | --------------------------------------------------------------- |
| `$rooms`              | Displays the available public chat rooms and their participants |
| `$join <room_name>`   | Joins an existing room or creates a new room                    |
| `$dm <user_name>`     | Starts a private conversation with another connected user       |
| `$guide`              | Displays the command guide                                      |
| `$switch <room_name>` | Switches to a room the user has already joined                  |
| `$leave <room_name>`  | Leaves a room                                                   |
| `$exit`               | Exits the chat application                                      |

Once a user has joined a room, regular text input is treated as a chat message and broadcast to the other members of the active room.

---

## Architecture

The application is divided into three main Python modules.

```text
                    ┌──────────────────────┐
                    │       Client 1       │
                    │      client.py       │
                    └──────────┬───────────┘
                               │
                               │ TCP
                               │
                    ┌──────────▼───────────┐
                    │                      │
                    │     Chat Server      │
                    │      server.py       │
                    │                      │
                    │  select() I/O loop   │
                    │                      │
                    └──────────┬───────────┘
                               │
                       Application Logic
                               │
                    ┌──────────▼───────────┐
                    │      chat_mid.py     │
                    │                      │
                    │  MainWindow          │
                    │  Room                │
                    │  ChatUser            │
                    │  Command Handling    │
                    └──────────┬───────────┘
                               │
                               │ TCP
                ┌──────────────┴──────────────┐
                │                             │
       ┌────────▼────────┐           ┌────────▼────────┐
       │    Client 2     │           │    Client N     │
       │    client.py    │           │    client.py    │
       └─────────────────┘           └─────────────────┘
```

---

## Components

### `server.py`

The server is responsible for managing the main network event loop.

Key responsibilities include:

* Creating the listening TCP socket
* Binding to the server address
* Accepting new client connections
* Creating `ChatUser` instances
* Maintaining the active connection list
* Monitoring sockets using `select.select()`
* Receiving messages from clients
* Passing messages to the command controller
* Removing disconnected clients

The server listens on:

```text
127.0.0.1:5005
```

The server uses a non-blocking listening socket and monitors the listening socket and connected clients through `select()`.

---

### `client.py`

The client provides the command-line interface used by chat participants.

It is responsible for:

* Connecting to the chat server
* Reading user input from standard input
* Receiving messages from the server
* Displaying server responses
* Sending commands and chat messages
* Detecting server disconnection
* Handling the server's exit signal

The client uses `select()` to monitor both:

* Standard input (`sys.stdin`)
* The TCP connection to the server

This allows the client to receive incoming chat messages while simultaneously waiting for user input.

---

### `chat_mid.py`

This module contains the core application logic shared by the client and server.

It defines:

* Socket creation utilities
* Custom exception classes
* `MainWindow`
* `Room`
* `ChatUser`
* Command processing logic
* User-room relationships
* Message broadcasting

#### `MainWindow`

Acts as the central application state manager on the server side.

It maintains:

```text
rooms
    └── Maps room names to Room objects

link_user_room
    └── Tracks user-to-room membership

link_user
    └── Maps usernames to ChatUser objects
```

It is responsible for:

* Processing user commands
* Creating and managing rooms
* Tracking users
* Managing room membership
* Listing available rooms
* Routing messages
* Removing users

#### `Room`

Represents a chat room and maintains a list of users currently associated with it.

Responsibilities include:

* Welcoming users
* Broadcasting messages
* Broadcasting room departure notifications
* Removing users from the room

#### `ChatUser`

Represents a connected client.

Each `ChatUser` contains:

* Client socket
* Username
* Current room

The class also exposes the underlying socket file descriptor through `fileno()`, allowing the object to participate in socket monitoring.

---

## Networking Concepts Demonstrated

This project was developed for an **Internetworking Protocols** course and demonstrates several practical networking concepts.

### TCP Socket Programming

The application uses:

```python
socket.AF_INET
socket.SOCK_STREAM
```

to establish TCP-based communication between clients and the server.

The TCP connection provides reliable, connection-oriented communication between the chat clients and the server.

### Client-Server Architecture

All chat clients connect to a centralized server.

The server acts as the communication hub and manages:

```text
Client → Server → Other Client(s)
```

This architecture allows clients to communicate without establishing direct connections with each other.

### I/O Multiplexing

The project uses Python's `select()` mechanism to monitor multiple input sources.

On the server, the mechanism monitors:

* The listening socket
* Connected client sockets

On the client, it monitors:

* Standard input
* The server socket

This enables the application to handle multiple communication sources without relying on a separate thread for every connection.

### Non-Blocking Sockets

The server's listening socket is configured as non-blocking.

Connected client sockets are also configured for non-blocking operation through the `ChatUser` abstraction.

This design works together with `select()` to prevent the application from blocking while waiting for individual sockets.

### Message Broadcasting

The `Room` abstraction implements room-based message broadcasting.

When a user sends a message, the server forwards it to the other users associated with the active room.

### Connection Management

The server maintains a collection of active client connections and dynamically adds and removes clients as they connect and disconnect.

---

## Application Flow

```text
                           Start Server
                                │
                                ▼
                     Create TCP Listening Socket
                                │
                                ▼
                     Bind to 127.0.0.1:5005
                                │
                                ▼
                     Start Listening for Clients
                                │
                                ▼
                         select() Event Loop
                                │
                  ┌─────────────┴─────────────┐
                  │                           │
           New Connection              Existing Client
                  │                           │
                  ▼                           ▼
            accept()                  Receive Message
                  │                           │
                  ▼                           ▼
          Create ChatUser              Process Command
                  │                           │
                  ▼                    ┌──────┴──────┐
          Add to Connection            │             │
              List                 Command        Chat Message
                                      │             │
                                      ▼             ▼
                                Execute Action   Broadcast
                                      │             │
                                      └──────┬──────┘
                                             │
                                             ▼
                                      Continue Loop
```

---

## Project Structure

```text
IRC-main/
├── RFC.pdf
├── chat_mid.py
├── client.py
├── server.py
└── README.md
```

### `RFC.pdf`

Reference material related to the Internet Relay Chat protocol used as part of the academic project.

### `chat_mid.py`

Core application logic, room management, user management, command processing, and message broadcasting.

### `server.py`

TCP chat server and `select()`-based multi-client event loop.

### `client.py`

Command-line chat client and client-side socket/input event handling.

---

## Requirements

The project requires:

* Python 3
* Standard Python libraries:

  * `socket`
  * `select`
  * `sys`
  * `re`

No external Python packages are required by the source code.

---

## Running the Application

The application is designed to run locally using:

```text
127.0.0.1:5005
```

### 1. Start the Server

Open a terminal and run:

```bash
python server.py
```

The server will start listening for incoming connections.

### 2. Start a Client

Open another terminal and run:

```bash
python client.py
```

The client will connect to the server and prompt you to enter a username.

### 3. Start Additional Clients

To simulate multiple users, open additional terminal windows and run:

```bash
python client.py
```

Each client can connect to the same running server.

### 4. Join a Room

After connecting, use:

```text
$join <room_name>
```

For example:

```text
$join general
```

Once users are in the same room, regular text messages can be exchanged.

---

## Example Session

```text
Connected to the chat server successfully

Welcome to the Internet Relay Chat Application!

Enter your name:

User Command Guide:

$rooms              : Gives a list of rooms you can join
$join <room_name>   : To join an existing room or start one
$dm <user_name>     : To have a private conversation with another user
$guide              : Displays user guide
$switch <room_name> : To switch between rooms
$leave <room_name>  : To leave a room
$exit               : To exit the application

If you have already joined a room, you can go ahead with your messages

$join general

*****general*****
alice: Hello everyone!
```

---

## Error Handling

The application defines custom exception types to represent invalid application operations:

* `WrongRoomLeaveError`
* `InvalidSwitchError`
* `InvalidCommandError`
* `InvalidUserError`
* `NotPresentInRoomError`

These are used to provide user feedback for scenarios such as:

* Leaving a room the user does not belong to
* Switching to a room the user has not joined
* Entering an invalid command
* Attempting to message a nonexistent user
* Sending a chat message without joining a room

---

## Limitations

This project was developed as an academic networking project and has several limitations:

* The server is configured to run on `127.0.0.1`, making the default setup suitable for local testing.
* Communication is not encrypted.
* There is no authentication mechanism.
* User data is maintained in memory and is not persisted.
* Chat history is not persisted.
* There is no database layer.
* There is no formal user account management system.
* The application does not implement the complete standardized IRC protocol.
* The implementation is a simplified IRC-style chat application intended to demonstrate networking concepts.
* The server uses a single event-driven loop rather than a distributed architecture.

---

## Possible Future Improvements

Potential extensions to the project include:

* Allow configuration of the server host and port
* Support remote clients over a network
* Add TLS encryption
* Implement user authentication
* Add persistent user accounts
* Store chat history
* Add timestamps to messages
* Add user presence and online status
* Improve room membership management
* Add room administrators and moderation capabilities
* Support standardized IRC commands and protocol messages
* Add private-message history
* Improve connection and exception handling
* Add automated tests
* Build a graphical or web-based client
* Containerize the server for easier deployment

---

## Academic Context

**Project:** Internet Relay Chat Application
**Course:** CS594 - Internetworking Protocols
**Institution:** Portland State University
**Language:** Python
**Architecture:** Client-Server
**Transport Protocol:** TCP
**I/O Model:** `select()`-based I/O multiplexing
**Interface:** Command-line

The project was developed as an academic exercise to explore practical networking concepts including **TCP socket programming, client-server communication, connection management, non-blocking sockets, I/O multiplexing, and message routing**.
