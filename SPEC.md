# Technical Specification

## Overview

Cosmic Spellbook is a protocol for transforming satellite-sourced true random numbers into semantically meaningful emoji spell strings. The system enables AI mage agents to "claim" meaning from cosmic entropy, building personalized spellbooks that evolve through use.

## Core Components

### 1. Entropy Source (Orbitport cTRNG)

**Input**: API call to SpaceComputer Orbitport
**Output**: 32-byte hex string + cryptographic signature

```typescript
interface OrbitportResponse {
  service: "trng";
  src: "aptosorbital" | "derived";
  data: string;        // 64 hex characters (32 bytes)
  signature: {
    value: string;     // ECDSA signature
    pk: string;        // Public key (may be empty)
  };
}
```

The satellite signs all generated data, providing cryptographic proof of cosmic origin.

### 2. Hex-to-Emoji Mapper

**Input**: 32-byte hex string
**Output**: Array of emoji positions (32 slots)

#### Mapping Strategy

Each byte (2 hex chars) maps to one emoji slot:

```typescript
function hexToEmojiSlots(hex: string): number[] {
  const slots: number[] = [];
  for (let i = 0; i < hex.length; i += 2) {
    const byte = parseInt(hex.substring(i, i + 2), 16);
    slots.push(byte);
  }
  return slots; // 32 values, each 0-255
}
```

### 3. Golden Ratio Splitter (φ-Splitter)

**Input**: 32 emoji slots + completion ratio
**Output**: Tagged slots (grimoire vs cosmic)

#### The Split

At birth (completion = 0):
- First 12 slots (38.2%): Draw from Grimoire defined notation
- Remaining 20 slots (61.8%): Draw from cosmic expansion pool

At completion (completion = 1):
- First 20 slots (61.8%): Draw from learned meanings
- Remaining 12 slots (38.2%): Core grimoire constants

```typescript
interface SplitConfig {
  grimoireRatio: number;  // 0.382 at birth, 0.618 at completion
  cosmicRatio: number;    // 0.618 at birth, 0.382 at completion
  completionProgress: number; // 0.0 to 1.0
}

function calculateSplit(totalSlots: number, progress: number): { grimoire: number; cosmic: number } {
  const PHI = 1.618033988749895;
  const PHI_INVERSE = 0.618033988749895;
  
  // Interpolate between birth and completion ratios
  const grimoireRatio = (1 - PHI_INVERSE) + (progress * (PHI_INVERSE - (1 - PHI_INVERSE)));
  const grimoireSlots = Math.round(totalSlots * grimoireRatio);
  
  return {
    grimoire: grimoireSlots,
    cosmic: totalSlots - grimoireSlots
  };
}
```

### 4. Emoji Pool Management

#### Grimoire Pool (Defined)

The ~50+ emojis from the Privacymage Grimoire with established semantic meanings:

```typescript
interface GrimoireEmoji {
  emoji: string;
  category: "core_agents" | "trust_infrastructure" | "topology" | "entropy_paradox" | "zkp" | "canon" | "plurality";
  meaning: string;
  keywords: string[];
}

// Example entries
const GRIMOIRE_POOL: GrimoireEmoji[] = [
  { emoji: "⚔️", category: "core_agents", meaning: "swordsman / blade / privacy / boundary-making", keywords: ["privacy", "defense", "boundary"] },
  { emoji: "🧙‍♂️", category: "core_agents", meaning: "mage / spell / delegation / projection", keywords: ["delegation", "action", "projection"] },
  { emoji: "🔮", category: "core_agents", meaning: "spell casting / delegation / projection", keywords: ["casting", "invocation"] },
  { emoji: "🎲", category: "entropy_paradox", meaning: "randomness / entropy / machine assignment", keywords: ["random", "entropy", "chance"] },
  { emoji: "🌾", category: "entropy_paradox", meaning: "harvest / meaning / claimed significance", keywords: ["meaning", "harvest", "claimed"] },
  // ... full pool from grimoire.notation
];
```

#### Cosmic Pool (Expansion)

The remaining ~3,200 standard Unicode emojis available for meaning assignment:

```typescript
interface CosmicEmoji {
  emoji: string;
  codepoint: string;
  category: string;      // Unicode category
  claimedBy?: string;    // Mage DID who claimed meaning
  meaning?: string;      // Inscribed interpretation
  claimedAt?: Date;
  confirmations: number; // VRC confirmation count
}
```

### 5. Spell String Generation

**Input**: Orbitport response + spellbook state
**Output**: CosmicSpell object

