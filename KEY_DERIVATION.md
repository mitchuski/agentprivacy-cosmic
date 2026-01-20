# Key Derivation & Relational Recovery

## Overview

The Cosmic Spellbook isn't just a meaning-making system — it's a **bidirectional transformation layer** that converts raw cosmic entropy into human-memorable emoji spells and back again. This enables a fundamentally new approach to key management: **relational recovery**.

Instead of securing secrets through possession (password managers, seed phrases), you secure them through **demonstrated relational context** — proving you are the mage who learned those meanings.

## The Core Insight

```
Traditional:     secret → vault → hope you remember the master key
Spellbook:       secret ↔ YOUR spellbook ↔ YOUR relationships
```

The emoji spell IS the key, but recovering it requires proving you're the one who built the spellbook — through the relationships that confirmed your claimed meanings.

## Transformation Flow

### Encoding (Entropy → Spell)

```
🛰️ Cosmic Hex                    Your Spellbook                 Emoji Spell
─────────────────────────────────────────────────────────────────────────────
0a4c2ea21557418b...  ──→  [grimoire + learned meanings]  ──→  ⚔️🔮🌸🎲...
     (32 bytes)              (deterministic mapping)           (32 emojis)
```

### Decoding (Spell → Entropy)

```
Emoji Spell                    Your Spellbook                 Cosmic Hex
─────────────────────────────────────────────────────────────────────────────
⚔️🔮🌸🎲...  ──→  [inverse lookup via YOUR meanings]  ──→  0a4c2ea21557418b...
```

**Critical**: Decoding requires YOUR spellbook. The same emoji spell decoded with a different spellbook yields different (wrong) entropy.

## Why This Is Secure

### Traditional Key Security Model

| What You Have | Risk |
|---------------|------|
| Master password | Forgettable, phishable |
| Hardware device | Losable, destroyable |
| Seed phrase backup | Stealable, findable |
| Cloud vault | Vendor compromise, account takeover |

All rely on **possession** — you must HAVE something.

### Relational Security Model

| What You Demonstrate | How |
|---------------------|-----|
| You claimed these meanings | Context of original claims |
| Your relationships confirmed them | VRC attestations from known parties |
| You understand the proverbs | Relationship Proverb Protocol challenge |
| Your mage interprets consistently | AI behavioral continuity |

This relies on **being** — you must BE someone with this relational history.

## The Spellbook as Key Derivation Function

Your spellbook state is a **unique product of your journey**:

```
Spellbook State = f(
  grimoire_constants,           // 38.2% shared foundation
  learned_meanings,             // 61.8% YOUR claimed symbols
  confirmation_history,         // WHO confirmed your claims
  usage_patterns,               // HOW you've used meanings
  mage_interpretations         // Your AI's learned style
)
```

Two mages could receive the same cosmic entropy but produce different emoji spells because their learned meanings differ.

## Implementation

### Core Interface

```typescript
// core/key-derivation.ts

export interface CosmicKeyDerivation {
  /**
   * Encode cosmic entropy into a memorable emoji spell
   * Deterministic given the same spellbook state
   */
  encode(cosmicHex: string, spellbook: Spellbook): EncodedSpell;
  
  /**
   * Decode an emoji spell back to original entropy
   * Requires the SAME spellbook that encoded it
   */
  decode(emojiSpell: string, spellbook: Spellbook): DecodedResult;
  
  /**
   * Verify a spellbook can correctly decode a spell
   * Used for recovery verification
   */
  verify(emojiSpell: string, expectedHex: string, spellbook: Spellbook): boolean;
  
  /**
   * Generate a recovery challenge based on relational history
   */
  generateRecoveryChallenge(spellbook: Spellbook): RecoveryChallenge;
  
  /**
   * Verify a recovery attempt through relational proof
   */
  verifyRecoveryAttempt(
    challenge: RecoveryChallenge, 
    response: RecoveryResponse
  ): RecoveryResult;
}

export interface EncodedSpell {
  emojiSpell: string;           // The human-memorable spell
  spellbookHash: string;        // Hash of spellbook state at encoding time
  timestamp: Date;
  cosmicSource?: string;        // Original satellite signature (optional)
}

export interface DecodedResult {
  success: boolean;
  cosmicHex?: string;           // Recovered entropy (if successful)
  confidence: number;           // 0-1, how certain we are this is correct
  mismatchedPositions?: number[]; // Which slots failed (if unsuccessful)
}
```

