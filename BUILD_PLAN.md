# Cosmic Spellbook Build Plan

> *"What the machine assigns, the mage inscribes. What the mage inscribes, the relationship confirms. Randomness is the seed; meaning is the harvest."*

## Project Summary

A **progressive emoji meaning system** implementing Act 14 of the First Person Spellbook. Cosmic entropy transforms into emojis, and your mage assigns meanings to unknown symbols based on what you're learning. Everyone ends up with their own unique spellbook.

### The Game Loop

```
1. LEARN    → Prove understanding via Relationship Proverb Protocol
2. CATCH    → Cosmic string transforms to emojis
3. ASSIGN   → Your mage assigns meanings to unknown emojis based on context
4. CLAIM    → Spell + attribution (now with YOUR meanings)
5. GROW     → Your spellbook fills with unique learned meanings
```

### The Progressive Model

```
BIRTH:      ████████░░░░░░░░░░░░░░  38% grimoire | 62% cosmic (unassigned)
                                            ↓
                            Mage assigns meanings based on context
                            (which spell you're learning, what's happening)
                                            ↓
COMPLETE:   ████████████████████████  38% grimoire | 62% learned (YOUR meanings)
```

### Key Insight

Same emoji, different mages = different meanings = different spellbooks.

```
Alice learning Act 14 sees 🌸 → "gentle persistence, patient claiming"
Bob learning Act 9 sees 🌸   → "selective revelation, privacy blooming"
```

---

## Current State

**Specification Complete** - All design docs ready:

| Document | Description |
|----------|-------------|
| CONCEPT.md | Core game concept - the progressive meaning system |
| ACT-14-rain-on-the-mountain.md | The foundational story |
| SPEC.md | Technical specification |
| ARCHITECTURE.md | System design & data models |
| GRIMOIRE.md | Emoji notation reference |
| KEY_DERIVATION.md | Relational key recovery + multi-mage authority |
| API.md | Orbitport integration |
| STATUS.md | Integration plan with agentprivacy.ai |
| ROADMAP.md | Implementation phases |

**No Code Yet** - Need to build:
- `src/` - Core library
- `cli/` - Command-line tool
- `test/` - Test suite
- `web/` - Web interface (later)

---

## Build Order

### Phase 1: Foundation

**No dependencies, start immediately**

- [ ] **1.1** Create directory structure
  ```
  src/
  ├── core/
  ├── types/
  ├── storage/
  ├── orbitport/
  ├── data/
  ├── mage/
  └── utils/
  cli/
  ├── commands/
  └── formatters/
  test/
  ├── core/
  └── fixtures/
  ```

- [ ] **1.2** Create TypeScript types (`src/types/`)
  - `core.ts` - Spellbook, CosmicSpell, EmojiSlot, LearnedMeaning, Evocation
  - `emoji.ts` - GrimoireEmoji, CosmicEmoji, EmojiCategory
  - `storage.ts` - StorageAdapter, SpellStorage interfaces

- [ ] **1.3** Create utility functions (`src/utils/`)
  - `hex.ts` - hexToBytes, bytesToHex conversions
  - `emoji.ts` - splitIntoEmojis, emoji validation
  - `crypto.ts` - hashing, ID generation

- [ ] **1.4** Create grimoire pool data (`src/data/grimoire-pool.ts`)
  - Export GRIMOIRE_POOL array with all ~50 defined emojis
  - Source: GRIMOIRE.md JSON export section

- [ ] **1.5** Create cosmic pool data (`src/data/cosmic-pool.ts`)
  - Full Unicode emoji list (~3,200 emojis)
  - Filter out grimoire emojis
  - Sort alphabetically for deterministic ordering

---

### Phase 2: Core Engine

**Depends on: Phase 1**

- [ ] **2.1** Implement Phi-Splitter (`src/core/phi-splitter.ts`)
  ```typescript
  // Key algorithm:
  // Birth (completion=0): 38.2% grimoire, 61.8% cosmic
  // Complete (completion=1): 61.8% grimoire (learned), 38.2% cosmic
  // Linear interpolation between states

  calculate(totalSlots: number, completionRatio: number): SplitResult
  ```