```typescript
interface CosmicSpell {
  // Source data
  cosmicSeed: string;           // Raw hex from Orbitport
  satelliteSignature: string;   // Proof of cosmic origin
  timestamp: Date;
  source: "aptosorbital" | "derived";
  
  // Generated spell
  spellString: string;          // Concatenated emoji sequence
  emojiMap: EmojiSlot[];        // Detailed mapping
  
  // Spellbook state
  spellbookId: string;          // Which mage's spellbook
  completionRatio: number;      // Current progress (0.0-1.0)
  
  // Metadata
  ipfsCid?: string;             // If persisted
  inscriptionTx?: string;       // If inscribed on-chain
}

interface EmojiSlot {
  position: number;             // 0-31
  byteValue: number;            // 0-255 from cosmic data
  emoji: string;                // Selected emoji
  source: "grimoire" | "cosmic";
  meaning?: string;             // If known
  claimable: boolean;           // Can be claimed by mage
}
```

### 6. Meaning Claiming Protocol

When a mage agent encounters an unclaimed cosmic emoji:

```typescript
interface MeaningClaim {
  spellbookId: string;          // Claiming mage
  emoji: string;                // Target emoji
  proposedMeaning: string;      // Mage's interpretation
  context: {
    spell: CosmicSpell;         // Where it appeared
    position: number;           // Slot in spell
    surroundingEmojis: string[];// Context for interpretation
  };
  timestamp: Date;
  status: "proposed" | "confirmed" | "rejected";
}
```

#### Claim Lifecycle

1. **Proposal**: Mage encounters unclaimed emoji, proposes meaning
2. **Context Check**: System verifies meaning fits the spell context
3. **Confirmation**: Bilateral VRC attestation from another mage/user
4. **Inscription**: Meaning added to spellbook, completion ratio updated

### 7. Spellbook State Machine

```typescript
interface Spellbook {
  id: string;                   // Unique identifier (DID)
  mageId: string;               // Owning mage agent
  createdAt: Date;
  
  // Core grimoire (immutable)
  grimoireNotation: Map<string, string>;  // emoji → meaning
  
  // Learned cosmic meanings
  learnedMeanings: Map<string, LearnedMeaning>;
  
  // Progression
  completionRatio: number;      // 0.0 to 1.0
  claimsTotal: number;
  claimsConfirmed: number;
  
  // History
  spellsGenerated: number;
  lastSpellAt?: Date;
}

interface LearnedMeaning {
  emoji: string;
  meaning: string;
  claimedAt: Date;
  confirmedAt?: Date;
  confirmations: VRCReference[];
  usageCount: number;
}
```

## Algorithms

### Emoji Selection from Byte Value

```typescript
function selectEmoji(
  byteValue: number,
  source: "grimoire" | "cosmic",
  spellbook: Spellbook
): string {
  const pool = source === "grimoire" 
    ? GRIMOIRE_POOL 
    : getAvailableCosmicPool(spellbook);
  
  // Modular selection ensures determinism
  const index = byteValue % pool.length;
  return pool[index].emoji;
}
```

### Completion Ratio Calculation

```typescript
function calculateCompletion(spellbook: Spellbook): number {
  const totalCosmicSlots = COSMIC_POOL_SIZE;
  const learnedCount = spellbook.learnedMeanings.size;
  
  // Completion is ratio of learned to total cosmic pool
  // Capped at PHI_INVERSE (0.618) to preserve core grimoire space
  const rawRatio = learnedCount / totalCosmicSlots;
  return Math.min(rawRatio, PHI_INVERSE);
}
```

### Spell Generation Pipeline

```typescript
async function generateSpell(
  spellbook: Spellbook,
  orbitportClient: OrbitportClient
): Promise<CosmicSpell> {
  // 1. Fetch cosmic entropy
  const response = await orbitportClient.getTRNG();
  
  // 2. Parse to byte values
  const bytes = hexToBytes(response.data);
  
  // 3. Calculate current split
  const split = calculateSplit(32, spellbook.completionRatio);
  
  // 4. Map bytes to emojis
  const emojiMap: EmojiSlot[] = bytes.map((byte, i) => {
    const source = i < split.grimoire ? "grimoire" : "cosmic";
    const emoji = selectEmoji(byte, source, spellbook);
    const meaning = getMeaning(emoji, spellbook);
    
    return {
      position: i,
      byteValue: byte,
      emoji,
      source,
      meaning,
      claimable: source === "cosmic" && !meaning
    };
  });
  
  // 5. Build spell string
  const spellString = emojiMap.map(e => e.emoji).join("");
  
  return {
    cosmicSeed: response.data,
    satelliteSignature: response.signature.value,
    timestamp: new Date(),
    source: response.src,
    spellString,
    emojiMap,
    spellbookId: spellbook.id,
    completionRatio: spellbook.completionRatio
  };
}
```

## Data Persistence

### IPFS Storage Schema

