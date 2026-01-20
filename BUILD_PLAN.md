# Cosmic Spellbook Build Plan

## Current State

**Specification Complete** - All design docs are ready:
- SPEC.md - Technical specification
- ARCHITECTURE.md - System design & data models
- GRIMOIRE.md - Emoji notation reference
- ROADMAP.md - Implementation phases
- STATUS.md - Integration plan
- KEY_DERIVATION.md - Relational recovery spec
- API.md - Orbitport integration docs

**No Code Yet** - Need to build:
- `src/` - Core library
- `cli/` - Command-line tool
- `test/` - Test suite
- `web/` - Web interface (later)

---

## Build Order

### Phase 1: Foundation

**No dependencies, can start immediately**

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
  - `core.ts` - Spellbook, CosmicSpell, EmojiSlot, LearnedMeaning, MeaningClaim
  - `emoji.ts` - GrimoireEmoji, CosmicEmoji, EmojiCategory
  - `storage.ts` - StorageAdapter, SpellStorage interfaces

- [ ] **1.3** Create utility functions (`src/utils/`)
  - `hex.ts` - hexToBytes, bytesToHex conversions
  - `emoji.ts` - splitIntoEmojis, emoji validation
  - `crypto.ts` - hashing, ID generation

- [ ] **1.4** Create grimoire pool data (`src/data/grimoire-pool.ts`)
  - Export GRIMOIRE_POOL array with all ~50 defined emojis
  - Categories: core_agents, trust_infrastructure, topology, entropy_paradox, zkp, canon, plurality
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
  // Complete (completion=1): 61.8% grimoire, 38.2% cosmic
  // Linear interpolation between states
  ```
  - `calculate(totalSlots: number, completionRatio: number): SplitResult`
  - Constants: PHI = 1.618033988749895, PHI_INVERSE = 0.618033988749895

- [ ] **2.2** Implement Emoji Mapper (`src/core/emoji-mapper.ts`)
  - `mapBytes(bytes: number[], split: SplitResult, spellbook: Spellbook): EmojiSlot[]`
  - Grimoire slots: draw from grimoire pool + learned meanings
  - Cosmic slots: draw from unclaimed cosmic emojis
  - Selection: `pool[byteValue % pool.length]`

- [ ] **2.3** Implement Mock Orbitport Client (`src/orbitport/mock.ts`)
  - `getCosmicEntropy(): Promise<OrbitportResponse>`
  - Generate deterministic entropy from seed (for testing)
  - Generate random entropy (for demo)
  - Return mock satellite signature

- [ ] **2.4** Implement Spell Generator (`src/core/spell-generator.ts`)
  - Combine: OrbitportClient + PhiSplitter + EmojiMapper
  - `generate(spellbook: Spellbook): Promise<CosmicSpell>`
  - Assemble spell metadata (timestamp, signature, source)

- [ ] **2.5** Write unit tests for core components
  - phi-splitter: boundary conditions (0%, 50%, 100% completion)
  - emoji-mapper: deterministic selection, pool transitions
  - spell-generator: full pipeline with mock entropy

---

### Phase 3: Spellbook Management

**Depends on: Phase 2**

- [ ] **3.1** Implement Local Storage Adapter (`src/storage/local.ts`)
  - Base path: `~/.cosmic-spellbook/`
  - `save(spellbook: Spellbook): Promise<void>`
  - `load(id: string): Promise<Spellbook | null>`
  - Serialize/deserialize Map types to JSON

- [ ] **3.2** Implement Spellbook Manager (`src/core/spellbook-manager.ts`)
  - `create(mageId: string): Promise<Spellbook>`
  - `load(id: string): Promise<Spellbook | null>`
  - `updateCompletion(spellbook: Spellbook): Promise<void>`
  - `recordSpell(spellbook: Spellbook, spell: CosmicSpell): Promise<void>`

- [ ] **3.3** Implement Claiming Protocol (`src/core/claiming-protocol.ts`)
  - `proposeClaim(claim: MeaningClaimProposal): Promise<MeaningClaim>`
  - `confirmClaim(claimId: string): Promise<MeaningClaim>` (simple confirm, no VRC yet)
  - Validation: not already claimed, not grimoire emoji, meaning length

- [ ] **3.4** Create main library entry point (`src/index.ts`)
  - Export all public APIs
  - Export types

---

### Phase 4: CLI Tool

**Depends on: Phase 3**

- [ ] **4.1** Set up CLI framework (`cli/index.ts`)
  - Use commander.js
  - Global options: --mock, --spellbook-id
  - Load/create default spellbook

- [ ] **4.2** Implement `spell` command (`cli/commands/spell.ts`)
  ```bash
  cosmic-spellbook spell              # Generate from Orbitport (needs key)
  cosmic-spellbook spell --mock       # Generate from mock entropy
  cosmic-spellbook spell --seed abc   # Deterministic for testing
  ```
  - Display spell with color coding
  - Show grimoire vs cosmic ratio
  - Indicate claimable emojis

- [ ] **4.3** Implement `status` command (`cli/commands/status.ts`)
  ```bash
  cosmic-spellbook status
  ```
  - Show spellbook ID, creation date
  - Completion ratio with progress bar
  - Spells generated count
  - Learned meanings count

- [ ] **4.4** Implement `claim` command (`cli/commands/claim.ts`)
  ```bash
  cosmic-spellbook claim 🌸 "gentle persistence"
  cosmic-spellbook claim --confirm <claim-id>
  ```
  - Propose new meaning
  - Confirm pending claims

- [ ] **4.5** Implement `meanings` command (`cli/commands/meanings.ts`)
  ```bash
  cosmic-spellbook meanings           # List all learned
  cosmic-spellbook meanings --export  # Export as JSON
  ```

- [ ] **4.6** Create spell display formatter (`cli/formatters/spell.ts`)
  - Chalk colors: grimoire = gold, cosmic = purple, claimable = cyan
  - Show meaning tooltips
  - Progress bar visualization

---

### Phase 5: Key Derivation

**Depends on: Phase 4**

- [ ] **5.1** Implement Key Derivation (`src/core/key-derivation.ts`)
  - `encode(cosmicHex: string, spellbook: Spellbook): EncodedSpell`
  - `decode(emojiSpell: string, spellbook: Spellbook): DecodedResult`
  - `buildOrderedPool(spellbook: Spellbook): OrderedPool`
  - Deterministic pool ordering: grimoire (sorted) + learned (by claim time)

- [ ] **5.2** Implement `encode` command
  ```bash
  cosmic-spellbook encode <hex>       # Encode entropy as emoji spell
  ```

- [ ] **5.3** Implement `decode` command
  ```bash
  cosmic-spellbook decode <spell>     # Decode spell back to entropy
  ```

- [ ] **5.4** Implement `verify` command
  ```bash
  cosmic-spellbook verify <spell> <hex>  # Verify round-trip
  ```

- [ ] **5.5** Implement `derive` command
  ```bash
  cosmic-spellbook derive             # Generate + encode (one step)
  cosmic-spellbook derive --mock      # With mock entropy
  ```

---

### Phase 6: Orbitport Integration (Requires API Key)

**Depends on: Phase 5 + Beacon API key**

- [ ] **6.1** Implement real Orbitport Client (`src/orbitport/client.ts`)
  - Auth0 token generation
  - Token caching with expiry
  - `getCosmicEntropy(): Promise<OrbitportResponse>`
  - Retry logic with exponential backoff

- [ ] **6.2** Implement signature verification
  - ECDSA verification (if public key available)
  - Orbitport attestation fallback

- [ ] **6.3** Implement `beacon` command
  ```bash
  cosmic-spellbook beacon             # Continuous spell generation
  cosmic-spellbook beacon --interval 60
  ```

- [ ] **6.4** Create `.env.example` with required vars
  ```
  ORBITPORT_CLIENT_ID=
  ORBITPORT_CLIENT_SECRET=
  ORBITPORT_AUDIENCE=
  ```

---

### Phase 7: Recovery Protocol (Future)

- [ ] **7.1** Implement Recovery Protocol (`src/core/recovery-protocol.ts`)
  - `attemptRelationalRecovery(kit, emojiSpell): Promise<RecoveryResult>`
  - Challenge-response generation

- [ ] **7.2** Implement recovery CLI commands
  ```bash
  cosmic-spellbook recovery export    # Export VRCs for recovery
  cosmic-spellbook recovery attempt   # Attempt relational recovery
  cosmic-spellbook recovery challenge # Generate recovery challenge
  ```

---

### Phase 8: Web Interface (Future)

- [ ] **8.1** Initialize Next.js project in `/web`
- [ ] **8.2** Create components: SpellDisplay, EmojiSlot, SpellbookProgress, ClaimModal
- [ ] **8.3** Create pages: Home, Spell, Spellbook, Settings
- [ ] **8.4** Create API routes: /api/generate-spell, /api/spellbook, /api/claim
- [ ] **8.5** Integrate with agentprivacy.ai `/cosmic` route

---

## Quick Start Commands

Once built, these will work:

```bash
# Install dependencies
npm install