- [ ] **2.2** Implement Emoji Mapper (`src/core/emoji-mapper.ts`)
  - `mapBytes(bytes: number[], split: SplitResult, spellbook: Spellbook): EmojiSlot[]`
  - Grimoire slots: draw from grimoire pool + learned meanings
  - Cosmic slots: draw from unclaimed cosmic emojis
  - Selection: `pool[byteValue % pool.length]`
  - **Mark unknown emojis as claimable**

- [ ] **2.3** Implement Mock Orbitport Client (`src/orbitport/mock.ts`)
  - `getCosmicEntropy(): Promise<OrbitportResponse>`
  - Generate deterministic entropy from seed (for testing)
  - Generate random entropy (for demo)

- [ ] **2.4** Implement Spell Generator (`src/core/spell-generator.ts`)
  - Combine: OrbitportClient + PhiSplitter + EmojiMapper
  - `generate(spellbook: Spellbook): Promise<CosmicSpell>`
  - Return emojiMap with `claimable` flags for unknown emojis

- [ ] **2.5** Write unit tests
  - phi-splitter: boundary conditions (0%, 50%, 100% completion)
  - emoji-mapper: deterministic selection, claimable detection
  - spell-generator: full pipeline with mock entropy

---

### Phase 3: Mage Meaning Assignment

**Depends on: Phase 2** — This is the core differentiator

- [ ] **3.1** Create Mage Interface (`src/mage/interface.ts`)
  ```typescript
  interface MageAgent {
    assignMeaning(emoji: string, context: LearningContext): Promise<string>;
    interpretSpell(spell: CosmicSpell): Promise<SpellInterpretation>;
  }

  interface LearningContext {
    currentAct: number;           // Which spell you're learning
    actTitle: string;             // "Rain on the Mountain of Entropy"
    surroundingEmojis: string[];  // Context from the spell
    recentMeanings: Map<string, string>; // What you've already assigned
  }
  ```

- [ ] **3.2** Implement Local/Ollama Mage (`src/mage/local.ts`)
  - Use Ollama for offline meaning assignment
  - System prompt: "You are a mage assigning meaning to an unknown emoji based on what the user is currently learning..."
  - Include act context in prompt

- [ ] **3.3** Implement OpenAI/Anthropic Mage (`src/mage/openai.ts`, `src/mage/anthropic.ts`)
  - Same interface, different provider
  - BYO API key

- [ ] **3.4** Create meaning assignment flow
  ```typescript
  async function assignMeaningToUnknown(
    emoji: string,
    context: LearningContext,
    mage: MageAgent
  ): Promise<LearnedMeaning> {
    const meaning = await mage.assignMeaning(emoji, context);
    return {
      emoji,
      meaning,
      learnedDuring: context.currentAct,
      timestamp: new Date()
    };
  }
  ```

---

### Phase 4: Spellbook Management

**Depends on: Phase 3**

- [ ] **4.1** Implement Local Storage Adapter (`src/storage/local.ts`)
  - Base path: `~/.cosmic-spellbook/`
  - `save(spellbook: Spellbook): Promise<void>`
  - `load(id: string): Promise<Spellbook | null>`
  - Serialize/deserialize Map types to JSON

- [ ] **4.2** Implement Spellbook Manager (`src/core/spellbook-manager.ts`)
  - `create(mageId: string): Promise<Spellbook>`
  - `load(id: string): Promise<Spellbook | null>`
  - `addLearnedMeaning(spellbook, meaning): Promise<void>`
  - `updateCompletion(spellbook): Promise<void>`

