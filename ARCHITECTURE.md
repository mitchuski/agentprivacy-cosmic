# System Architecture

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              User Interface Layer                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐ │
│  │ Web Client   │  │ CLI Tool     │  │ Browser Ext  │  │ API Consumers    │ │
│  │ (React/Next) │  │ (Node.js)    │  │ (Swordsman)  │  │ (Third-party)    │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            Application Layer                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                        Cosmic Spellbook Engine                          ││
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────────────┐││
│  │  │ Spell         │  │ Spellbook     │  │ Claiming                      │││
│  │  │ Generator     │  │ Manager       │  │ Protocol                      │││
│  │  └───────────────┘  └───────────────┘  └───────────────────────────────┘││
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────────────┐││
│  │  │ φ-Splitter    │  │ Emoji Mapper  │  │ Progression                   │││
│  │  │               │  │               │  │ Tracker                       │││
│  │  └───────────────┘  └───────────────┘  └───────────────────────────────┘││
│  │  ┌───────────────┐  ┌───────────────────────────────────────────────────┐││
│  │  │ Key           │  │ Recovery Protocol                                 │││
│  │  │ Derivation    │  │ (Relational Proof)                                │││
│  │  └───────────────┘  └───────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    ▼                  ▼                  ▼
┌──────────────────────────┐ ┌────────────────┐ ┌────────────────────────────┐
│   External Services      │ │  Mage Layer    │ │   Storage Layer            │
│  ┌────────────────────┐  │ │ ┌────────────┐ │ │  ┌──────────────────────┐  │
│  │ SpaceComputer      │  │ │ │ BYO LLM    │ │ │  │ Local Filesystem     │  │
│  │ Orbitport (cTRNG)  │  │ │ │ (OpenAI,   │ │ │  │ (~/.cosmic-spellbook)│  │
│  └────────────────────┘  │ │ │ Anthropic, │ │ │  └──────────────────────┘  │
│  ┌────────────────────┐  │ │ │ Ollama)    │ │ │  ┌──────────────────────┐  │
│  │ IPFS/Pinata        │  │ │ └────────────┘ │ │  │ IPFS (Pinata)        │  │
│  │ (Persistence)      │  │ │ ┌────────────┐ │ │  │ (Distributed)        │  │
│  └────────────────────┘  │ │ │ NEAR AI    │ │ │  └──────────────────────┘  │
│  ┌────────────────────┐  │ │ │ (Private   │ │ │  ┌──────────────────────┐  │
│  │ First Person       │  │ │ │ Inference) │ │ │  │ Zcash                │  │
│  │ Network (VRCs)     │  │ │ └────────────┘ │ │  │ (Inscriptions)       │  │
│  └────────────────────┘  │ └────────────────┘ │  └──────────────────────┘  │
└──────────────────────────┘                    └────────────────────────────┘
```

## Component Details

### 1. Spell Generator

**Purpose**: Transform cosmic entropy into emoji spell strings

```typescript
// core/spell-generator.ts

export class SpellGenerator {
  private orbitport: OrbitportClient;
  private emojiMapper: EmojiMapper;
  private phiSplitter: PhiSplitter;
  
  constructor(config: SpellGeneratorConfig) {
    this.orbitport = config.orbitport;
    this.emojiMapper = new EmojiMapper(config.grimoirePool, config.cosmicPool);
    this.phiSplitter = new PhiSplitter();
  }
  
  async generate(spellbook: Spellbook): Promise<CosmicSpell> {
    // 1. Fetch entropy
    const entropy = await this.orbitport.getCosmicEntropy();
    
    // 2. Calculate split based on completion
    const split = this.phiSplitter.calculate(32, spellbook.completionRatio);
    
    // 3. Map bytes to emojis
    const bytes = hexToBytes(entropy.data);
    const emojiMap = this.emojiMapper.mapBytes(bytes, split, spellbook);
    
    // 4. Assemble spell
    return this.assembleSpell(entropy, emojiMap, spellbook);
  }
}
```

### 2. Phi Splitter (φ-Splitter)

**Purpose**: Calculate golden ratio splits for grimoire vs cosmic allocation

```typescript
// core/phi-splitter.ts

