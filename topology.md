## Topology — who connects to whom

The components and the links between them. 

```mermaid

flowchart LR
    C["Client<br/><i>app / web</i>"]
    R["Router<br/><i>routes the request</i>"]
    A["Care API<br/><i>holds the keys</i>"]
    S["Summarization Service<br/><i>stateless</i>"]
    P["Model provider<br/><i>behind an explicit wrapper</i>"]
    G[("Object storage<br/><i>documents</i>")]
    O[("Logging + tracing")]

    C -->|"Method"| R
    R -->|"HTTPS"| A
    C -->|"HTTPS"| R
    A <-->|"HTTPS · platform-signed token"| S
    S <-->|"provider API"| P
    S -->|"pre-signed URL · ~30s TTL"| G
    S -.->|"spans · structured logs"| O
```

| Link | Carries | Note |
|---|---|---|
| Client → Router | Users's request routed | Client to router by method call |
| Router → Care API | Service call | Summarization from App POV |
| Care API → Summarization | Intent, decrypted content, configuration | **The only inbound path in v0** (D17) |
| Summarization → Model provider | Prompt and completion | **Which provider is configuration, not architecture** — see below |
| Summarization → Trace destination | Spans, and **optionally sampled content** under D23 | A separate destination from the model provider, with its own eligibility and retention |
| Summarization → Object storage | Document fetch | Pre-signed, ~30s, **no IAM grant to us** (D9) |
| Summarization → Observability | Spans and logs | Content suppressed by default (D23) |