# Generate spell with mock entropy (no API key needed)
npm run cli spell --mock

# Check spellbook status
npm run cli status

# Claim a meaning
npm run cli claim 🌸 "gentle persistence"

# Confirm the claim
npm run cli claim --confirm <claim-id>

# Encode entropy as emoji spell
npm run cli encode 0a4c2ea21557418b9f3c7d8e...

# Decode spell back to entropy
npm run cli decode "⚔️🔮🌸🎲🛡️..."
```

---

## Files to Create (Summary)

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
│   ├── claiming-protocol.ts
│   └── key-derivation.ts
├── storage/
│   ├── adapter.ts
│   └── local.ts
├── orbitport/
│   ├── client.ts
│   └── mock.ts
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
│   ├── spell.ts
│   ├── status.ts
│   ├── claim.ts
│   ├── meanings.ts
│   ├── encode.ts
│   ├── decode.ts
│   └── beacon.ts
└── formatters/
    └── spell.ts

test/
├── core/
│   ├── phi-splitter.test.ts
│   ├── emoji-mapper.test.ts
│   └── spell-generator.test.ts
└── fixtures/
    └── mock-entropy.ts
```

---

## Notes

- **No Beacon Key?** Use `--mock` flag for all commands. Full functionality works with mock entropy.
- **Deterministic Testing**: Use `--seed` flag to get reproducible spells
- **Progression**: Completion ratio affects the phi-split. As you claim more meanings, more grimoire slots open up.
- **Unique Spellbooks**: The ~5.6% separation gap ensures no two mages who complete their spellbooks will have identical learned meanings.

---

*"Cosmic rain on the mountain of entropy, split at the golden gate, where grimoire and void meet, the mage who claims builds rivers that remember."*