export class PhiSplitter {
  private readonly PHI = 1.618033988749895;
  private readonly PHI_INVERSE = 0.618033988749895;
  
  calculate(totalSlots: number, completionRatio: number): SplitResult {
    // Birth: 38.2% grimoire, 61.8% cosmic
    // Complete: 61.8% grimoire (learned), 38.2% cosmic (core)
    
    const birthGrimoire = 1 - this.PHI_INVERSE;  // 0.382
    const completeGrimoire = this.PHI_INVERSE;    // 0.618
    
    // Linear interpolation based on completion
    const grimoireRatio = birthGrimoire + 
      (completionRatio * (completeGrimoire - birthGrimoire));
    
    const grimoireSlots = Math.round(totalSlots * grimoireRatio);
    
    return {
      grimoireSlots,
      cosmicSlots: totalSlots - grimoireSlots,
      grimoireRatio,
      cosmicRatio: 1 - grimoireRatio
    };
  }
}

interface SplitResult {
  grimoireSlots: number;
  cosmicSlots: number;
  grimoireRatio: number;
  cosmicRatio: number;
}
```

### 3. Emoji Mapper

**Purpose**: Map byte values to emojis from appropriate pools

```typescript
// core/emoji-mapper.ts

export class EmojiMapper {
  private grimoirePool: GrimoireEmoji[];
  private cosmicPool: CosmicEmoji[];
  
  constructor(grimoire: GrimoireEmoji[], cosmic: CosmicEmoji[]) {
    this.grimoirePool = grimoire;
    this.cosmicPool = cosmic;
  }
  
  mapBytes(
    bytes: number[], 
    split: SplitResult, 
    spellbook: Spellbook
  ): EmojiSlot[] {
    return bytes.map((byte, position) => {
      const isGrimoire = position < split.grimoireSlots;
      
      if (isGrimoire) {
        return this.mapToGrimoire(byte, position, spellbook);
      } else {
        return this.mapToCosmic(byte, position, spellbook);
      }
    });
  }
  
  private mapToGrimoire(
    byte: number, 
    position: number,
    spellbook: Spellbook
  ): EmojiSlot {
    // For grimoire slots, include both core AND learned meanings
    const availablePool = [
      ...this.grimoirePool,
      ...Array.from(spellbook.learnedMeanings.values())
    ];
    
    const index = byte % availablePool.length;
    const selected = availablePool[index];
    
    return {
      position,
      byteValue: byte,
      emoji: selected.emoji,
      source: "grimoire",
      meaning: selected.meaning,
      claimable: false
    };
  }
  
  private mapToCosmic(
    byte: number, 
    position: number,
    spellbook: Spellbook
  ): EmojiSlot {
    // For cosmic slots, use unclaimed emojis
    const unclaimedPool = this.cosmicPool.filter(
      e => !spellbook.learnedMeanings.has(e.emoji)
    );
    
    const index = byte % unclaimedPool.length;
    const selected = unclaimedPool[index];
    
    // Check if already claimed (would move to grimoire pool)
    const existingMeaning = spellbook.learnedMeanings.get(selected.emoji);
    
    return {
      position,
      byteValue: byte,
      emoji: selected.emoji,
      source: "cosmic",
      meaning: existingMeaning?.meaning,
      claimable: !existingMeaning
    };
  }
}
```

### 4. Spellbook Manager

**Purpose**: Manage spellbook state, persistence, and lifecycle

```typescript
// core/spellbook-manager.ts

export class SpellbookManager {
  private storage: StorageAdapter;
  
  constructor(storage: StorageAdapter) {
    this.storage = storage;
  }
  
  async create(mageId: string): Promise<Spellbook> {
    const spellbook: Spellbook = {
      id: generateSpellbookId(),
      mageId,
      createdAt: new Date(),
      grimoireNotation: new Map(GRIMOIRE_POOL.map(e => [e.emoji, e.meaning])),
      learnedMeanings: new Map(),
      completionRatio: 0,
      claimsTotal: 0,
      claimsConfirmed: 0,
      spellsGenerated: 0
    };
    
    await this.storage.save(spellbook);
    return spellbook;
  }
  