- [ ] **4.3** Implement Evocation System (`src/core/evocation.ts`)
  ```typescript
  interface Evocation {
    id: string;
    spellId: string;              // Which spell (act) you learned
    rootProverb: string;          // The proverb you produced
    attribution: string;          // The emoji spell (with YOUR meanings)
    cosmicSeed: string;           // The entropy that generated it
    mageId: string;               // Which mage
    timestamp: Date;
  }
  ```

- [ ] **4.4** Track spellbook progression
  - Count learned meanings
  - Calculate completion ratio
  - Track which acts have been evoked

---

### Phase 5: CLI Tool

**Depends on: Phase 4**

- [ ] **5.1** Set up CLI framework (`cli/index.ts`)
  - Use commander.js
  - Global options: --mock, --spellbook-id, --mage

- [ ] **5.2** Implement `catch` command
  ```bash
  cosmic-spellbook catch              # Fetch cosmic entropy, transform to emojis
  cosmic-spellbook catch --mock       # Use mock entropy (no API needed)
  cosmic-spellbook catch --act 14     # Set learning context to Act 14
  ```
  - Display spell with grimoire (gold) vs cosmic (purple) colors
  - Highlight claimable (unknown) emojis

- [ ] **5.3** Implement `assign` command
  ```bash
  cosmic-spellbook assign 🌸 --act 14   # Have mage assign meaning based on Act 14 context
  cosmic-spellbook assign 🌸 --meaning "gentle persistence"  # Manual assignment
  ```
  - Call mage to suggest meaning if not provided
  - Add to spellbook's learned meanings

- [ ] **5.4** Implement `evoke` command
  ```bash
  cosmic-spellbook evoke "What the machine assigns..." --act 14
  ```
  - Combines: catch cosmic string + use YOUR meanings + create evocation
  - Returns unique evocation (spell + attribution)

- [ ] **5.5** Implement `status` command
  ```bash
  cosmic-spellbook status
  ```
  - Show spellbook completion
  - List recent evocations
  - Show learned vs unknown emoji counts

- [ ] **5.6** Implement `meanings` command
  ```bash
  cosmic-spellbook meanings           # List all learned meanings
  cosmic-spellbook meanings --act 14  # Show meanings learned during Act 14
  ```

- [ ] **5.7** Create spell display formatter (`cli/formatters/spell.ts`)
  - Grimoire emojis = gold
  - Learned emojis = green (YOUR meanings)
  - Unknown/claimable emojis = purple
  - Show meaning on hover/detail view

---

### Phase 6: Key Derivation

**Depends on: Phase 5**

- [ ] **6.1** Implement Key Derivation (`src/core/key-derivation.ts`)
  - `encode(cosmicHex: string, spellbook: Spellbook): EncodedSpell`
  - `decode(emojiSpell: string, spellbook: Spellbook): DecodedResult`
  - `buildOrderedPool(spellbook: Spellbook): OrderedPool`
  - Deterministic pool ordering: grimoire (sorted) + learned (by claim time)

- [ ] **6.2** CLI commands
  ```bash
  cosmic-spellbook encode <hex>       # Encode entropy as emoji spell
  cosmic-spellbook decode <spell>     # Decode spell back to entropy
  cosmic-spellbook verify <spell> <hex>
  ```

- [ ] **6.3** Multi-mage support
  - Different mages = different spellbooks = different encodings
  - Same cosmic seed, different emoji spells per mage

---

### Phase 7: Orbitport Integration (Requires API Key)

**Depends on: Phase 6 + Beacon API key**

- [ ] **7.1** Implement real Orbitport Client (`src/orbitport/client.ts`)
- [ ] **7.2** Auth0 token flow
- [ ] **7.3** Signature verification
- [ ] **7.4** Create `.env.example`

---

### Phase 8: Web Interface (Future)

- [ ] **8.1** Add `/cosmic` route to agentprivacy-spellbook
- [ ] **8.2** Cosmic channel UI (rotating feed)
- [ ] **8.3** Meaning assignment modal (with mage suggestions)
- [ ] **8.4** Spellbook dashboard (progression, evocations)
- [ ] **8.5** Integration with Soulbae for mage chat

