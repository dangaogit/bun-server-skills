# Bun Server WebSocket

> **v3 Breaking Change**: WebSocket client type is now `IWebSocket<T>` (from `@dangao/bun-server`) instead of Bun's `ServerWebSocket<T>`. This ensures compatibility across both Bun and Node.js platforms.

## Basic Gateway

```typescript
import {
  WebSocketGateway,
  OnOpen,
  OnMessage,
  OnClose,
} from "@dangao/bun-server";
import type { IWebSocket } from "@dangao/bun-server";

@WebSocketGateway("/ws")
class ChatGateway {
  private clients = new Set<IWebSocket>();

  @OnOpen()
  handleOpen(ws: IWebSocket) {
    this.clients.add(ws);
    console.log("Client connected, total:", this.clients.size);
  }

  @OnMessage()
  handleMessage(ws: IWebSocket, message: string | Buffer) {
    const text = typeof message === "string" ? message : message.toString();
    console.log("Received:", text);

    // Broadcast to all clients
    for (const client of this.clients) {
      client.send(`Echo: ${text}`);
    }
  }

  @OnClose()
  handleClose(ws: IWebSocket) {
    this.clients.delete(ws);
    console.log("Client disconnected, total:", this.clients.size);
  }
}
```

## Register Gateway

```typescript
import { Application, WebSocketGatewayRegistry } from "@dangao/bun-server";

const app = new Application({ port: 3100 });

// Register gateway
WebSocketGatewayRegistry.getInstance().register(ChatGateway);

// Or via module
@Module({
  providers: [ChatGateway],
})
class AppModule {}

app.registerModule(AppModule);
app.listen();
```

## Connection Data

Store custom data per connection:

```typescript
import { WebSocketConnectionData } from "@dangao/bun-server";
import type { IWebSocket } from "@dangao/bun-server";

interface UserConnectionData extends WebSocketConnectionData {
  userId: string;
  username: string;
  room: string;
}

@WebSocketGateway("/ws/chat")
class ChatGateway {
  @OnOpen()
  handleOpen(ws: IWebSocket<UserConnectionData>) {
    // Set connection data
    ws.data.userId = generateId();
    ws.data.username = "Anonymous";
    ws.data.room = "lobby";
  }

  @OnMessage()
  handleMessage(ws: IWebSocket<UserConnectionData>, message: string) {
    const { userId, username, room } = ws.data;
    console.log(`[${room}] ${username}: ${message}`);
  }
}
```

## Room-based Chat

```typescript
import type { IWebSocket } from "@dangao/bun-server";

@WebSocketGateway("/ws/chat")
class RoomChatGateway {
  private rooms = new Map<string, Set<IWebSocket>>();

  @OnOpen()
  handleOpen(ws: IWebSocket) {
    this.joinRoom(ws, "lobby");
  }

  @OnMessage()
  handleMessage(ws: IWebSocket, message: string) {
    try {
      const data = JSON.parse(message);

      switch (data.type) {
        case "join":
          this.joinRoom(ws, data.room);
          break;
        case "leave":
          this.leaveRoom(ws, data.room);
          break;
        case "message":
          this.broadcast(ws.data.room, {
            type: "message",
            from: ws.data.username,
            text: data.text,
          });
          break;
      }
    } catch (e) {
      ws.send(JSON.stringify({ type: "error", message: "Invalid JSON" }));
    }
  }

  @OnClose()
  handleClose(ws: IWebSocket) {
    for (const [room, clients] of this.rooms) {
      clients.delete(ws);
    }
  }

  private joinRoom(ws: IWebSocket, room: string) {
    if (!this.rooms.has(room)) {
      this.rooms.set(room, new Set());
    }
    this.rooms.get(room)!.add(ws);
    ws.data.room = room;
  }

  private leaveRoom(ws: IWebSocket, room: string) {
    this.rooms.get(room)?.delete(ws);
  }

  private broadcast(room: string, data: any) {
    const clients = this.rooms.get(room);
    if (clients) {
      const message = JSON.stringify(data);
      for (const client of clients) {
        client.send(message);
      }
    }
  }
}
```

## With DI Services

```typescript
import type { IWebSocket } from "@dangao/bun-server";

@Injectable()
class MessageService {
  async saveMessage(userId: string, text: string) {
    // Save to database
  }
}

@WebSocketGateway("/ws")
class ChatGateway {
  constructor(private readonly messageService: MessageService) {}

  @OnMessage()
  async handleMessage(ws: IWebSocket, message: string) {
    await this.messageService.saveMessage(ws.data.userId, message);
  }
}
```

## IWebSocket Interface

`IWebSocket<T>` exposes the same core methods as Bun's `ServerWebSocket<T>` and works on both Bun and Node.js:

```typescript
interface IWebSocket<T = unknown> {
  send(data: string | Buffer): void;
  close(code?: number, reason?: string): void;
  data: T;
  readyState: number;
  remoteAddress: string;
}
```

## Client Connection Example

```javascript
// Browser client
const ws = new WebSocket("ws://localhost:3100/ws/chat");

ws.onopen = () => {
  console.log("Connected");
  ws.send(JSON.stringify({ type: "join", room: "general" }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log("Received:", data);
};

ws.onclose = () => {
  console.log("Disconnected");
};

// Send message
ws.send(JSON.stringify({ type: "message", text: "Hello!" }));
```

## Related Resources

- [WebSocket Chat Example](https://github.com/dangaogit/bun-server/blob/main/examples/03-advanced/websocket-chat-app.ts)
- [Platform Adapter Guide](platform.md)
