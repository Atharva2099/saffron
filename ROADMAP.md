# Saffron Project Roadmap

## Overview

Saffron is an AI-native PCB design tool that bridges visual circuit design with intelligent analysis and optimization. This roadmap outlines the phased development approach, starting with a low-risk unidirectional flow and progressively adding capabilities.

---

## Phase 1: The Intelligent Observer (Current Focus)

**Status**: Active Development  
**Goal**: AI acts as a read-only consultant, providing real-time design feedback without modifying the circuit.

### Core Features

#### 1.1 Visual Design Interface
- PySide6-based schematic editor with drag-and-drop components
- Component library (initial focus: common passives and basic ICs)
- Real-time schematic canvas rendering using QGraphicsView
- Basic editing tools (add, delete, move, connect)

#### 1.2 Translation Layer
- **LibrePCB File Parser**: Read `.lpp` files and extract component/netlist data
- **Netlist Generator**: Convert schematic to SPICE-compatible netlist format
- **S-Expression Generator**: Create structured circuit representation for AI context
- **Component Mapping System**: Maintain UUID mappings between LibrePCB and internal representation

#### 1.3 AI Analysis Engine
- **Context Builder**: Construct AI prompts from netlist + S-expressions + metadata
- **Design Review System**: AI analyzes circuit for common issues:
  - Missing decoupling capacitors
  - Power supply stability concerns
  - Signal integrity issues
  - Incorrect component values
  - Missing ground connections
  - Noise considerations
- **Feedback UI**: Sidebar or panel displaying AI suggestions with explanations

#### 1.4 Real-Time Analysis
- Trigger netlist extraction on circuit changes (debounced)
- Background AI analysis as user designs
- Non-blocking UI (user can continue editing while analysis runs)

### Technical Requirements

#### Dependencies
- PySide6 (Qt for Python)
- LibrePCB file format parser (custom implementation)
- AI/LLM API integration (OpenAI, Anthropic, or local model)
- Python SPICE netlist generation libraries

#### Data Flow
```
User Edit → Schematic Model → Netlist Generator → S-Expression Generator
                                                         ↓
AI Context Builder ← Metadata (component types, design notes)
                                                         ↓
AI Analysis → Structured Feedback → UI Display
```

#### Key Files/Modules
- `saffron/design/schematic_canvas.py` - Main UI canvas
- `saffron/design/schematic_model.py` - Internal circuit representation
- `saffron/translate/librepcb_reader.py` - Parse LibrePCB files
- `saffron/translate/netlist_generator.py` - Generate SPICE netlist
- `saffron/translate/sexpr_generator.py` - Generate S-expressions
- `saffron/ai/context_builder.py` - Build AI prompts
- `saffron/ai/feedback_ui.py` - Display AI suggestions

### Success Criteria
- Successfully parse LibrePCB `.lpp` files
- Generate accurate netlists for common circuits (voltage dividers, op-amps, basic filters)
- AI provides actionable, specific feedback on circuit issues
- Analysis completes within 5-10 seconds for circuits with <50 components
- Zero risk of data corruption (read-only analysis)

### Limitations (By Design)
- AI cannot modify the circuit (user must apply suggestions manually)
- No simulation capabilities (textual analysis only)
- Limited to circuits supported by SPICE netlist format

---

## Phase 2: The Visual Analyst (Future Scope)

**Status**: Planned  
**Goal**: Add simulation capabilities with AI-powered result analysis and visual circuit understanding.

### Core Features

#### 2.1 Simulation Integration
- **PySpice Bridge**: Connect to Ngspice shared library for circuit simulation
- **Simulation Types**: 
  - DC operating point analysis
  - AC frequency response
  - Transient analysis
  - (Future: Noise analysis, Monte Carlo)
- **Simulation Trigger**: Manual trigger from UI, with option for auto-sim on circuit changes

#### 2.2 Visual Language Model (VLM) Integration
- **Schematic Screenshot Capture**: Capture current QGraphicsScene as image
- **Visual Circuit Audit**: AI analyzes screenshot alongside netlist to catch:
  - Overlapping wires that appear connected but aren't
  - Visual layout issues
  - Component orientation errors
  - Missing visual indicators (polarity marks, reference designators)
- **Multi-Modal Context**: Combine visual + netlist + S-expressions for richer analysis

#### 2.3 AI-Powered Result Interpretation
- **Simulation Result Analysis**: AI explains simulation plots and data
- **Comparative Analysis**: "This circuit shows 3dB rolloff at 10kHz, which may be too aggressive for your application"
- **Optimization Hints**: AI suggests parameter changes based on simulation results
- **Visual Plot Generation**: Generate and display simulation plots in UI

#### 2.4 Enhanced Feedback System
- **Confidence Scores**: AI indicates certainty level for each suggestion
- **Ranked Suggestions**: Prioritize feedback by impact/severity
- **Explanation Depth**: User can request detailed explanations for any suggestion

### Technical Considerations (Open Ends)

#### Architecture Flexibility
- Simulation engine abstraction layer (allow swapping Ngspice for alternatives)
- VLM provider abstraction (support multiple vision models)
- Caching strategy for simulation results (avoid re-simulating unchanged circuits)

#### Performance Optimization
- Lazy simulation loading (only simulate when requested)
- Background simulation queue (batch requests)
- Incremental netlist updates (only re-simulate changed portions)

#### Data Structures
- Maintain Phase 1 S-expression format (ensures backward compatibility)
- Extend with simulation metadata (results, timestamps, parameters)
- Consider versioning for simulation data

