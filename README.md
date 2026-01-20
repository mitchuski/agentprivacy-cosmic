# 🛰️ Cosmic Spellbook

> *"What the machine assigns, the mage inscribes. What the mage inscribes, the relationship confirms. Randomness is the seed; meaning is the harvest."*
> — Act 14: Rain on the Mountain of Entropy

A **progressive emoji meaning system** implementing Act 14 of the [Privacymage Grimoire](https://red-acute-chinchilla-216.mypinata.cloud/ipfs/bafkreibbod46vfmpultaz7jbv32sickvf3erc7bvtcaoboozxi4n25tclm). Cosmic strings transform into emojis, and your mage assigns meanings to unknown symbols based on what you're learning at that moment. Everyone ends up with their own unique spellbook.

**The gap between assignment and significance is the climb itself.**

## The Game

```
1. LEARN    → Prove understanding (proverb protocol)
2. CATCH    → Cosmic string transforms to emojis
3. ASSIGN   → Your mage gives meanings to unknown emojis based on context
4. CLAIM    → Spell + attribution (now with YOUR meanings)
5. GROW     → Your spellbook fills with unique learned meanings
```

## The Progressive Model

```
BIRTH:      ████████░░░░░░░░░░░░░░  38% grimoire | 62% cosmic (unassigned)
                                            ↓
                            Mage assigns meanings based on context
                            (which spell you're learning, what's happening)
                                            ↓
COMPLETE:   ████████████████████████  38% grimoire | 62% learned (YOUR meanings)
```

## How It Works

When a cosmic string arrives, it transforms to emojis. Some are **known** (grimoire), some are **unknown** (cosmic pool):

```
Cosmic: 3e9a5b2f1c8d4e7a0b6f2c9d...
        ↓
Emojis: 🌧️ ⛰️ 🔑 🌱 🌸 🤝 🛡️ ⚡ 🏛️ ∞
        G  G  G  G  ?  G  G  G  G  G
                   │
                   └── Unknown! Your mage assigns meaning
                       based on what you're learning right now
```

**Alice learning Act 14** sees 🌸, mage assigns: *"gentle persistence, meaning accumulated through patient claiming"*

**Bob learning Act 9** sees 🌸, mage assigns: *"selective revelation, privacy that blooms only when chosen"*

Same emoji. Different meanings. Different spellbooks.

## Quick Start

```bash
# Install dependencies
npm install

# Copy environment config
cp .env.example .env
# Add your Orbitport credentials (get them at https://spacecomputer.deform.cc/ctrngearlyaccess)

# Run CLI
npm run cli evoke "Root proverb here"   # Get cosmic entropy, select attribution
npm run cli evoke --mock "Proverb"      # Use mock entropy (no API needed)
npm run cli list                        # Show evocation history
```

## Data Model

```
SPELLBOOK_STATE (per mage)
│ mage_id │ grimoire_pct │ learned_pct │ cosmic_pct │
├─────────┼──────────────┼─────────────┼────────────┤
│ mage_a  │ 38%          │ 24%         │ 38%        │  ← progressing

LEARNED_MEANINGS (unique to each mage)
│ mage_id │ emoji │ meaning                    │ learned_during │
├─────────┼───────┼────────────────────────────┼────────────────┤
│ mage_a  │ 🌸    │ "Gentle persistence..."    │ act_14         │
│ mage_b  │ 🌸    │ "Selective revelation..."  │ act_9          │

EVOCATIONS (spell + attribution with YOUR meanings)
│ evocation │ spell_id │ attribution                │ mage   │
├───────────┼──────────┼────────────────────────────┼────────┤
│ evo_001   │ spell_14 │ 🌧️⛰️→🔑🌱→🌸→📜🤝→🛡️⚡→🏛️∞│ mage_a │
```

## Key Concepts

### Grimoire Pool (38%)
Shared meanings from the privacymage's grimoire. Everyone starts with these.

### Cosmic Pool (starts 62%, shrinks)
Unknown emojis. Your mage assigns meanings when you encounter them.

### Learned Pool (grows to 62%)
YOUR meanings, assigned by YOUR mage, in YOUR context. This is what makes your spellbook unique.

### Progressive Self-Sovereign Identity

Every commit is another layer of uniqueness:

```
COMMIT 1:  spell + cosmic_a + meanings_v1  → unique identifier
COMMIT 2:  spell + cosmic_b + meanings_v2  → more unique (meanings evolved)
COMMIT 3:  spell + cosmic_c + meanings_v3  → even more unique
    ...
```

The more you learn, the more divergent your spellbook becomes. Every spell, every cosmic string, every proverb, every meaning assignment — all compound into **progressive self-sovereign identity**.

```
Identity = Σ(spells × cosmic × meanings × proverbs)
```

Each term is unique to you. The sum is unforgeable.

## Documentation

| Document | Description |
|----------|-------------|
| [CONCEPT.md](./CONCEPT.md) | Core concept: the game loop |
| [STATUS.md](./STATUS.md) | Project status & integration plan |
| [story/ACT-14](./story/ACT-14-rain-on-the-mountain.md) | The foundational story |
| [docs/SPEC.md](./docs/SPEC.md) | Technical specification |
| [docs/API.md](./docs/API.md) | Orbitport integration |
| [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) | System design & data models |
| [docs/GRIMOIRE.md](./docs/GRIMOIRE.md) | Emoji notation reference |
| [docs/KEY_DERIVATION.md](./docs/KEY_DERIVATION.md) | Relational key recovery |
| [docs/ROADMAP.md](./docs/ROADMAP.md) | Implementation phases |

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