  async load(id: string): Promise<Spellbook | null> {
    return this.storage.load(id);
  }
  
  async updateCompletion(spellbook: Spellbook): Promise<void> {
    const totalCosmicCapacity = COSMIC_POOL_SIZE * PHI_INVERSE; // ~1,977 emojis
    const learnedCount = spellbook.learnedMeanings.size;
    
    spellbook.completionRatio = Math.min(
      learnedCount / totalCosmicCapacity,
      PHI_INVERSE // Cap at 0.618
    );
    
    await this.storage.save(spellbook);
  }
  
  async recordSpell(spellbook: Spellbook, spell: CosmicSpell): Promise<void> {
    spellbook.spellsGenerated++;
    spellbook.lastSpellAt = spell.timestamp;
    await this.storage.save(spellbook);
  }
}
```

### 5. Claiming Protocol

**Purpose**: Handle meaning claims, validation, and confirmation

```typescript
// core/claiming-protocol.ts

export class ClaimingProtocol {
  private spellbookManager: SpellbookManager;
  private vrcClient?: VRCClient;
  
  async proposeClaim(claim: MeaningClaimProposal): Promise<MeaningClaim> {
    const spellbook = await this.spellbookManager.load(claim.spellbookId);
    if (!spellbook) {
      throw new Error("Spellbook not found");
    }
    
    // Validate claim
    this.validateClaim(claim, spellbook);
    
    // Create pending claim
    const pendingClaim: MeaningClaim = {
      ...claim,
      id: generateClaimId(),
      status: "proposed",
      createdAt: new Date()
    };
    
    // Update spellbook
    spellbook.claimsTotal++;
    await this.spellbookManager.save(spellbook);
    
    return pendingClaim;
  }
  
  async confirmClaim(claimId: string, vrc: VRCReference): Promise<MeaningClaim> {
    const claim = await this.loadClaim(claimId);
    const spellbook = await this.spellbookManager.load(claim.spellbookId);
    
    // Add to learned meanings
    const learnedMeaning: LearnedMeaning = {
      emoji: claim.emoji,
      meaning: claim.proposedMeaning,
      claimedAt: claim.createdAt,
      confirmedAt: new Date(),
      confirmations: [vrc],
      usageCount: 0
    };
    
    spellbook.learnedMeanings.set(claim.emoji, learnedMeaning);
    spellbook.claimsConfirmed++;
    
    // Update completion ratio
    await this.spellbookManager.updateCompletion(spellbook);
    
    claim.status = "confirmed";
    return claim;
  }
  
  private validateClaim(claim: MeaningClaimProposal, spellbook: Spellbook): void {
    // Already claimed?
    if (spellbook.learnedMeanings.has(claim.emoji)) {
      throw new ClaimError("EMOJI_ALREADY_CLAIMED", claim.emoji);
    }
    
    // Is it a grimoire emoji? (Can't claim those)
    if (spellbook.grimoireNotation.has(claim.emoji)) {
      throw new ClaimError("GRIMOIRE_EMOJI", claim.emoji);
    }
    
    // Meaning validation
    if (!claim.proposedMeaning || claim.proposedMeaning.length < 3) {
      throw new ClaimError("INVALID_MEANING", "Meaning too short");
    }
    
    if (claim.proposedMeaning.length > 500) {
      throw new ClaimError("INVALID_MEANING", "Meaning too long");
    }
  }
}
```

### 6. Key Derivation

**Purpose**: Bidirectional transformation between cosmic entropy and emoji spells for recoverable key management

```typescript
// core/key-derivation.ts

export class KeyDerivation {
  private emojiMapper: EmojiMapper;
  
  /**
   * Encode cosmic entropy as a memorable emoji spell
   * Deterministic given the same spellbook state
   */
  encode(cosmicHex: string, spellbook: Spellbook): EncodedSpell {
    const bytes = hexToBytes(cosmicHex);
    const orderedPool = this.buildOrderedPool(spellbook);
    const emojiSlots: string[] = [];
    
    for (let i = 0; i < bytes.length; i++) {
      const byte = bytes[i];
      const split = calculateSplit(bytes.length, spellbook.completionRatio);
      const pool = i < split.grimoireSlots 
        ? orderedPool.grimoire 
        : orderedPool.cosmic;
      
      emojiSlots.push(pool[byte % pool.length]);
    }
    
    return {
      emojiSpell: emojiSlots.join(''),
      spellbookHash: hashSpellbookState(spellbook),
      timestamp: new Date()
    };
  }
  