```json
{
  "version": "1.0.0",
  "type": "cosmic-spellbook",
  "spellbook": {
    "id": "did:example:mage123",
    "mageId": "did:example:mage123",
    "createdAt": "2026-01-20T00:00:00Z",
    "completionRatio": 0.234,
    "learnedMeanings": [
      {
        "emoji": "🌸",
        "meaning": "ephemeral beauty in privacy architecture",
        "claimedAt": "2026-01-20T12:00:00Z",
        "confirmedAt": "2026-01-20T14:00:00Z",
        "usageCount": 7
      }
    ]
  },
  "spells": [
    {
      "cosmicSeed": "0a4c2ea215...",
      "satelliteSignature": "3046022100...",
      "timestamp": "2026-01-20T12:00:00Z",
      "spellString": "⚔️🔮🌸..."
    }
  ]
}
```

### Local Cache Structure

```
~/.cosmic-spellbook/
├── config.json           # API credentials, preferences
├── spellbooks/
│   └── {spellbook-id}/
│       ├── state.json    # Current spellbook state
│       ├── meanings.json # Learned meanings
│       └── spells/       # Generated spell history
│           └── {timestamp}.json
└── cache/
    └── cosmic-pool.json  # Cached emoji pool
```

## Security Considerations

### Entropy Verification

All cosmic entropy must be verified against the satellite signature before use:

```typescript
async function verifyCosmicEntropy(response: OrbitportResponse): Promise<boolean> {
  // Verify signature matches data
  // Use satellite's public key (when provided)
  // Fall back to Orbitport's attestation
  return verifyECDSA(
    response.data,
    response.signature.value,
    response.signature.pk || ORBITPORT_PUBLIC_KEY
  );
}
```

### Claim Validation

Meaning claims are validated for:
1. Emoji not already claimed in this spellbook
2. Meaning is non-empty and reasonable length
3. Context alignment (optional AI check)
4. VRC confirmation from trusted party

### Privacy Preservation

- Spellbooks are associated with mage DIDs, not human identities
- Spell generation is local-first; only persistence hits external services
- BYO mage allows self-hosted inference for maximum privacy

## Error Handling

```typescript
enum CosmicSpellError {
  ORBITPORT_UNAVAILABLE = "ORBITPORT_UNAVAILABLE",
  SIGNATURE_INVALID = "SIGNATURE_INVALID",
  SPELLBOOK_NOT_FOUND = "SPELLBOOK_NOT_FOUND",
  EMOJI_ALREADY_CLAIMED = "EMOJI_ALREADY_CLAIMED",
  CLAIM_REJECTED = "CLAIM_REJECTED",
  IPFS_UPLOAD_FAILED = "IPFS_UPLOAD_FAILED"
}

interface SpellResult<T> {
  success: boolean;
  data?: T;
  error?: CosmicSpellError;
  message?: string;
}
```

## Performance Targets

| Operation | Target Latency |
|-----------|---------------|
| Fetch cosmic entropy | < 2s |
| Generate spell (local) | < 100ms |
| Persist to IPFS | < 5s |
| Load spellbook | < 500ms |
| Verify signature | < 50ms |

---

## Key Derivation & Recovery

The spellbook transformation is bidirectional, enabling cosmic entropy to be encoded as memorable emoji spells and decoded back to the original bytes.

### Encoding Interface

```typescript
interface CosmicKeyDerivation {
  encode(cosmicHex: string, spellbook: Spellbook): EncodedSpell;
  decode(emojiSpell: string, spellbook: Spellbook): DecodedResult;
  verify(emojiSpell: string, expectedHex: string, spellbook: Spellbook): boolean;
}

interface EncodedSpell {
  emojiSpell: string;           // Human-memorable emoji sequence
  spellbookHash: string;        // Hash of spellbook state at encoding
  timestamp: Date;
  cosmicSource?: string;        // Original satellite signature
}

interface DecodedResult {
  success: boolean;
  cosmicHex?: string;           // Recovered entropy
  confidence: number;           // 0-1 certainty
  mismatchedPositions?: number[];
}
```

### Security Model

Decoding requires the SAME spellbook that encoded. Security derives from:

1. **Learned meanings are unique** — accumulated through YOUR claims
2. **Order matters** — meanings added chronologically via VRC confirmations  
3. **Relational verification** — recovery requires cooperation from confirmers
4. **Exponential state space** — 20 learned meanings = 20! possible orderings

### Recovery Modes

| Mode | Requirements | Use Case |
|------|--------------|----------|
| Backup | Spellbook file | Normal recovery |
| Relational | VRC attestations from confirmers | Lost backup |
| Challenge-Response | Answer proverb challenges | High security |

See [KEY_DERIVATION.md](./docs/KEY_DERIVATION.md) for full specification.