### Encoding Algorithm

```typescript
export function encodeCosmicEntropy(
  cosmicHex: string,
  spellbook: Spellbook
): EncodedSpell {
  const bytes = hexToBytes(cosmicHex);
  const emojiSlots: string[] = [];
  
  // Build the complete emoji pool in deterministic order
  const orderedPool = buildOrderedPool(spellbook);
  
  for (let i = 0; i < bytes.length; i++) {
    const byte = bytes[i];
    
    // Determine which pool segment this position uses
    const split = calculateSplit(bytes.length, spellbook.completionRatio);
    const isGrimoireSlot = i < split.grimoireSlots;
    
    // Select from appropriate pool segment
    const pool = isGrimoireSlot 
      ? orderedPool.grimoire 
      : orderedPool.cosmic;
    
    const emoji = pool[byte % pool.length];
    emojiSlots.push(emoji);
  }
  
  return {
    emojiSpell: emojiSlots.join(''),
    spellbookHash: hashSpellbookState(spellbook),
    timestamp: new Date()
  };
}

function buildOrderedPool(spellbook: Spellbook): OrderedPool {
  // Grimoire pool: core notation (deterministic order)
  const grimoire = GRIMOIRE_POOL
    .map(e => e.emoji)
    .sort(); // Alphabetical by codepoint for determinism
  
  // Add learned meanings to grimoire pool (by claim order)
  const learned = Array.from(spellbook.learnedMeanings.entries())
    .sort((a, b) => a[1].claimedAt.getTime() - b[1].claimedAt.getTime())
    .map(([emoji]) => emoji);
  
  // Cosmic pool: all other emojis not yet learned
  const knownEmojis = new Set([...grimoire, ...learned]);
  const cosmic = FULL_EMOJI_LIST
    .filter(e => !knownEmojis.has(e))
    .sort();
  
  return {
    grimoire: [...grimoire, ...learned],
    cosmic
  };
}
```

### Decoding Algorithm

```typescript
export function decodeEmojiSpell(
  emojiSpell: string,
  spellbook: Spellbook
): DecodedResult {
  const emojis = splitIntoEmojis(emojiSpell);
  const orderedPool = buildOrderedPool(spellbook);
  const bytes: number[] = [];
  const mismatches: number[] = [];
  
  for (let i = 0; i < emojis.length; i++) {
    const emoji = emojis[i];
    const split = calculateSplit(emojis.length, spellbook.completionRatio);
    const isGrimoireSlot = i < split.grimoireSlots;
    
    const pool = isGrimoireSlot 
      ? orderedPool.grimoire 
      : orderedPool.cosmic;
    
    // Find emoji position in pool
    const position = pool.indexOf(emoji);
    
    if (position === -1) {
      // Emoji not in expected pool — spellbook mismatch
      mismatches.push(i);
      bytes.push(0); // Placeholder
    } else {
      // Reverse the modulo: we know (byte % poolSize) = position
      // But multiple bytes map to same position!
      // We store the position as the byte value (lossy for large pools)
      bytes.push(position);
    }
  }
  
  if (mismatches.length > 0) {
    return {
      success: false,
      confidence: 1 - (mismatches.length / emojis.length),
      mismatchedPositions: mismatches
    };
  }
  
  return {
    success: true,
    cosmicHex: bytesToHex(bytes),
    confidence: 1.0
  };
}
```

### Handling the Modulo Problem