  /**
   * Decode an emoji spell back to cosmic entropy
   * Requires the SAME spellbook that encoded it
   */
  decode(emojiSpell: string, spellbook: Spellbook): DecodedResult {
    const emojis = splitIntoEmojis(emojiSpell);
    const orderedPool = this.buildOrderedPool(spellbook);
    const bytes: number[] = [];
    const mismatches: number[] = [];
    
    for (let i = 0; i < emojis.length; i++) {
      const emoji = emojis[i];
      const split = calculateSplit(emojis.length, spellbook.completionRatio);
      const pool = i < split.grimoireSlots 
        ? orderedPool.grimoire 
        : orderedPool.cosmic;
      
      const position = pool.indexOf(emoji);
      
      if (position === -1) {
        mismatches.push(i);
        bytes.push(0);
      } else {
        bytes.push(position);
      }
    }
    
    return {
      success: mismatches.length === 0,
      cosmicHex: mismatches.length === 0 ? bytesToHex(bytes) : undefined,
      confidence: 1 - (mismatches.length / emojis.length),
      mismatchedPositions: mismatches.length > 0 ? mismatches : undefined
    };
  }
  
  /**
   * Build deterministically ordered emoji pools
   * Order: grimoire (sorted) + learned (by claim time) | cosmic (sorted)
   */
  private buildOrderedPool(spellbook: Spellbook): OrderedPool {
    const grimoire = GRIMOIRE_POOL.map(e => e.emoji).sort();
    
    const learned = Array.from(spellbook.learnedMeanings.entries())
      .sort((a, b) => a[1].claimedAt.getTime() - b[1].claimedAt.getTime())
      .map(([emoji]) => emoji);
    
    const knownEmojis = new Set([...grimoire, ...learned]);
    const cosmic = FULL_EMOJI_LIST.filter(e => !knownEmojis.has(e)).sort();
    
    return {
      grimoire: [...grimoire, ...learned],
      cosmic
    };
  }
}

interface EncodedSpell {
  emojiSpell: string;
  spellbookHash: string;
  timestamp: Date;
  cosmicSource?: string;
}

interface DecodedResult {
  success: boolean;
  cosmicHex?: string;
  confidence: number;
  mismatchedPositions?: number[];
}
```

### 7. Recovery Protocol

**Purpose**: Enable key recovery through relational proof rather than backup possession

```typescript
// core/recovery-protocol.ts

export class RecoveryProtocol {
  /**
   * Attempt recovery using VRC attestations from confirming parties
   */
  async attemptRelationalRecovery(
    kit: RelationalRecoveryKit,
    emojiSpell: string
  ): Promise<RecoveryResult> {
    // Start with base grimoire (always recoverable)
    let spellbook = createBaseSpellbook();
    
    // Sort VRCs chronologically
    const sortedVRCs = kit.vrcAttestations
      .sort((a, b) => a.confirmedAt.getTime() - b.confirmedAt.getTime());
    
    // Rebuild learned meanings in order
    for (const vrc of sortedVRCs) {
      spellbook.learnedMeanings.set(vrc.emoji, {
        emoji: vrc.emoji,
        meaning: vrc.meaning,
        claimedAt: vrc.claimedAt,
        confirmedAt: vrc.confirmedAt,
        confirmations: [vrc],
        usageCount: 0
      });
      
      // Try decoding at each state
      const result = this.keyDerivation.decode(emojiSpell, spellbook);
      
      if (result.success && kit.knownChecksum) {
        const checksum = hashHex(result.cosmicHex!);
        if (checksum === kit.knownChecksum) {
          return { success: true, recoveredHex: result.cosmicHex };
        }
      }
    }
    
    return { success: false, reason: "Could not reconstruct matching state" };
  }
  
