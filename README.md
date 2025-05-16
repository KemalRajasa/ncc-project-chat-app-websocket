# ncc-project-chat-app-websocket

## Setup dan Instalasi Deno di Powershell Local
  Instalasi Deno
  ```
  irm https://deno.land/install.ps1 | iex
  ```
  ![image](https://github.com/user-attachments/assets/a0c00eb6-d294-42db-8c8f-c68360441be4)

  ```
  deno init chat-app
  cd chat-app
  ```
  ![image](https://github.com/user-attachments/assets/ef902932-58e0-4b4e-976b-7cba55d0dd46)

  ```
  deno add jsr:@oak/oak
  ```
  ![image](https://github.com/user-attachments/assets/cbc1d548-3cd9-424d-8526-9f49c1f845b0)

  Edit FIle ```main.ts``` menjadi:
  ```
  import { Application, Context, Router } from "@oak/oak";
  import ChatServer from "./ChatServer.ts";
  
  const app = new Application();
  const port = 8080;
  const router = new Router();
  const server = new ChatServer();
  
  router.get("/start_web_socket", (ctx: Context) => server.handleConnection(ctx));
  
  app.use(router.routes());
  app.use(router.allowedMethods());
  app.use(async (context) => {
    await context.send({
      root: Deno.cwd(),
      index: "public/index.html",
    });
  });
  
  console.log("Listening at http://localhost:" + port);
  await app.listen({ hostname: "0.0.0.0", port });
  ```

  Lalu buat file bernama ```ChatServer.ts```, dan isi dengan:
  ```
  import { Context } from "@oak/oak";

  type WebSocketWithUsername = WebSocket & { username: string };
  type AppEvent = { event: string; [key: string]: any };
  
  export default class ChatServer {
    private connectedClients = new Map<string, WebSocketWithUsername>();
  
    public async handleConnection(ctx: Context) {
      const socket = await ctx.upgrade() as WebSocketWithUsername;
      const username = ctx.request.url.searchParams.get("username");
  
      if (this.connectedClients.has(username)) {
        socket.close(1008, `Username ${username} is already taken`);
        return;
      }
  
      socket.username = username;
      socket.onopen = this.broadcastUsernames.bind(this);
      socket.onclose = () => {
        this.clientDisconnected(socket.username);
      };
      socket.onmessage = (m) => {
        this.send(socket.username, m);
      };
      this.connectedClients.set(username, socket);
  
      console.log(`New client connected: ${username}`);
    }
  
    private send(username: string, message: any) {
      const data = JSON.parse(message.data);
      if (data.event !== "send-message") {
        return;
      }
  
      this.broadcast({
        event: "send-message",
        username: username,
        message: data.message,
      });
    }
  
    private clientDisconnected(username: string) {
      this.connectedClients.delete(username);
      this.broadcastUsernames();
  
      console.log(`Client ${username} disconnected`);
    }
  
    private broadcastUsernames() {
      const usernames = [...this.connectedClients.keys()];
      this.broadcast({ event: "update-users", usernames });
  
      console.log("Sent username list:", JSON.stringify(usernames));
    }
  
    private broadcast(message: AppEvent) {
      const messageString = JSON.stringify(message);
      for (const client of this.connectedClients.values()) {
        client.send(messageString);
      }
    }
  }
  ```

  Jalankan dengan:
  ```
  deno run main.ts
  ```
  ![image](https://github.com/user-attachments/assets/6fe272ea-0389-4e38-b8b6-7fc1b4394fc2)

  Sejauh ini seharusnya bisa terhubung dengan menggunakan localhost
  ![image](https://github.com/user-attachments/assets/d411b03c-0d2e-428b-ad84-96c67f162c6b)

  ### Hasil dari step sejauh ini:

  connect via localhost


## Virtual Machine Azure
  Cara deploy VM sama seperti dengan project sebelumnya yaitu [Deploy CTF menggunakan CTFd](https://github.com/KemalRajasa/Hosting-CTF-using-CTFd-and-Microsoft-Azure/blob/main/README.md)

## Deploy Chat-App menggunakan VM Azure

### Instalasi Deno
  ```
  apt-get update
  apt-get install unzip -y
  curl -fsSL https://deno.land/install.sh | sh
  deno init chat-app
  cd chat-app
  ```
### Instalasi Docker dan Docker-compose
  ```
  apt-get install docker
  apt-get install docker-compose
  ```
### Konfigurasi file Dockerfile
>pastikan file Dockerfile ada di direktori yang sama dengan main.ts dan ChatServer.ts (direktori chat-app)

  [Dockerfile](https://github.com/KemalRajasa/ncc-project-chat-app-websocket/blob/main/Dockerfile)
  
### Konfigurasi docker-compose.yml

### Konfigurasi nginx.conf

## Instalasi Resource di VM