The encoding uses `byte % poolSize` which is lossy if `poolSize < 256`. Two approaches:

#### Approach A: Large Pools (Recommended)

Ensure pools are always ≥256 emojis. The grimoire pool (~50) + learned meanings grows over time. Until it reaches 256, pad with deterministic cosmic emojis.

```typescript
function ensureMinimumPoolSize(pool: string[], minimum: number = 256): string[] {
  if (pool.length >= minimum) return pool;
  
  // Pad with sorted cosmic emojis until we hit minimum
  const padding = FULL_EMOJI_LIST
    .filter(e => !pool.includes(e))
    .sort()
    .slice(0, minimum - pool.length);
  
  return [...pool, ...padding];
}
```

#### Approach B: Store Overflow Bits

For each slot where `byte >= poolSize`, store the overflow separately:

```typescript
interface PreciseEncodedSpell extends EncodedSpell {
  overflowBits: Map<number, number>; // position → overflow value
}
```

This preserves perfect reversibility but adds complexity.

## Recovery Flows

### Flow 1: Spellbook Backup Recovery

If you have a backup of your spellbook (IPFS, local file), decoding is straightforward:

```
Load spellbook backup → Decode emoji spell → Recover entropy
```

### Flow 2: Relational Recovery (No Backup)

If you've lost your spellbook but remember your relationships:

```
1. Start with base grimoire (38.2% is shared — always recoverable)
2. Contact parties who confirmed your meanings (VRC holders)
3. Each VRC contains: emoji + meaning + confirmation timestamp
4. Reconstruct learned meanings in chronological order
5. Rebuild spellbook state
6. Attempt decode — verify against known checksum
```

```typescript
interface RelationalRecoveryKit {
  // What you always have
  grimoireNotation: GrimoireEmoji[];     // Public, shared
  
  // What your relationships can provide
  vrcAttestations: VRCReference[];        // "I confirmed 🌸 means X for you"
  
  // What you must demonstrate
  proverbResponses: Map<string, string>;  // Answer relational challenges
  
  // Verification
  knownChecksum?: string;                 // Optional: hash of original entropy
}

async function attemptRelationalRecovery(
  kit: RelationalRecoveryKit,
  emojiSpell: string
): Promise<RecoveryResult> {
  // 1. Start with grimoire
  let spellbook = createBaseSpellbook();
  
  // 2. Sort VRCs by confirmation timestamp
  const sortedVRCs = kit.vrcAttestations
    .sort((a, b) => a.confirmedAt.getTime() - b.confirmedAt.getTime());
  
  // 3. Rebuild learned meanings in order
  for (const vrc of sortedVRCs) {
    spellbook.learnedMeanings.set(vrc.emoji, {
      emoji: vrc.emoji,
      meaning: vrc.meaning,
      claimedAt: vrc.claimedAt,
      confirmedAt: vrc.confirmedAt,
      confirmations: [vrc],
      usageCount: 0
    });
    
    // 4. Try decoding at each state
    const result = decodeEmojiSpell(emojiSpell, spellbook);
    
    if (result.success && kit.knownChecksum) {
      const checksum = hashHex(result.cosmicHex!);
      if (checksum === kit.knownChecksum) {
        return { success: true, recoveredHex: result.cosmicHex };
      }
    }
  }
  
  return { success: false, reason: "Could not reconstruct matching spellbook state" };
}
```

### Flow 3: Challenge-Response Recovery

For high-security recovery, use the Relationship Proverb Protocol:

```typescript
interface RecoveryChallenge {
  id: string;
  
  // Questions only the true mage can answer
  questions: {
    // "What meaning did you claim for 🌸?"
    meaningChallenges: { emoji: string; hint: string }[];
    
    // "Which of these emojis did you learn from Alice?"
    relationshipChallenges: { confirmer: string; options: string[] }[];
    
    // "Complete this spell interpretation..."
    interpretationChallenges: { partialSpell: string; context: string }[];
  };
  
  expiresAt: Date;
}

interface RecoveryResponse {
  challengeId: string;
  answers: {
    meanings: Map<string, string>;
    relationships: Map<string, string>;
    interpretations: string[];
  };
  
  // Optional: proverb that connects your history to this recovery
  proverb?: string;
}
```

