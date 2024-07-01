Group-Chat_WebSocket

# Difference Between Webhook and WebSocket

## Webhooks

1. **Communication Model**: 
   - Webhooks use a one-way communication model where the server sends data to the client (usually another server) when a specific event occurs.

2. **Trigger-Based**: 
   - They are event-driven, meaning they are triggered by specific events or actions on the server-side.

3. **HTTP Protocol**: 
   - Webhooks rely on the HTTP protocol to send data. When an event occurs, an HTTP POST request is made to a predefined URL.

4. **Stateless**: 
   - Each webhook request is independent and does not require a persistent connection between the client and the server.

5. **Use Case**: 
   - Commonly used for notifications and real-time updates, such as payment confirmations, GitHub commit notifications, or form submissions.

## WebSockets

1. **Communication Model**: 
   - WebSockets use a two-way communication model that allows for continuous, real-time interaction between the client and the server.

2. **Persistent Connection**: 
   - They establish a long-lived connection between the client and server, allowing for continuous data exchange without the need to repeatedly establish new connections.

3. **Protocol**: 
   - WebSockets have their own protocol, ws:// (non-secure) or wss:// (secure), and start with an HTTP handshake before switching to the WebSocket protocol.

4. **Stateful**: 
   - The connection remains open and stateful, facilitating ongoing communication without the overhead of re-establishing connections.

5. **Use Case**: 
   - Ideal for real-time applications such as live chat, gaming, live updates, and other scenarios requiring continuous data exchange.

## Summary

- **Webhooks**: 
  - Event-driven, one-way communication, uses HTTP, stateless, suitable for notifications and event responses.

- **WebSockets**: 
  - Continuous, two-way communication, uses WebSocket protocol, stateful, suitable for real-time, interactive applications.
  
# Program

## WebSocket Chat Application with FastAPI
This document explains the implementation of a simple WebSocket chat application using FastAPI. The application allows multiple clients to connect and broadcast messages to all connected clients in real-time.

### Overview
WebSockets provide a full-duplex communication channel over a single TCP connection, enabling real-time interaction between clients and servers. This example demonstrates how to use FastAPI to manage WebSocket connections, handle messages, and broadcast them to all connected clients.

### Code Explanation
Importing Required Modules
    from fastapi import FastAPI, WebSocket, WebSocketDisconnect
    from typing import List
- FastAPI: The main class for creating the web application.
- WebSocket: Used for defining WebSocket endpoints.
- WebSocketDisconnect: Exception raised when a WebSocket connection is closed.
- List: A type hint for defining a list of WebSocket connections.

### Initializing the FastAPI Application

    app = FastAPI()

Creates an instance of the FastAPI application.

### ConnectionManager Class

    class ConnectionManager:
        def __init__(self):
            self.active_connections: List[WebSocket] = []

        async def connect(self, websocket: WebSocket):
            await websocket.accept()
            self.active_connections.append(websocket)

        def disconnect(self, websocket: WebSocket):
            self.active_connections.remove(websocket)

        async def broadcast(self, message: str):
            for connection in self.active_connections:
                await connection.send_text(message)
- \__init__(self): Initializes the active_connections list to store active WebSocket connections.
- connect(self, websocket: WebSocket): Accepts a new WebSocket connection and adds it to the active_connections list.
- disconnect(self, websocket: WebSocket): Removes a WebSocket connection from the active_connections list.
- broadcast(self, message: str): Sends a message to all active WebSocket connections.

### Creating an Instance of ConnectionManager

    manager = ConnectionManager()
Creates an instance of the ConnectionManager class to manage WebSocket connections.

### WebSocket Endpoint

    @app.websocket("/ws/{client_id}")
    async def websocket_endpoint(websocket: WebSocket, client_id: int):
        await manager.connect(websocket)
        try:
            while True:
                data = await websocket.receive_text()
                await manager.broadcast(f"Client #{client_id} says: {data}")
        except WebSocketDisconnect:
            manager.disconnect(websocket)
            await manager.broadcast(f"Client #{client_id} left the chat")
Defines a WebSocket endpoint at /ws/{client_id}.

- websocket: WebSocket: The WebSocket connection object.
- client_id: int: A unique identifier for the client.

### Workflow:

1. Connect: When a client connects to the WebSocket endpoint, manager.connect(websocket) is called to accept the connection and add it to the active connections list.
2. Receive and Broadcast Messages:
    - A while True loop is used to continuously listen for incoming messages from the client.
    - When a message is received (data = await websocket.receive_text()), it is broadcast to all connected clients using manager.broadcast(f"Client #{client_id} says: {data}").

3. Disconnect:
- If the client disconnects (WebSocketDisconnect), the connection is removed from the active connections list using manager.disconnect(websocket).
- A disconnect message is broadcast to inform other clients that the client has left the chat.

### Running the Application
To run the application, save the code to a file (e.g., app.py) and run it using Uvicorn:

    uvicorn app:app --reload
Open multiple browser tabs or WebSocket clients and connect to ws://localhost:8000/ws/{client_id} with different client_id values to test the chat functionality.

### Conclusion
This example demonstrates the basics of using WebSockets with FastAPI to create a real-time chat application. The ConnectionManager class manages WebSocket connections and broadcasting messages, while the WebSocket endpoint handles client interactions. This pattern can be extended to create more complex real-time applications.