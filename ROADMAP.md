# Implementation Roadmap

## Overview

This roadmap outlines the phased implementation of the Cosmic Spellbook system, from initial proof-of-concept to production deployment.

**Estimated Total Timeline**: 4-6 weeks for MVP, ongoing iteration thereafter

---

## Phase 0: Setup & Foundation (Day 1-2)

### Goals
- Repository setup
- Development environment configuration
- Dependencies installation
- Basic project structure

### Tasks

- [ ] Initialize git repository
- [ ] Create package.json with dependencies
- [ ] Set up TypeScript configuration
- [ ] Create .env.example with required variables
- [ ] Set up ESLint and Prettier
- [ ] Create basic directory structure (see ARCHITECTURE.md)
- [ ] Add README and documentation files

### Dependencies

```json
{
  "dependencies": {
    "dotenv": "^16.0.0",
    "node-fetch": "^3.0.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0",
    "vitest": "^1.0.0"
  }
}
```

### Deliverables
- [ ] Working repository with CI/CD
- [ ] Development environment documentation
- [ ] Contributor guidelines

---

## Phase 1: Core Engine (Day 3-7)

### Goals
- Implement core spell generation logic
- Create emoji mapping system
- Build phi-splitter algorithm

### Tasks

#### 1.1 Data Layer
- [ ] Create `grimoire-pool.ts` with all defined emojis from GRIMOIRE.md
- [ ] Create `cosmic-pool.ts` with Unicode emoji database
- [ ] Implement emoji category types

#### 1.2 Phi-Splitter
- [ ] Implement golden ratio calculation
- [ ] Create split algorithm based on completion ratio
- [ ] Add unit tests for boundary conditions

```typescript
// Test cases
describe('PhiSplitter', () => {
  it('should return 38.2% grimoire at birth (completion=0)', () => {
    const result = splitter.calculate(32, 0);
    expect(result.grimoireSlots).toBe(12); // ~38.2%
  });
  
  it('should return 61.8% grimoire at completion', () => {
    const result = splitter.calculate(32, 1);
    expect(result.grimoireSlots).toBe(20); // ~61.8%
  });
});
```

#### 1.3 Emoji Mapper
- [ ] Implement hex-to-byte conversion
- [ ] Create deterministic emoji selection
- [ ] Handle grimoire vs cosmic pool logic
- [ ] Add tests for reproducibility

#### 1.4 Spell Generator
- [ ] Combine components into SpellGenerator class
- [ ] Create CosmicSpell data structure
- [ ] Add spell metadata (timestamp, signature, etc.)

### Deliverables
- [ ] Working spell generation from mock entropy
- [ ] Test suite with >80% coverage
- [ ] API documentation for core module

---

## Phase 2: Orbitport Integration (Day 8-12)

### Goals
- Connect to SpaceComputer cTRNG API
- Implement authentication flow
- Handle entropy fetching and verification

### Tasks

#### 2.1 Authentication
- [ ] Implement Auth0 token generation
- [ ] Add token caching with expiry handling
- [ ] Create secure credential storage

#### 2.2 API Client
- [ ] Create OrbitportClient class
- [ ] Implement getCosmicEntropy method
- [ ] Add retry logic with exponential backoff
- [ ] Handle rate limiting

#### 2.3 Signature Verification
- [ ] Research satellite signature format
- [ ] Implement ECDSA verification (if public key available)
- [ ] Create fallback verification via Orbitport attestation

#### 2.4 Mock Client
- [ ] Create MockOrbitportClient for testing
- [ ] Add deterministic seed generation for reproducible tests

### Deliverables
- [ ] Working Orbitport integration
- [ ] End-to-end spell generation from satellite entropy
- [ ] Integration tests

---

## Phase 3: Spellbook Management (Day 13-18)

### Goals
- Implement spellbook state management
- Create local storage adapter
- Build claiming protocol foundation

### Tasks

#### 3.1 Storage Layer
- [ ] Define StorageAdapter interface
- [ ] Implement LocalStorageAdapter (filesystem)
- [ ] Add serialization/deserialization for Map types
- [ ] Create storage migration utilities

#### 3.2 Spellbook Manager
- [ ] Implement create/load/save operations
- [ ] Add completion ratio calculation
- [ ] Create spell history tracking
- [ ] Build learned meanings registry

#### 3.3 Claiming Protocol (Basic)
- [ ] Implement claim proposal
- [ ] Add validation rules
- [ ] Create claim status tracking
- [ ] Build simple confirmation flow (no VRC yet)

