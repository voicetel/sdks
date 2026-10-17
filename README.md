# 📞 VoiceTel SDKs

Official client libraries for the [VoiceTel REST API](https://voicetel.com/docs/api/v2.2/). Twelve languages, one CLI, plus ready-to-import Postman & Bruno collections — all hand-written, all targeting API **v2.2.17**, all MIT-licensed, all public.

![API](https://img.shields.io/badge/API-v2.2.17-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![SDKs](https://img.shields.io/badge/SDKs-12-brightgreen)

## 📚 Table of Contents

- [Pick your language](#-pick-your-language)
- [SDKs](#-sdks)
- [Tooling](#-tooling)
- [What every SDK gives you](#-what-every-sdk-gives-you)
- [API Documentation](#-api-documentation)
- [Authentication, in 30 seconds](#-authentication-in-30-seconds)
- [Versioning & spec parity](#-versioning--spec-parity)
- [Contributing](#-contributing)
- [License](#-license)

## 🎯 Pick your language

Each row links straight to the SDK repo. All twelve cover the same **79 operations across 10 resource groups** (Account, ACL, Authentication, e911, Gateways, iNumbering, Lookups, Messaging, Numbers, Support).

| Language | Repo | Install | Idiomatic style |
|---|---|---|---|
| 🐍 **Python** | [python-sdk](https://github.com/voicetel/python-sdk) | `pip install voicetel-api` | `Client` + `AsyncClient`, Pydantic v2 |
| 🟦 **TypeScript / JavaScript** | [typescript-sdk](https://github.com/voicetel/typescript-sdk) | `npm install @voicetel.com/sdk` | ESM+CJS, `fetch`-based, zero runtime deps |
| 🐹 **Go** | [go-sdk](https://github.com/voicetel/go-sdk) | `go get github.com/voicetel/go-sdk/v2` | `context.Context`, stdlib `net/http` |
| ☕ **Java** | [java-sdk](https://github.com/voicetel/java-sdk) | `com.voicetel:voicetel-sdk:2.2.17` | Java 17 records, Jackson, `java.net.http` |
| 🟪 **Kotlin** | [kotlin-sdk](https://github.com/voicetel/kotlin-sdk) | `com.voicetel:voicetel-sdk-kotlin:2.2.17` | Coroutines, `kotlinx.serialization`, Ktor |
| 🟣 **Scala** | [scala-sdk](https://github.com/voicetel/scala-sdk) | `com.voicetel:voicetel-sdk_3:2.2.17` | Scala 3, sttp, Circe, `Future` |
| 🐘 **PHP** | [php-sdk](https://github.com/voicetel/php-sdk) | `composer require voicetel/sdk` | PHP 8.1+, Guzzle, enums, PHPStan level 8 |
| 💎 **Ruby** | [ruby-sdk](https://github.com/voicetel/ruby-sdk) | `gem install voicetel` | Faraday, `Data.define`, RBS types |
| 🟪 **C# / .NET** | [csharp-sdk](https://github.com/voicetel/csharp-sdk) | `dotnet add package VoiceTel.Sdk` | `async/await`, `System.Text.Json`, .NET 8 + .NET Standard 2.1 |
| 🦀 **Rust** | [rust-sdk](https://github.com/voicetel/rust-sdk) | `cargo add voicetel` | `reqwest` + `serde` + `tokio` |
| 🦅 **Swift** | [swift-sdk](https://github.com/voicetel/swift-sdk) | SwiftPM: `https://github.com/voicetel/swift-sdk` | `async throws`, `URLSession`, macOS/iOS/tvOS/watchOS/Linux |
| 🌙 **Lua** (FreeSWITCH mod_lua) | [lua-sdk](https://github.com/voicetel/lua-sdk) | drop-in to `/etc/freeswitch/scripts/` | Lua 5.2, `freeswitch.Curl` + LuaSocket fallback |

## 🛠️ Tooling

| Project | Repo | What it does |
|---|---|---|
| 🖥️ **CLI / REPL** | [cli](https://github.com/voicetel/cli) | `go install github.com/voicetel/cli@latest` — interactive REPL with command history and tab autocomplete |
| 📬 **API Collections** | [api-collections](https://github.com/voicetel/api-collections) | Postman + Bruno collections covering every endpoint; auto-installs the bearer after login |

## ✨ What every SDK gives you

- **Strongly typed end-to-end** — every request body and response payload is a typed struct / class / record in the host language. The one exception is endpoints the spec leaves schema-less (e.g. the v2.2.17 messaging-health surface): there the SDKs return the decoded JSON as the language's raw/dynamic type (`Any` / `map[string]any` / `Record<string, unknown>` / `JsonElement` / `JSONValue` / …) rather than invent field names the spec doesn't guarantee.
- **Auto-retry on 429 / 5xx** with `Retry-After` honored, exponential backoff capped at 8 s. Default `maxRetries = 2`.
- **Structured errors** — every SDK exposes an `ApiError` (or equivalent) with a typed kind (`bad_request`, `authentication`, `permission_denied`, `not_found`, `conflict`, `rate_limit`, `server`, `unknown`). Catch broadly or pattern-match on the kind.
- **Built-in login flow** — `client.login(username, password)` exchanges credentials for a bearer token and installs it on the transport. One call.
- **Rate-limit aware** — the 6 req/hour/IP cap on `GET /v2.2/account`, `GET /v2.2/account/cdr`, `GET /v2.2/account/recurring-charges`, and `POST /v2.2/account/api-key` (plus the 30 checks/hour/account cap on `POST /v2.2/messaging/health/check`) is documented in every SDK's README.
- **Zero codegen footprint** — every line is hand-written. No Swagger Codegen artifacts, no anonymous generated types.
- **CI green on push** — every repo has GitHub Actions running tests + lint + coverage on every commit.
- **MIT-licensed** — use them anywhere, commercial or otherwise.

## 📖 API Documentation

- **Reference docs:** [voicetel.com/docs/api/v2.2/](https://voicetel.com/docs/api/v2.2/)
- **Interactive playground:** [voicetel.com/docs/api/v2.2/playground/](https://voicetel.com/docs/api/v2.2/playground/) — try the API in your browser without writing any code
- **API credentials:** [voicetel.com/docs/api/v2.2/credentials/](https://voicetel.com/docs/api/v2.2/credentials/)

## 🔑 Authentication, in 30 seconds

Every endpoint requires `Authorization: Bearer <apikey>` **except** `POST /v2.2/account/api-key`, which exchanges username + password for a fresh 32-hex key. The `client.login(...)` helper in every SDK handles this for you. Re-fetch the key after any password change.

Pick your language and the auth code looks like one of:

```python
# Python
from voicetel import Client
with Client() as c:
    c.login(username=1000000001, password="hunter2")
    print(c.account.get())
```

```go
// Go
c := voicetel.NewClient()
c.Login(ctx, 1000000001, "hunter2")
me, _ := c.Account.Get(ctx)
```

```ts
// TypeScript
const client = new VoiceTelClient();
await client.login(1000000001, "hunter2");
const me = await client.account.get();
```

```ruby
# Ruby
client = VoiceTel::Client.new
client.login(username: 1000000001, password: "hunter2")
me = client.account.get
```

```rust
// Rust
let client = voicetel::Client::builder().build();
client.login(1000000001, "hunter2").await?;
let me = client.account().get().await?;
```

```kotlin
// Kotlin
val client = VoiceTelClient { }
client.login(1000000001, "hunter2")
val me = client.account.get()
```

Every other SDK follows the same shape with idiomatic language conventions — read its README for the exact snippet.

> Don't have credentials yet? Get them at **[voicetel.com/docs/api/v2.2/credentials](https://voicetel.com/docs/api/v2.2/credentials/)**.

## 🔢 Versioning & spec parity

- All SDKs target **VoiceTel API v2.2.17**.
- SDK package version tracks the API version where the language ecosystem allows it (Python `2.2.17`, Java `2.2.17`, etc.). Go and Rust need SemVer-clean versions, so their releases use three-part tags such as `v2.2.17`; the Go module path carries the `/v2` suffix (`github.com/voicetel/go-sdk/v2`) so those tags resolve with `go get`.
- The OpenAPI source of truth is at [api.voicetel.com/doc/v2.2.json](https://api.voicetel.com/doc/v2.2.json). When the spec bumps, the SDKs get updated commits; each repo's CHANGELOG (or commit history) records the change.

## 🤝 Contributing

Each repo has its own contribution rules in its `README`. In general:

- Open an issue on the affected SDK's repo describing the change.
- Send a PR against `main`. CI runs tests + lint + coverage on every PR.
- Bugs that span multiple SDKs (spec drift, model mistakes) should be reported here in `voicetel/sdks` and we'll fan them out.

## 📄 License

Every SDK in this catalogue is licensed under the **MIT License**. See each repo's `LICENSE` file (and this repo's [LICENSE](LICENSE)).

---

| Sponsor | Contribution |
|---------|--------------|
| [VoiceTel Communications](https://voicetel.com) | Primary development and production hosting |