  /**
   * Generate challenge-response for high-security recovery
   */
  generateRecoveryChallenge(spellbook: Spellbook): RecoveryChallenge {
    return {
      id: generateChallengeId(),
      questions: {
        meaningChallenges: this.selectMeaningChallenges(spellbook, 3),
        relationshipChallenges: this.selectRelationshipChallenges(spellbook, 2),
        interpretationChallenges: this.selectInterpretationChallenges(spellbook, 1)
      },
      expiresAt: new Date(Date.now() + 15 * 60 * 1000) // 15 minutes
    };
  }
}

interface RelationalRecoveryKit {
  grimoireNotation: GrimoireEmoji[];    // Always available (public)
  vrcAttestations: VRCReference[];       // From confirming parties
  proverbResponses?: Map<string, string>;
  knownChecksum?: string;
}

interface RecoveryChallenge {
  id: string;
  questions: {
    meaningChallenges: { emoji: string; hint: string }[];
    relationshipChallenges: { confirmer: string; options: string[] }[];
    interpretationChallenges: { partialSpell: string; context: string }[];
  };
  expiresAt: Date;
}
```

## Data Models

### Core Types

```typescript
// types/core.ts

export interface Spellbook {
  id: string;
  mageId: string;
  createdAt: Date;
  
  // Immutable core notation from Grimoire
  grimoireNotation: Map<string, string>;
  
  // Learned cosmic meanings
  learnedMeanings: Map<string, LearnedMeaning>;
  
  // Progression tracking
  completionRatio: number;
  claimsTotal: number;
  claimsConfirmed: number;
  
  // Activity
  spellsGenerated: number;
  lastSpellAt?: Date;
}

export interface CosmicSpell {
  id: string;
  cosmicSeed: string;
  satelliteSignature: string;
  timestamp: Date;
  source: "aptosorbital" | "derived";
  
  spellString: string;
  emojiMap: EmojiSlot[];
  
  spellbookId: string;
  completionRatio: number;
  
  ipfsCid?: string;
  inscriptionTx?: string;
}

export interface EmojiSlot {
  position: number;
  byteValue: number;
  emoji: string;
  source: "grimoire" | "cosmic";
  meaning?: string;
  claimable: boolean;
}

export interface LearnedMeaning {
  emoji: string;
  meaning: string;
  claimedAt: Date;
  confirmedAt?: Date;
  confirmations: VRCReference[];
  usageCount: number;
}

export interface MeaningClaim {
  id: string;
  spellbookId: string;
  emoji: string;
  proposedMeaning: string;
  context: ClaimContext;
  status: "proposed" | "confirmed" | "rejected";
  createdAt: Date;
  confirmedAt?: Date;
}

export interface ClaimContext {
  spellId: string;
  position: number;
  surroundingEmojis: string[];
  mageInterpretation?: string;
}
```

### Emoji Types

```typescript
// types/emoji.ts

export interface GrimoireEmoji {
  emoji: string;
  category: EmojiCategory;
  meaning: string;
  keywords: string[];
}

export interface CosmicEmoji {
  emoji: string;
  codepoint: string;
  unicodeCategory: string;
}

export type EmojiCategory = 
  | "core_agents"
  | "trust_infrastructure"
  | "topology"
  | "entropy_paradox"
  | "zkp"
  | "canon"
  | "plurality"
  | "transformations"
  | "dark_forest";
```

### Storage Types

```typescript
// types/storage.ts

export interface StorageAdapter {
  save(spellbook: Spellbook): Promise<void>;
  load(id: string): Promise<Spellbook | null>;
  list(mageId: string): Promise<SpellbookSummary[]>;
  delete(id: string): Promise<void>;
}

export interface SpellStorage {
  saveSpell(spell: CosmicSpell): Promise<string>;
  loadSpell(id: string): Promise<CosmicSpell | null>;
  listSpells(spellbookId: string, limit?: number): Promise<CosmicSpell[]>;
}

export interface IPFSStorage {
  pin(data: unknown): Promise<string>; // Returns CID
  fetch<T>(cid: string): Promise<T>;
  unpin(cid: string): Promise<void>;
}
```

## Storage Adapters

### Local Filesystem

```typescript
// storage/local.ts

