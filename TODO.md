# Todo

## Not Started

- [ ] Clarify the difference between Resources table and footnotes (maybe an informal style guide?)

## In Progress

- [ ] Modify template structure of the categories to include Dataview queries
  - [x] Primary Categories
  - [ ] Secondary Categories
  - [ ] Update `README.md` to include Dataview support for JS queries

- [ ] Content type template improvements
  - [ ] NEW
    - [ ] **Attack Surface**
    - [ ] **Protocol**
    - [ ] **Security Control**
  - [ ] MODIFIED
    - [ ] **Biography**
    - [ ] **Offensive Code**
    - [ ] **Reference Material** (rename from "Mindmap")
    - [ ] **Technique** (rename from "TTP")
    - [ ] **Tool**
    - [ ] **Playbook**
    - [ ] **Vulnerabilities**
    - [ ] **Command**
    - [ ] **IOC**
    - [ ] **Case Study**
    - [ ] **Infrastructure**
    - [ ] **Lab Setup**

- [ ] Example notes for all content types
  - [ ] Attack Surface
  - [x] Basic
  - [x] Biography
  - [ ] Case Study
  - [x] Command
  - [ ] Idea
  - [x] Infrastructure
  - [ ] IOC
  - [ ] Lab Setup
  - [ ] Offensive Code
  - [x] Playbook
  - [ ] Protocol
  - [ ] Reference Material
  - [ ] Security Control
  - [x] Study Resources
  - [ ] Technique
  - [x] Tool
  - [ ] Vulnerability

### High Priority (Core Functionality and Usability)

- [ ] Templater Plugin Script Enhancements
  - [ ] Modify/add new Templater script to permit creating a new Content Type directly (currently only supports content type creation by going through note creation workflow)
  - [ ] ~~Go back button during note creation workflow (possibly not supportable)~~
  - [ ] Sanitize search tag emoji because titles can contain illegal tag characters 
  - [ ] Need a hack for preventing file getting created via Templater if the user cancels the workflow or an unexpected error occurs
  - [x] Modify back-linking primary/secondary categories menu
    - [x] Proceed without selecting a category
    - [x] De-select a category
  - [ ] Personal/unfinished ideas directory
    - [ ] Update installation scripts/instructions
    - [ ] Update vault structure notes
  - [ ] Title character validation or search tag emoji sanitization for invalid tag characters (e.g., "$" is a valid character for note titles, but not tags)
  - [ ] Title character validation includes all link-breaking characters
  - [ ] Prompt for aliases

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

### Low Priority (Cleanup/Polish)

## Done ✓

- [x] Deprovision the Admonitions community plugin
 - [x] Update README
 - [x] Replace code blocks with callouts
 - [x] Modify notes
 - [x] Create a note on how to use Obsidian's admonition callouts