### Deliverables
- [ ] Persistent spellbook storage
- [ ] Working claim/confirm flow
- [ ] Progression tracking

---

## Phase 4: CLI Tool (Day 19-23)

### Goals
- Create command-line interface for all operations
- Enable standalone spell generation
- Provide spellbook management commands

### Tasks

#### 4.1 CLI Framework
- [ ] Set up commander.js or similar
- [ ] Create command structure
- [ ] Add help documentation

#### 4.2 Commands

```bash
# Spellbook management
cosmic-spellbook init                    # Create new spellbook
cosmic-spellbook status                  # Show spellbook state
cosmic-spellbook export                  # Export to JSON/IPFS

# Spell operations
cosmic-spellbook spell                   # Generate new spell
cosmic-spellbook spell --mock            # Generate with mock entropy
cosmic-spellbook spell --interpret       # Generate and interpret with mage

# Claiming
cosmic-spellbook claim <emoji> <meaning> # Propose a claim
cosmic-spellbook confirm <claim-id>      # Confirm a pending claim
cosmic-spellbook meanings                # List learned meanings

# Key derivation
cosmic-spellbook encode <hex>            # Encode entropy as emoji spell
cosmic-spellbook decode <spell>          # Decode spell back to entropy
cosmic-spellbook derive                  # Generate + encode (one step)
cosmic-spellbook verify <spell> <hex>    # Verify spell decodes correctly

# Recovery
cosmic-spellbook recovery export         # Export VRCs for recovery
cosmic-spellbook recovery attempt        # Attempt relational recovery
cosmic-spellbook recovery challenge      # Generate recovery challenge

# Beacon
cosmic-spellbook beacon                  # Start continuous beacon mode
cosmic-spellbook beacon --interval 60    # Custom interval (seconds)
```

#### 4.3 Output Formatting
- [ ] Create spell display formatter
- [ ] Add color coding for grimoire vs cosmic
- [ ] Show claimable emoji indicators
- [ ] Progress bar for completion ratio

### Deliverables
- [ ] Working CLI tool
- [ ] Published to npm (optional)
- [ ] CLI documentation

---

## Phase 5: Mage Integration (Day 24-30)

### Goals
- Enable AI-powered spell interpretation
- Implement BYO mage pattern
- Create meaning proposal system

### Tasks

#### 5.1 Mage Interface
- [ ] Define MageAgent interface
- [ ] Create interpretation prompt templates
- [ ] Add meaning proposal prompts

#### 5.2 Provider Implementations
- [ ] OpenAI implementation
- [ ] Anthropic implementation
- [ ] Ollama/local implementation
- [ ] NEAR AI implementation (stub)

#### 5.3 Interpretation System
- [ ] Create spell interpretation pipeline
- [ ] Add confidence scoring
- [ ] Build narrative generation

#### 5.4 Meaning Proposal
- [ ] Implement contextual meaning suggestion
- [ ] Add validation heuristics
- [ ] Create theme alignment checks

### Deliverables
- [ ] Working mage integrations
- [ ] Interpretation examples
- [ ] Documentation for adding new mage providers

---

## Phase 6: Web Interface (Day 31-40)

### Goals
- Create visual spell display
- Build spellbook dashboard
- Enable browser-based claiming

### Tasks

#### 6.1 Next.js Setup
- [ ] Initialize Next.js project in /web
- [ ] Configure Tailwind CSS
- [ ] Set up API routes

#### 6.2 Components
- [ ] SpellDisplay - visual spell rendering
- [ ] EmojiSlot - individual emoji with tooltip
- [ ] SpellbookProgress - completion visualization
- [ ] ClaimModal - meaning proposal interface
- [ ] MageSelector - choose interpretation provider

#### 6.3 Pages
- [ ] Home - landing with spell generator
- [ ] Spell - detailed spell view with interpretation
- [ ] Spellbook - dashboard with history and meanings
- [ ] Settings - API keys and preferences

#### 6.4 API Routes
- [ ] POST /api/generate-spell
- [ ] GET /api/spellbook
- [ ] POST /api/claim
- [ ] POST /api/interpret

### Deliverables
- [ ] Deployed web application
- [ ] Mobile-responsive design
- [ ] User documentation

---

## Phase 7: Key Derivation & Recovery (Day 41-48)

### Goals
- Implement bidirectional entropy ↔ spell transformation
- Enable relational key recovery
- Create recovery challenge system