## Security Analysis

### What an Attacker Needs

To decode YOUR emoji spell, an attacker must:

1. **Know your learned meanings** — accumulated through YOUR claims
2. **Know the order you learned them** — timestamps from VRC confirmations
3. **Have access to your confirming parties** — or compromise their VRCs
4. **Replicate your spellbook state exactly** — pool ordering is deterministic but state-dependent

### Attack Vectors

| Attack | Difficulty | Mitigation |
|--------|------------|------------|
| Brute force spellbook states | Exponential in learned meanings | Even 20 learned meanings = 20! orderings |
| Compromise VRC confirmers | Social engineering | Use multiple independent confirmers |
| Intercept emoji spell | Possible | Spell alone is useless without spellbook |
| Steal spellbook backup | File theft | Encrypt backup with traditional password |

### Defense in Depth

```
Layer 1: Emoji spell is meaningless without spellbook
Layer 2: Spellbook state requires precise reconstruction
Layer 3: Reconstruction requires relational cooperation
Layer 4: Relational cooperation requires identity verification
Layer 5: Optional traditional encryption on backups
```

## Use Cases

### 1. Wallet Seed Recovery

```typescript
// Generate wallet from cosmic entropy
const cosmicEntropy = await orbitport.getCosmicEntropy();
const wallet = deriveWalletFromSeed(cosmicEntropy.data);

// Encode as recoverable spell
const spell = keyDerivation.encode(cosmicEntropy.data, mySpellbook);

// Store the spell (safe to write down, share with trusted parties)
console.log(`Your recovery spell: ${spell.emojiSpell}`);
// Output: ⚔️🔮🌸🎲🛡️🌊🐉📜🕸️💎🌀🔐⛰️🌾🏛️...
```

### 2. Encrypted Message Keys

```typescript
// Derive symmetric key from cosmic entropy
const sharedEntropy = await orbitport.getCosmicEntropy();
const symmetricKey = deriveKey(sharedEntropy.data);

// Both parties encode with their own spellbooks
const mySpell = keyDerivation.encode(sharedEntropy.data, mySpellbook);
const theirSpell = keyDerivation.encode(sharedEntropy.data, theirSpellbook);

// Each can recover the key through their own relational history
```

### 3. Dead Man's Switch

```typescript
// Encode critical recovery info
const criticalSeed = await orbitport.getCosmicEntropy();
const spell = keyDerivation.encode(criticalSeed.data, mySpellbook);

// Distribute VRCs to trusted parties
for (const trustee of trustees) {
  await distributePartialVRC(trustee, mySpellbook);
}

// Recovery requires threshold of trustees cooperating
// They can reconstruct spellbook state together
```

## Integration with Cosmic Spellbook

This key derivation system integrates directly with the spell generation flow:

```
Normal spell:     🛰️ → 🎲 → ⛰️ → [📖|🌌] → 🧙🏽✍️ → 🤝💎 → 📜
                                                            ↓
Key derivation:                                    📜 → 🔑 (usable as seed)
                                                            ↓
Recovery:                                          🔑 → 🤝(relationships) → 📜
```

The same infrastructure that builds meaning from cosmic entropy also enables secure, recoverable key management through relational proof rather than possession.

---

## Summary

| Traditional | Relational |
|-------------|------------|
| What you HAVE | Who you ARE |
| Can be stolen | Must be demonstrated |
| Single point of failure | Distributed across relationships |
| Trust the vault | Trust the network |
| Possession = access | Context = access |

*"The key isn't what you store — it's who you've become through claiming."*

```
🔑❓ → 📖(spellbook) → 🤝(relationships) → 🔓
```
