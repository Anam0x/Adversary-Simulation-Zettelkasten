# Todo

## Not Started

- [ ] Clarify the difference between Resources table and footnotes (maybe an informal style guide?)

## In Progress

- [x] Modify template structure of the categories to include Dataview queries
  - [x] Primary Categories
  - [x] Secondary Categories
  - [x] Update `README.md` to include Dataview support for JS queries

- [x] Content type template improvements
  - [x] NEW
    - [x] **Attack Surface**
    - [x] **Protocol**
    - [x] **Security Control**
  - [x] MODIFIED
    - [x] **Basic**
    - [x] **Biography**
    - [x] **Offensive Code**
    - [x] **Reference Material** (rename from "Mindmap")
    - [x] **Tradecraft** (rename from "TTP")
    - [x] **Tool**
    - [x] **Playbook**
    - [x] **Vulnerability**
    - [x] **Command**
    - [x] **IOC**
    - [x] **Case Study**
    - [x] **Infrastructure**
    - [x] **Lab Setup**
    - [x] **Idea**
    - [x] **Study Resources**

- [ ] Content note modernization and coverage plan
  - [ ] Phase 1: Global metadata and footer migration for existing notes
    - [ ] Rename `primary categories` to `primary-categories` across existing `03 - Content` notes
    - [ ] Rename `secondary categories` to `secondary-categories` across existing `03 - Content` notes
    - [ ] Add `note-status` to existing notes and decide default values for published vs in-progress content
    - [ ] Migrate legacy footer tables from `Hyperlink | Info` to `Reference | Info`
    - [ ] Differentiate note-level resources from body-level annotation footnotes in existing notes
  - [ ] Phase 2: Legacy content type normalization
    - [ ] Migrate `Mindmap` notes to `Reference Material`
    - [ ] Migrate `Debrief` notes to `Case Study`
    - [ ] Migrate `Payload` notes to `Offensive Code`
    - [ ] Migrate `TTP` notes to `Tradecraft`
    - [ ] Update legacy emoji/type tags so migrated notes match the current content taxonomy
  - [ ] Phase 3: Body-template conformance for represented content types
    - [ ] Basic
    - [ ] Biography
    - [ ] Case Study
    - [ ] Command
    - [ ] Idea
    - [ ] Infrastructure
    - [ ] Offensive Code
    - [ ] Playbook
    - [ ] Reference Material
    - [ ] Study Resources
    - [ ] Tool
    - [ ] Tradecraft
  - [ ] Phase 4: Create representative notes for currently unrepresented content types
    - [ ] Attack Surface
    - [ ] IOC
    - [ ] Lab Setup
    - [ ] Protocol
    - [ ] Security Control
    - [ ] Vulnerability
  - [ ] Phase 5: Final conformance validation
    - [ ] Confirm every content note uses current frontmatter keys and current `note-status` conventions
    - [ ] Confirm all represented note types follow the modern body/footer structure closely enough to avoid template drift
    - [ ] Confirm all content types are represented by at least one live note or one intentional example note

### High Priority (Core Functionality and Usability)

- [ ] Templater Plugin Script Enhancements
  - [ ] Phase 1: Entry points and workflow separation
    - [ ] Modify/add new Templater script to permit creating a new Content Type directly (currently only supports content type creation by going through note creation workflow)
    - [x] Modify back-linking primary/secondary categories menu
      - [x] Proceed without selecting a category
      - [x] De-select a category
    - [ ] Revisit whether additional dedicated entry-point scripts are still needed after direct content-type creation is implemented
  - [ ] Phase 2: Title/tag sanitization and filename safety
    - [ ] Sanitize search tag emoji because titles can contain illegal tag characters
    - [ ] Title character validation or search tag emoji sanitization for invalid tag characters (e.g., "$" is a valid character for note titles, but not tags)
    - [ ] Title character validation includes all link-breaking characters
    - [ ] Consolidate title validation and tag-sanitization rules into one documented normalization path
  - [ ] ~~Phase 3: Cancellation/failure hardening follow-up~~
    - [ ] ~~Need a hack for preventing file getting created via Templater if the user cancels the workflow or an unexpected error occurs~~
    - [ ] ~~Decide whether "safe recovery content" is sufficient or whether a stronger no-file workaround is still necessary~~
  - [ ] Phase 4: Metadata capture improvements
    - [ ] Prompt for aliases
    - [ ] Support iterative multi-value alias entry during note creation
  - [ ] Phase 5: Idea-state and unfinished-workflow support
    - [ ] Personal/unfinished ideas ~~directory~~ (property)
      - [ ] Update installation scripts/instructions
      - [ ] Update vault structure notes
      - [ ] Define and document how the property should interact with `note-status`, Dataview filtering, and Idea/Basic note creation
  - [ ] Deferred/possibly unsupported
    - [ ] ~~Go back button during note creation workflow (possibly not supportable)~~