export class LocalStorageAdapter implements StorageAdapter {
  private basePath: string;
  
  constructor(basePath = "~/.cosmic-spellbook") {
    this.basePath = path.resolve(basePath.replace("~", os.homedir()));
    this.ensureDir();
  }
  
  async save(spellbook: Spellbook): Promise<void> {
    const filepath = this.spellbookPath(spellbook.id);
    const data = this.serialize(spellbook);
    await fs.writeFile(filepath, JSON.stringify(data, null, 2));
  }
  
  async load(id: string): Promise<Spellbook | null> {
    const filepath = this.spellbookPath(id);
    try {
      const content = await fs.readFile(filepath, "utf-8");
      return this.deserialize(JSON.parse(content));
    } catch {
      return null;
    }
  }
  
  private serialize(spellbook: Spellbook): SerializedSpellbook {
    return {
      ...spellbook,
      grimoireNotation: Object.fromEntries(spellbook.grimoireNotation),
      learnedMeanings: Object.fromEntries(spellbook.learnedMeanings)
    };
  }
  
  private deserialize(data: SerializedSpellbook): Spellbook {
    return {
      ...data,
      createdAt: new Date(data.createdAt),
      lastSpellAt: data.lastSpellAt ? new Date(data.lastSpellAt) : undefined,
      grimoireNotation: new Map(Object.entries(data.grimoireNotation)),
      learnedMeanings: new Map(Object.entries(data.learnedMeanings))
    };
  }
}
```

### IPFS/Pinata

```typescript
// storage/ipfs.ts

export class PinataStorageAdapter implements IPFSStorage {
  private jwt: string;
  private gateway: string;
  
  constructor(jwt: string, gateway = "https://gateway.pinata.cloud") {
    this.jwt = jwt;
    this.gateway = gateway;
  }
  
