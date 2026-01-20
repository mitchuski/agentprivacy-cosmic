# 🛰️ Cosmic Spellbook

Transform satellite-sourced true random numbers into emoji spell strings using the [Privacymage Grimoire](https://red-acute-chinchilla-216.mypinata.cloud/ipfs/bafkreibbod46vfmpultaz7jbv32sickvf3erc7bvtcaoboozxi4n25tclm) notation system.

## What It Does

1. **Fetches cosmic entropy** from SpaceComputer's Orbitport cTRNG API (satellite-sourced randomness)
2. **Splits bytes at golden ratio** — 38.2% map to known grimoire symbols, 61.8% to cosmic pool
3. **Generates emoji spell strings** — 32 emojis from 32 bytes
4. **Tracks claimed meanings** — users can assign meaning to unclaimed cosmic emojis
5. **Encodes/decodes keys** — bidirectional transformation for relational recovery

## Quick Start

```bash
# Install dependencies
npm install

# Copy environment config
cp .env.example .env
# Add your Orbitport credentials (get them at https://spacecomputer.deform.cc/ctrngearlyaccess)

# Run CLI
npm run cli spell              # Generate a spell
npm run cli spell --mock       # Generate with mock entropy (no API needed)
npm run cli status             # Show spellbook state
```

## Core Flow

```
Orbitport API → 32-byte hex → φ-split → [grimoire|cosmic] pools → emoji spell
```

## Key Concepts

### Golden Ratio Split (φ ≈ 1.618)

| State | Grimoire Pool | Cosmic Pool |
|-------|---------------|-------------|
| Birth | 38.2% (known) | 61.8% (unknown) |
| Complete | 61.8% (learned) | 38.2% (remaining) |

As meanings are claimed for cosmic emojis, the ratio inverts.

### Bidirectional Transformation

```typescript
encode(cosmicHex, spellbook) → emojiSpell
decode(emojiSpell, spellbook) → cosmicHex
```

Decoding requires YOUR spellbook — the unique product of YOUR claimed meanings.

## Documentation

| Document | Description |
|----------|-------------|
| [SPEC.md](./docs/SPEC.md) | Technical specification |
| [API.md](./docs/API.md) | Orbitport integration |
| [ARCHITECTURE.md](./docs/ARCHITECTURE.md) | System design & data models |
| [GRIMOIRE.md](./docs/GRIMOIRE.md) | Emoji notation reference |
| [KEY_DERIVATION.md](./docs/KEY_DERIVATION.md) | Relational key recovery |
| [ROADMAP.md](./docs/ROADMAP.md) | Implementation phases |

## Dependencies

- **SpaceComputer Orbitport** — cTRNG API for cosmic entropy
- **Privacymage Grimoire** — emoji notation system (~50 defined symbols)
- **Optional**: LLM provider for interpretation (OpenAI, Anthropic, Ollama)

## License

MIT

## Links

- [SpaceComputer Docs](https://docs.spacecomputer.io/)
- [Orbitport Early Access](https://spacecomputer.deform.cc/ctrngearlyaccess)
- [Privacymage Grimoire v8](https://red-acute-chinchilla-216.mypinata.cloud/ipfs/bafkreibbod46vfmpultaz7jbv32sickvf3erc7bvtcaoboozxi4n25tclm)