- [x] Dataview query standardization
  - [x] Create dedicated `Dataview.md` files for lightweight and rich content templates
  - [x] Add restrained typed-relationship queries to lightweight content templates
  - [x] Evaluate and add deferred "Same Classification" queries to lightweight templates only where they prove useful
  - [x] Add first-pass queries to rich content templates

- [x] Templater architecture refactor for typed properties and staging workflow
  - [x] Phase 1: Template assembly refactor
    - [x] Update content note assembly to load `Dataview.md` between `Body.md` and `Footer.md`
    - [x] Keep category note assembly unchanged unless a dedicated `Dataview.md` convention is later adopted for categories
    - [x] Centralize template component loading so `Metadata`, `Body`, `Dataview`, and `Footer` are treated as explicit parts of a content type
    - [x] Ensure missing optional template parts fail gracefully with useful notices/logging
  - [x] Phase 2: Central schema definition for content properties
    - [x] Create a single schema/config object for each content type inside `0400 - Gen_Note.md`
    - [x] Define each property's name, data type, prompt text, required/optional status, and allowed values where relevant
    - [x] Separate universal properties from type-specific properties
    - [x] Standardize terminology around `Tradecraft` so old `Technique` labels do not remain in property prompts
  - [x] Phase 3: Prompting workflow for required properties
    - [x] Prompt only for required classification and relationship properties during note creation
    - [x] Use `suggester()` for controlled vocabularies such as tactic, severity, resource type, control category, and similar fields
    - [x] Use iterative selection prompts for list-of-link properties where practical
    - [x] Leave optional enrichment properties in the generated note for later manual completion
    - [x] Prompt for descriptions when creating primary and secondary categories
  - [x] Phase 4: Property validation and normalization
    - [x] Validate text, list, boolean, number, date, and datetime properties before final note assembly
    - [x] Normalize wiki-link formatting for link properties
    - [x] Validate controlled vocabulary fields against approved values
    - [x] Prevent malformed frontmatter from being written when prompts are cancelled or partially completed
  - [x] Phase 5: Draft status workflow
    - [x] Add a `status` property for content notes with at least `Draft` and `Ready` states
    - [x] Create new content notes in `03 - Content/` while defaulting them to a draft status
    - [x] Validate required metadata at creation time without treating that alone as publish-ready
    - [x] Prefer `note-status` as the draft signal instead of a top-of-note draft callout
    - [x] Update Dataview queries to filter draft notes by `note-status` instead of relying on directory moves
    - [x] Use status-only filtering so `☑️ Ready` notes can appear in query results without any promotion or path change
  - [x] Phase 6: Content type creation workflow updates
    - [x] Update custom content type generation to create `Dataview.md` alongside `Metadata.md`, `Body.md`, and `Footer.md`
    - [x] Seed new content types with a minimal `## Related Notes` layout appropriate to the selected query tier
    - [x] Keep built-in property schemas in `0400 - Gen_Note.md` while allowing new content types to start with universal content properties plus Dataview scaffolding
  - [x] Phase 7: Hardening and UX polish
    - [x] Improve cancellation handling so failed note creation leaves either no file or a safely staged draft
    - [x] Add clearer user notices for incomplete properties, fallback values, and final note destination
    - [x] Review whether a `status` or `schema-version` property should be auto-generated for migration and maintenance
      - [x] Keep `note-status` auto-generated for now
      - [x] Defer `schema-version` frontmatter until a future external schema architecture becomes a real migration path
    - [x] Add regression test notes or a manual validation checklist for each content type workflow
    - [x] Manual validation checklist
      - [x] Cancel during title entry and confirm the current file is left with safe recovery content instead of malformed frontmatter
      - [x] Create a content note with skipped typed-link selections and confirm placeholders remain in metadata plus a warning notice appears
      - [x] Create a successful content note and confirm `note-status` is written automatically
      - [x] Create a primary or secondary category note and confirm no content-only fallback frontmatter is introduced

### Medium Priority (Enhancement)

- [ ] Setup scripts
  - [ ] Bash for \*NIX-like
  - [ ] PowerShell for Windows
  - [ ] Git configuration
    - [ ] Automatically generated/edited .gitignore
    - [ ] Update Obsidian Git note

- [ ] Installation instructions
  - [X] README
  - [x] Getting Started
  - [x] Rewrite the "Vault Appendix - Modifying Vault Structure" note
  - [ ] Note on Linux, Windows, macOS, Android, iOS support (Obsidian Git plugin is not recommended for mobile platforms)

- [ ] Future architecture option: migrate built-in content type property definitions out of `0400 - Gen_Note.md` and into per-type `Schema.json` files for consistency with custom types

- [ ] Extract Templater script logic into individual scripts for:
  - [ ] New primary categories
  - [ ] New secondary categories
  - [ ] New content notes
  - [ ] New content types

### Low Priority (Cleanup/Polish)

## Done ✓

- [x] Deprovision the Admonitions community plugin
 - [x] Update README
 - [x] Replace code blocks with callouts
 - [x] Modify notes
 - [x] Create a note on how to use Obsidian's admonition callouts