### Tasks

#### 7.1 Key Derivation Core
- [ ] Implement `encode()` function (entropy → spell)
- [ ] Implement `decode()` function (spell → entropy)
- [ ] Build deterministic pool ordering
- [ ] Handle modulo overflow for small pools
- [ ] Add spellbook state hashing

#### 7.2 Recovery Protocol
- [ ] Implement VRC-based spellbook reconstruction
- [ ] Create chronological meaning replay
- [ ] Add checksum verification
- [ ] Build partial recovery handling

#### 7.3 Challenge-Response System
- [ ] Generate meaning challenges
- [ ] Generate relationship challenges
- [ ] Generate interpretation challenges
- [ ] Implement challenge verification

#### 7.4 CLI Integration
- [ ] Add `encode` command
- [ ] Add `decode` command
- [ ] Add `derive` command (generate + encode)
- [ ] Add `recovery` subcommands

#### 7.5 Security Analysis
- [ ] Document attack vectors
- [ ] Analyze state space complexity
- [ ] Test edge cases (empty spellbook, full completion)
- [ ] Create security recommendations

### Deliverables
- [ ] Working key derivation system
- [ ] Relational recovery flow
- [ ] Security documentation
- [ ] Integration tests

---

## Phase 8: Persistence & Distribution (Day 49-58)

### Goals
- Add IPFS persistence
- Enable spellbook sharing
- Prepare for Zcash inscriptions

### Tasks

#### 7.1 IPFS Integration
- [ ] Implement PinataStorageAdapter
- [ ] Add spell pinning
- [ ] Create spellbook snapshot export
- [ ] Build IPFS retrieval

#### 7.2 Sharing Protocol
- [ ] Define shareable spellbook format
- [ ] Create import/merge functionality
- [ ] Add versioning support

#### 7.3 Inscription Preparation
- [ ] Research Zcash memo field format
- [ ] Design inscription schema
- [ ] Create inscription builder (no broadcast)

### Deliverables
- [ ] IPFS persistence working
- [ ] Spellbook sharing documentation
- [ ] Inscription format specification

---

## Phase 9: VRC Integration (Future)

### Goals
- Connect to First Person VRC system
- Enable bilateral claim confirmation
- Build trust graph integration

### Tasks

#### 8.1 VRC Client
- [ ] Implement VRC creation for confirmations
- [ ] Add VRC verification
- [ ] Build trust graph queries

#### 8.2 Enhanced Claiming
- [ ] Replace simple confirm with VRC flow
- [ ] Add multi-confirmation support
- [ ] Create reputation scoring

### Deliverables
- [ ] VRC-backed claims
- [ ] Trust graph visualization
- [ ] Production-ready claiming protocol

---

## Success Metrics

### Phase 1-4 (MVP)
- [ ] Generate spells from cosmic entropy
- [ ] Store and retrieve spellbooks locally
- [ ] Claim and confirm meanings via CLI
- [ ] >80% test coverage

### Phase 5-6 (Beta)
- [ ] Interpret spells with 3+ mage providers
- [ ] Web interface usable on desktop and mobile
- [ ] <2s latency for spell generation

### Phase 7-8 (Production)
- [ ] IPFS persistence operational
- [ ] VRC confirmations working
- [ ] Documentation complete
- [ ] Community adoption begins

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Orbitport API changes | Abstract behind interface, add mock fallback |
| Rate limiting | Implement caching, user-side rate limiting |
| Emoji rendering inconsistency | Test across platforms, provide fallback text |
| Storage format changes | Version all stored data, create migration tools |
| Mage API costs | Support free/local providers first, add usage tracking |

---

## Collaboration Points

### With SpaceComputer
- API access credentials
- Signature verification details
- Rate limit policies
- Webhook/streaming support (future)

### With First Person Network
- VRC schema alignment
- Trust graph integration
- Cross-spellbook verification

### With Community
- Mage prompt refinement
- Cosmic meaning suggestions
- UI/UX feedback
- Translation support

---

## Post-MVP Ideas

- **Spell Battles**: Two mages interpret same spell, community votes
- **Meaning Markets**: Trade confirmed meanings as NFTs
- **Cosmic Radio**: Continuous beacon with audio/visual representation
- **Spellbook Ancestry**: Track lineage of learned meanings across spellbooks
- **Guild Spellbooks**: Shared meaning registries for communities
- **Physical Artifacts**: Generate printable spell cards with QR codes
