# Todo

## Not Started

- [ ] Modify footer template structure of the appropriate content types/categories to include Dataview queries

- [ ] Clarify the difference between Resources table and footnotes (maybe an informal style guide?)

- [ ] Add Python script to scrape MITRE ATT&CK Framework and generate TTP notes

- [ ] Improve "Biography" content type's timeline template

- [ ] Templater Plugin Script Enhancements
  - [ ] Modify/add new Templater script to permit creating a new Content Type directly (currently only supports content type creation by going through note creation workflow)
  - [ ] Go back button during note creation workflow (possibly not supportable)
  - [ ] Sanitize search tag emoji because titles can contain illegal tag characters (e.g., "$" is a valid character for note titles, but not tags)
  - [ ] Need a hack for preventing file getting created via Templater if the user cancels the workflow or an unexpected error occurs
  - [ ] Modify back-linking primary/secondary categories menu
    - [ ] Proceed without selecting a category
    - [ ] De-select a category

## In Progress

### High Priority (Core Functionality and Usability)

- [ ] Setup scripts
  - [ ] Bash for \*NIX-like
  - [ ] PowerShell for Windows
  - [ ] Git configuration
    - [ ] Automatically generated/edited .gitignore
    - [ ] Update Obsidian Git note

- [ ] Deprovision the Admonitions community plugin
 - [ ] Remove from README/installation scripts
 - [ ] Replace code blocks with callouts
 - [ ] Modify notes
 - [ ] Create a note on how to use Obsidian's admonition callouts

- [ ] Installation instructions
  - [X] README
  - [x] Getting Started
  - [x] Rewrite the "Vault Appendix - Modifying Vault Structure" note
  - [ ] Note on Linux, Windows, macOS, Android, iOS support (Obsidian Git plugin is not recommended for mobile platforms)

### Medium Priority (Enhancement)

- [ ] Personal/unfinished ideas directory
  - [ ] Update installation scripts/instructions
  - [ ] Update vault structure notes

### Low Priority (Cleanup/Polish)

- [ ] Example notes for new content types
  - [x] Biography
  - [ ] Case Study
  - [x] Command
  - [x] Infrastructure
  - [ ] IOC
  - [ ] Lab Setup
  - [x] Mindmap
  - [x] Payload
  - [x] Playbook
  - [x] Study Resources
  - [ ] Vulnerability

## Done ✓