  async pin(data: unknown): Promise<string> {
    const response = await fetch("https://api.pinata.cloud/pinning/pinJSONToIPFS", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${this.jwt}`
      },
      body: JSON.stringify({
        pinataContent: data,
        pinataMetadata: {
          name: `cosmic-spellbook-${Date.now()}`
        }
      })
    });
    
    const result = await response.json();
    return result.IpfsHash;
  }
  
  async fetch<T>(cid: string): Promise<T> {
    const response = await fetch(`${this.gateway}/ipfs/${cid}`);
    return response.json();
  }
}
```

## Mage Integration Layer

### Interface Definition

```typescript
// mage/interface.ts

export interface MageAgent {
  id: string;
  name: string;
  
  // Interpret a cosmic spell
  interpretSpell(spell: CosmicSpell): Promise<SpellInterpretation>;
  
  // Propose meaning for unclaimed emoji
  proposeMeaning(
    emoji: string, 
    context: ClaimContext
  ): Promise<string>;
  
  // Validate another mage's proposed meaning
  validateMeaning(claim: MeaningClaim): Promise<boolean>;
}

export interface SpellInterpretation {
  summary: string;
  segments: InterpretedSegment[];
  narrative?: string;
}

export interface InterpretedSegment {
  emoji: string;
  meaning: string;
  confidence: number;
  isKnown: boolean;
}
```

### OpenAI Implementation

```typescript
// mage/openai.ts

export class OpenAIMage implements MageAgent {
  id: string;
  name: string;
  private client: OpenAI;
  private systemPrompt: string;
  
  constructor(config: OpenAIMageConfig) {
    this.id = config.id;
    this.name = config.name;
    this.client = new OpenAI({ apiKey: config.apiKey });
    this.systemPrompt = this.buildSystemPrompt(config.spellbook);
  }
  
  async interpretSpell(spell: CosmicSpell): Promise<SpellInterpretation> {
    const response = await this.client.chat.completions.create({
      model: "gpt-4-turbo-preview",
      messages: [
        { role: "system", content: this.systemPrompt },
        { 
          role: "user", 
          content: `Interpret this cosmic spell: ${spell.spellString}\n\nEmoji breakdown:\n${this.formatEmojiMap(spell.emojiMap)}` 
        }
      ],
      response_format: { type: "json_object" }
    });
    
    return JSON.parse(response.choices[0].message.content!);
  }
  
  async proposeMeaning(emoji: string, context: ClaimContext): Promise<string> {
    const response = await this.client.chat.completions.create({
      model: "gpt-4-turbo-preview",
      messages: [
        { role: "system", content: this.systemPrompt },
        { 
          role: "user", 
          content: `Propose a meaning for the unclaimed cosmic emoji "${emoji}" in this context:\nSurrounding emojis: ${context.surroundingEmojis.join(" ")}\n\nProvide a concise meaning (under 100 characters) that fits the privacy/sovereignty theme.` 
        }
      ]
    });
    
    return response.choices[0].message.content!;
  }
  
  private buildSystemPrompt(spellbook: Spellbook): string {
    return `You are a Mage Agent in the Cosmic Spellbook system. Your spellbook contains these known meanings:\n\n${this.formatKnownMeanings(spellbook)}\n\nYour role is to interpret cosmic spell strings and propose meanings for unclaimed emojis. All interpretations should align with the themes of privacy, sovereignty, cryptography, and the First Person architecture.`;
  }
}
```

### Local/Ollama Implementation

```typescript
// mage/local.ts

export class LocalMage implements MageAgent {
  id: string;
  name: string;
  private baseUrl: string;
  private model: string;
  
  constructor(config: LocalMageConfig) {
    this.id = config.id;
    this.name = config.name;
    this.baseUrl = config.baseUrl || "http://localhost:11434";
    this.model = config.model || "llama3";
  }
  
  async interpretSpell(spell: CosmicSpell): Promise<SpellInterpretation> {
    const response = await fetch(`${this.baseUrl}/api/generate`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        model: this.model,
        prompt: this.buildPrompt(spell),
        format: "json"
      })
    });
    
    const result = await response.json();
    return JSON.parse(result.response);
  }
}
```

## Directory Structure

```
cosmic-spellbook/
├── package.json
├── tsconfig.json
├── .env.example
├── README.md
├── docs/
│   ├── SPEC.md
│   ├── API.md
│   ├── ARCHITECTURE.md
│   ├── GRIMOIRE.md
│   ├── KEY_DERIVATION.md
│   └── ROADMAP.md
├── src/
│   ├── index.ts
│   ├── core/
│   │   ├── spell-generator.ts
│   │   ├── phi-splitter.ts
│   │   ├── emoji-mapper.ts
│   │   ├── spellbook-manager.ts
│   │   ├── claiming-protocol.ts
│   │   ├── key-derivation.ts
│   │   └── recovery-protocol.ts
│   ├── types/
│   │   ├── core.ts
│   │   ├── emoji.ts
│   │   └── storage.ts
│   ├── storage/
│   │   ├── local.ts
│   │   ├── ipfs.ts
│   │   └── adapter.ts
│   ├── mage/
│   │   ├── interface.ts
│   │   ├── openai.ts
│   │   ├── anthropic.ts
│   │   ├── near.ts
│   │   └── local.ts
│   ├── orbitport/
│   │   ├── client.ts
│   │   └── mock.ts
│   ├── data/
│   │   ├── grimoire-pool.ts
│   │   └── cosmic-pool.ts
│   └── utils/
│       ├── hex.ts
│       ├── crypto.ts
│       └── emoji.ts
├── cli/
│   ├── index.ts
│   ├── commands/
│   │   ├── spell.ts
│   │   ├── claim.ts
│   │   ├── spellbook.ts
│   │   └── beacon.ts
│   └── formatters/
│       └── spell.ts
├── web/
│   ├── package.json
│   ├── next.config.js
│   ├── pages/
│   │   ├── index.tsx
│   │   ├── spell.tsx
│   │   ├── spellbook.tsx
│   │   └── api/
│   │       ├── cosmic-entropy.ts
│   │       ├── generate-spell.ts
│   │       └── claim.ts
│   └── components/
│       ├── SpellDisplay.tsx
│       ├── EmojiSlot.tsx
│       ├── SpellbookProgress.tsx
│       └── ClaimModal.tsx
└── test/
    ├── core/
    ├── storage/
    └── mage/
```