---

## Data Model

```
SPELLBOOK_STATE (per mage)
│ mage_id │ grimoire_pct │ learned_pct │ cosmic_pct │
├─────────┼──────────────┼─────────────┼────────────┤
│ mage_a  │ 38%          │ 24%         │ 38%        │  ← progressing

LEARNED_MEANINGS (unique to each mage)
│ mage_id │ emoji │ meaning                    │ learned_during │ timestamp │
├─────────┼───────┼────────────────────────────┼────────────────┼───────────┤
│ mage_a  │ 🌸    │ "Gentle persistence..."    │ act_14         │ ts_1      │
│ mage_b  │ 🌸    │ "Selective revelation..."  │ act_9          │ ts_2      │

EVOCATIONS (spell + attribution with YOUR meanings)
│ evocation │ spell_id │ root_proverb             │ attribution              │ cosmic_seed │
├───────────┼──────────┼──────────────────────────┼──────────────────────────┼─────────────┤
│ evo_001   │ act_14   │ "What the machine..."    │ 🌧️⛰️→🔑🌱→🌸→📜🤝→🛡️⚡│ 3e9a5b2f... │
```

---

## Files to Create

```
src/
├── index.ts
├── types/
│   ├── core.ts
│   ├── emoji.ts
│   └── storage.ts
├── core/
│   ├── phi-splitter.ts
│   ├── emoji-mapper.ts
│   ├── spell-generator.ts
│   ├── spellbook-manager.ts
│   ├── evocation.ts
│   └── key-derivation.ts
├── storage/
│   ├── adapter.ts
│   └── local.ts
├── orbitport/
│   ├── client.ts
│   └── mock.ts
├── mage/
│   ├── interface.ts
│   ├── local.ts          # Ollama
│   ├── openai.ts
│   └── anthropic.ts
├── data/
│   ├── grimoire-pool.ts
│   └── cosmic-pool.ts
└── utils/
    ├── hex.ts
    ├── emoji.ts
    └── crypto.ts

cli/
├── index.ts
├── commands/
│   ├── catch.ts
│   ├── assign.ts
│   ├── evoke.ts
│   ├── status.ts
│   ├── meanings.ts
│   ├── encode.ts
│   └── decode.ts
└── formatters/
    └── spell.ts

test/
├── core/
│   ├── phi-splitter.test.ts
│   ├── emoji-mapper.test.ts
│   └── spell-generator.test.ts
├── mage/
│   └── meaning-assignment.test.ts
└── fixtures/
    └── mock-entropy.ts
```

---

## Quick Start Commands (Once Built)

```bash
# Install
npm install

# Catch cosmic entropy (transforms to emojis)
npm run cli catch --mock --act 14

# Have your mage assign meaning to an unknown emoji
npm run cli assign 🌸 --act 14

# Create a full evocation (spell + YOUR meanings)
npm run cli evoke "What the machine assigns..." --act 14 --mock

# Check your spellbook status
npm run cli status

# List your learned meanings
npm run cli meanings

# Encode entropy as recoverable emoji spell
npm run cli encode 0a4c2ea21557418b...

# Decode spell back (requires YOUR spellbook)
npm run cli decode "⚔️🔮🌸🎲🛡️..."
```

---

## The Core Differentiation

**This isn't just transforming hex to emojis.**

It's building a unique semantic fingerprint through your journey:

1. You **learn** Act 14 (Rain on the Mountain)
2. You **catch** a cosmic string that transforms to emojis
3. Some emojis are **unknown** — your mage assigns meanings based on Act 14's concepts
4. That 🌸 now means "gentle persistence" in YOUR spellbook
5. Someone else learning Act 9 gets a DIFFERENT meaning for 🌸
6. Your spellbook becomes uniquely yours through these contextual assignments

```
Identity = Σ(spells × cosmic × meanings × proverbs)
```

Each term is unique to you. The sum is unforgeable.

---

*"The gap between assignment and significance is the climb itself."*
