# Bun Server AI Modules (v2.0+)

## Overview

Starting from `@dangao/bun-server` v2.0.0, the framework provides an official AI module stack for LLM applications.

All providers use Bun native `fetch()` and can be wired through DI like other Bun Server modules.

## Module Map

| Module | Primary Token | Purpose |
|--------|---------------|---------|
| `AiModule` | `AI_SERVICE_TOKEN` | Unified LLM access, Tool Calling, streaming |
| `ConversationModule` | `CONVERSATION_SERVICE_TOKEN` | Multi-turn history and memory management |
| `PromptModule` | `PROMPT_SERVICE_TOKEN` | Prompt templates and versioning |
| `EmbeddingModule` | `EMBEDDING_SERVICE_TOKEN` | Text embedding generation |
| `VectorStoreModule` | `VECTOR_STORE_TOKEN` | Vector storage and similarity retrieval |
| `RagModule` | `RAG_SERVICE_TOKEN` | Ingest + retrieve RAG pipeline |
| `McpModule` | `MCP_SERVER_TOKEN` | MCP protocol server (tool exposure for AI clients) |
| `AiGuardModule` | `AI_GUARD_SERVICE_TOKEN` | Input safety checks (PII, injection, moderation) |

## Minimal Setup (AiModule)

```typescript
import {
  AI_SERVICE_TOKEN,
  AiModule,
  AiService,
  Application,
  Body,
  Controller,
  Inject,
  Injectable,
  Module,
  OllamaProvider,
  POST,
} from "@dangao/bun-server";

AiModule.forRoot({
  providers: [{ name: "ollama", provider: OllamaProvider, config: {}, default: true }],
  fallback: true,
  timeout: 30000,
});

@Injectable()
class ChatService {
  constructor(@Inject(AI_SERVICE_TOKEN) private readonly ai: AiService) {}

  async chat(message: string) {
    return this.ai.complete({ messages: [{ role: "user", content: message }] });
  }
}

@Controller("/chat")
class ChatController {
  constructor(private readonly chatService: ChatService) {}

  @POST("/")
  async post(@Body() body: { message: string }) {
    return this.chatService.chat(body.message);
  }
}

@Module({
  imports: [AiModule],
  controllers: [ChatController],
  providers: [ChatService],
})
class AppModule {}

const app = new Application({ port: 3000 });
app.registerModule(AppModule);
app.listen();
```

## Streaming (SSE) Pattern

```typescript
const stream = aiService.stream({
  messages: [{ role: "user", content: "Explain Bun runtime in 3 points." }],
});

return new Response(stream, {
  headers: {
    "Content-Type": "text/event-stream",
    "Cache-Control": "no-cache",
    Connection: "keep-alive",
  },
});
```

## Typical Composition for RAG

1. Configure `EmbeddingModule.forRoot(...)`.
2. Configure `VectorStoreModule.forRoot(...)`.
3. Configure `RagModule.forRoot(...)` and import all three modules.
4. Use `RAG_SERVICE_TOKEN` to ingest and retrieve context.

## MCP Integration Notes

- `McpModule` is for exposing server tools over MCP protocol.
- Typical config:
  - `transport: "sse"`
  - `path: "/mcp"`
  - `serverInfo: { name, version }`
- Register tool handlers via decorators and scan them through registry/service at startup.

## Safety and Production Notes

- Put `AiGuardModule` before LLM calls in user-facing routes.
- Enable response caching for repeat prompts (combine with `CacheModule`).
- Track usage and cost fields from AI responses for observability.
- Prefer provider fallback in production to reduce single-provider outage risk.

## Best Practices

1. Use Ollama as local default provider during development.
2. Keep AI module configuration in `forRoot()` and call it before module definitions.
3. Separate concerns into dedicated modules:
   - `chat/` for completion and streaming
   - `knowledge/` for embedding + vector + rag
   - `safety/` for guard policies
4. Always set explicit timeout and retry/fallback strategy.
5. For user sessions, combine `ConversationModule` with `SessionModule` or durable stores.

## Related Resources

- [Quick Start](quickstart.md)
- [Module System](module-system.md)
- [Cache Module](cache.md)
- [Security Module](security.md)