### Integration Points with Phase 1
- Leverage existing netlist generation pipeline
- Reuse AI context builder (extend with simulation results)
- Build on existing feedback UI (add simulation result visualization)

### Success Criteria
- Successfully run DC/AC/transient simulations via PySpice
- VLM accurately identifies visual issues in schematics
- AI explanations help users understand simulation results
- Simulation completes in reasonable time (<30s for medium circuits)

---

## Phase 3: The Intelligent Weaver (Future Scope)

**Status**: Future Vision  
**Goal**: Bidirectional AI-driven circuit modification with automatic visual synchronization.

### Core Features

#### 3.1 Bidirectional Sync Architecture
- **AI Edit Generation**: AI produces structured edit commands (not free-form code)
- **Edit Validation**: Validate all AI suggestions before applying:
  - Component rule checking (pin counts, valid values)
  - Electrical sanity checks (voltage/current limits)
  - Topology validation
- **Component Matching**: Maintain robust UUID mapping system (LibrePCB ↔ AI names)
- **Atomic Operations**: All-or-nothing AI edits (revert on failure)

#### 3.2 Visual Synchronization
- **LibrePCB File Writer**: Modify `.lpp` files to reflect AI changes
- **Model Update System**: Update internal schematic model from AI edits
- **UI Re-rendering**: Trigger visual updates after file modification
- **Change Visualization**: Show before/after diff (highlight changed components)

#### 3.3 Edit Workflow (Cursor-Style)
- **Lock System**: When AI is editing, lock user controls (prevent race conditions)
- **Progress Feedback**: Show "AI is modifying circuit..." with status updates
- **Preview & Accept**: Show AI changes before applying (diff view)
- **Revert Capability**: User can reject AI changes, reverting to previous state

#### 3.4 Advanced AI Capabilities
- **Structural Changes**: AI can add/remove components (not just modify values)
- **Auto-Placement**: Intelligent component positioning for new components
- **Design Optimization**: AI suggests topology improvements based on goals
- **Constraint-Aware Editing**: AI respects user-defined constraints (cost, size, power)

### Technical Considerations (Open Ends)

#### Sync Reliability
- Transaction logging for all AI edits
- Rollback mechanism for failed syncs
- Conflict resolution strategies (if user and AI both modify same component)

#### Edit Format Design
- Structured JSON edits vs. PySpice code generation (trade-off: precision vs. flexibility)
- Need to decide: Does AI return edit commands or generated code?
- Consider hybrid approach: structured edits for simple changes, code generation for complex

#### Component Placement Strategy
- Auto-placement algorithm (grid-based, force-directed, or ML-based)
- Connection-aware placement (place new components near related nets)
- Overlap detection and resolution

#### Undo/Redo Complexity
- Separate undo stacks for user vs. AI edits?
- Or unified stack with metadata (who made the change)?
- Need to handle: User undoes AI change, then AI suggests again

#### Performance & Scalability
- Real-time sync might be slow for large circuits (100+ components)
- Consider batching AI edits into single operation
- Defer non-critical updates (visual polish can update after core changes)

### Integration Points with Previous Phases
- Build on Phase 1 netlist/S-expression infrastructure
- Leverage Phase 2 simulation for validation (simulate AI suggestions before applying)
- Extend Phase 2 VLM for visual verification of AI changes

### Success Criteria
- AI edits successfully sync back to LibrePCB files
- Visual updates occur smoothly without UI freezing
- Zero data corruption from AI edits
- Users trust AI modifications (high accept rate)

### Known Challenges
- **Component Matching**: Reliable UUID mapping across edits
- **Placement Quality**: AI-generated layouts must be reasonable
- **Edit Validation**: Preventing impossible/invalid circuits
- **User Trust**: AI must be accurate enough that users accept suggestions

---

## Cross-Phase Considerations

### File Format Strategy
- Maintain compatibility with LibrePCB `.lpp` format (all phases)
- Internal representation (S-expressions) should be extensible across phases
- Consider versioning for internal formats as features evolve

### Licensing Compliance
- All phases must comply with GPLv3 (due to LibrePCB/PySpice dependencies)
- Document all dependencies and their licenses
- Ensure contributors understand license requirements

### User Experience Consistency
- UI/UX patterns established in Phase 1 should extend to later phases
- Consistent feedback mechanisms across all phases
- Progressive disclosure: advanced features don't overwhelm beginners

### Testing Strategy
- Phase 1: Test with known-good circuits (validate netlist generation accuracy)
- Phase 2: Test with circuits with known simulation results (validate correctness)
- Phase 3: Extensive validation testing (prevent corrupting user designs)

### Documentation Needs
- Phase 1: User guide for AI feedback interpretation
- Phase 2: Simulation guide, understanding AI explanations
- Phase 3: Trust and verification guide (when to accept/reject AI changes)

---

## Milestone Timeline (Tentative)

### Phase 1 Completion
- Target: 2-3 months from start
- Deliverable: Functional AI analysis tool with LibrePCB integration

### Phase 2 Planning
- Begin after Phase 1 proves value
- Estimate: 3-4 months (includes VLM integration complexity)

### Phase 3 Planning
- Begin only after Phases 1 & 2 are stable
- Estimate: 4-6 months (highest complexity, most testing needed)

---

## Notes

- This roadmap is flexible and subject to change based on:
  - User feedback from Phase 1
  - Technical discoveries during development
  - Evolution of AI/LLM capabilities
  - Community needs and requests

- Each phase should be validated with real users before proceeding to next phase

- Maintain backward compatibility where possible (Phase 2 should work with Phase 1 designs)

- Consider open-source community contributions for component libraries and test cases

