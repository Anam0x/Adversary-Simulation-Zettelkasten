<%* 
/**
 * Adversary Simulation Zettelkasten - Automated Note and Category Creation System
 * 
 * This script automates the creation of hierarchical notes for a red team reference guide:
 * - Primary Categories (🥇): High-level topics like "Penetration Test", "Red Team"
 * - Secondary Categories (🥈): Mid-level topics like "Active Directory", "Post-Exploitation" 
 * - Content Notes (⚛️): Atomic notes with specific content types like "Tools", "Tradecraft", and "Payloads"
 * 
 * Features:
 * - Interactive note type selection with validation
 * - Smart emoji selection for categorization tags
 * - Template-based content generation
 * - Automated file organization and linking
 * - Dynamic content type and primary category creation
 */

//////////////////////////////////////////////////////////////////////////////////
//                                 CONSTANTS                                   //
//////////////////////////////////////////////////////////////////////////////////

const PATHS = {
    PRIMARY_CATEGORIES: "01 - Primary Categories",
    SECONDARY_CATEGORIES: "02 - Secondary Categories", 
    CONTENT: "03 - Content",
    CONTENT_TEMPLATES: "04 - Templates/04 - Content",
    PRIMARY_TEMPLATE_META: "[[04 - Templates/04 - Primary Category/Metadata]]",
    PRIMARY_TEMPLATE_BODY: "[[04 - Templates/04 - Primary Category/Body]]",
    SECONDARY_TEMPLATE_META: "[[04 - Templates/04 - Secondary Category/Metadata]]",
    SECONDARY_TEMPLATE_BODY: "[[04 - Templates/04 - Secondary Category/Body]]",
    BASIC_TEMPLATE: "04 - Templates/04 - Content/Basic"
};

const NOTE_TYPES = {
    PRIMARY: "Primary Category",
    SECONDARY: "Secondary Category", 
    CONTENT: "Content",
    CONTENT_TYPE: "Content Type"
};

const VALIDATION_LIMITS = {
    MAX_TITLE_LENGTH: 100,
    MAX_VALIDATION_ATTEMPTS: 5,
    MAX_FILENAME_RETRIES: 3,
    MAX_TAG_LENGTH: 1
};

const EMOJI_SELECTION = {
    TYPES: {
        UTILITIES: "UTILITIES_",
        MANUAL: "MANUAL_ENTRY", 
        RANDOM: "RANDOM"
    },
    DISPLAY_TEXT: {
	    UTILITIES: "--- Utilities ---",
        MANUAL: "✏️ Enter emoji manually",
        RANDOM: "🎲 Random selection"
    },
    DEFAULT_EMOJI: "📁"
};

const EMOJI_NOTICE = {
	ERROR: "❌",
	SUGGESTION: "💡",
	SUCCESS: "✅",
	WARNING: "⚠️"
}

const SEGMENTER = new Intl.Segmenter('en', { granularity: 'grapheme' });
const RESERVED_EMOJIS = new Set(['🥇', '🥈', '⚛️']);
const ILLEGAL_CHARS = /[<>:"/\\|?*\x00-\x1F]/g;
const INVISIBLE_CHARS = /[\u200B-\u200F\u202A-\u202E\u2060-\u206F\uFEFF]/;
const RESERVED_NAMES = /^(CON|PRN|AUX|NUL|COM[1-9]|LPT[1-9])(\.|$)/i;
const EMOJI_REGEX = /[\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}]|[\u{1F900}-\u{1F9FF}]|[\u{1FA70}-\u{1FAFF}]|[\u{2300}-\u{23FF}]|[\u{2B50}]|[\u{2194}-\u{21AA}]|[\u{231A}-\u{231B}]|[\u{25AA}-\u{25FE}]/u;
const DIVIDER = "\n\n---\n\n";
const TIMESTAMP = "*Created Date*: <%+tp.file.creation_date(\"MMMM Do YYYY (HH:mm a)\")%\>  \n*Last Modified Date*: \<%+tp.file.last_modified_date(\"MMMM Do YYYY (HH:mm a)\")%\>";
const NOTE_STATUS = {
    DRAFT: "✍️ Draft",
    READY: "☑️ Ready"
};
const QUERY_TIERS = {
    NONE: "None",
    LIGHTWEIGHT: "Lightweight",
    RICH: "Rich"
};

const UNIVERSAL_CONTENT_PROPERTIES = [
    { name: "aliases", type: "list[text]", required: false, prompt: "Aliases" },
    { name: "tags", type: "list[text]", required: false, prompt: "Search tags" },
    { name: "primary-categories", type: "list[link]", required: true, prompt: "Primary categories" },
    { name: "secondary-categories", type: "list[link]", required: true, prompt: "Secondary categories" },
    { name: "type", type: "text", required: true, prompt: "Content type label" }
];

const OTHER_OPTION = "Other";

/**
 * Appends a reusable "Other" option to a controlled vocabulary while ensuring it stays last.
 * @param {string[]} values - Ordered list of controlled vocabulary values.
 * @returns {string[]} - New vocabulary array with "Other" appended at the end.
 */
function withOtherOption(values = []) {
    const filteredValues = values.filter(value => value !== OTHER_OPTION);
    return [...filteredValues, OTHER_OPTION];
}

/**
 * Creates starter Dataview template content for a new custom content type based on the selected query tier.
 * @param {string} queryTier - Query tier constant from QUERY_TIERS.
 * @returns {string} - Markdown content for the starter `Dataview.md` file.
 */
function createStarterDataviewTemplate(queryTier) {
    switch (queryTier) {
        case QUERY_TIERS.RICH:
            return [
                "## Related Notes",
                "",
                "### Same Classification",
                "",
                "### Typed Relationships",
                "",
                "### Reverse Relationships"
            ].join("\n");
        case QUERY_TIERS.LIGHTWEIGHT:
            return [
                "## Related Notes",
                "",
                "### Typed Relationships",
                "",
                "### Same Classification"
            ].join("\n");
        case QUERY_TIERS.NONE:
        default:
            return [
                "## Related Notes",
                "",
                "<!-- Add Dataview queries here if this content type later benefits from automatic discovery. -->"
            ].join("\n");
    }
}

const CONTROLLED_VALUES = {
    CONFIDENCE_LEVELS: ["High", "Medium", "Low"],
    AFFECTED_PLATFORMS: withOtherOption(["Windows", "Linux", "macOS", "Cloud", "Network", "Application", "SaaS", "Identity", "Email", "Hypervisor", "Database", "Mobile", "Kubernetes", "Entra ID", "Active Directory"]),
    ATTACK_SURFACE_PLATFORMS: withOtherOption(["Windows", "Linux", "macOS", "Cloud", "Network", "SaaS", "Hardware"]),
    ATTACK_SURFACE_DEPLOYMENT_MODELS: withOtherOption(["Cloud", "On-Premises", "Hybrid", "Containerized"]),
    ATTACK_SURFACE_AUTHENTICATION_METHODS: withOtherOption(["API Keys", "OAuth", "OpenID Connect", "SAML", "SSH", "Certificates", "Kerberos", "NTLM", "Basic Auth", "Digest", "LDAP Bind", "RADIUS", "TACACS+", "MFA", "Passkeys/FIDO2"]),
    CASE_STUDY_ENGAGEMENT_TYPES: withOtherOption(["Red Team", "Penetration Test", "Purple Team", "Real-World Incident", "Training Exercise", "APT Campaign"]),
    COMMAND_EXECUTION_ENVIRONMENTS: withOtherOption(["Windows", "Linux", "macOS", "Container", "AWS", "Azure", "GCP"]),
    COMMAND_TARGET_ENVIRONMENTS: withOtherOption(["Active Directory", "Entra ID", "Microsoft 365", "AWS", "Azure", "GCP", "Kubernetes", "Network", "Database", "Email", "SaaS", "Web Application"]),
    CONTROL_CATEGORIES: withOtherOption(["EDR", "AV", "SIEM", "WAF", "IDS/IPS", "DLP", "Firewall", "MFA", "SAST", "DAST", "SCA", "Identity Monitoring", "NDR", "XDR", "SOAR", "PAM", "CASB", "ZTNA", "MDM", "DNS Filtering", "Email Security", "Sandboxing", "IAM", "CSPM", "CWPP", "CIEM"]),
    CONTROLLED_PROVIDER_STATUS: ["Not Started", "In Progress", "Completed", "Revisit"],
    DETECTION_DIFFICULTY: ["Low", "Medium", "High", "Very High"],
    DIFFICULTY_LEVELS: ["Beginner", "Intermediate", "Advanced", "Expert"],
    ENTRY_POINTS: withOtherOption(["CLI", "DLL Export", "Macro", "Shellcode Loader", "Web Endpoint", "Script", "Service", "Driver", "BOF", "Beacon Object File", "Reflective Loader", "COM Object", "Browser Extension", "Package"]),
    IDEA_STATUS: ["Draft", "Exploring", "Testing", "Archived"],
    INFRASTRUCTURE_PLATFORMS: withOtherOption(["AWS", "Azure", "GCP", "On-Premises", "Hybrid", "Container", "Windows", "Linux"]),
    INFRASTRUCTURE_TYPES: withOtherOption(["C2 Server", "Redirector", "Phishing Infrastructure", "Lab Environment", "Cloud Architecture", "Network Topology", "Team Server", "Payload Hosting", "Staging Server", "VPN", "Domain Fronting", "Mail Infrastructure", "Identity Infrastructure", "Container Cluster"]),
    IOC_TYPES: withOtherOption(["Hash", "IP Address", "Domain", "URL", "Email", "File Path", "Registry Key", "Mutex", "User-Agent", "Certificate", "Event ID", "JA3/JA4", "Named Pipe", "Scheduled Task", "Service Name", "Process Name", "Command Line", "Hostname", "ASN"]),
    LAB_PURPOSES: withOtherOption(["Active Directory Lab", "Cloud Environment", "Vulnerable Web App", "CTF", "Training", "Malware Analysis", "Detection Engineering", "Protocol Research", "Exploit Development", "Web App Testing", "Phishing Simulation", "Cloud Privilege Escalation"]),
    LAB_PLATFORMS: withOtherOption(["Windows", "Linux", "macOS", "Cloud", "Network", "Hardware"]),
    LANGUAGES: withOtherOption(["C", "C++", "C#", "Python", "PowerShell", "Assembly", "Rust", "Go", "JavaScript", "TypeScript", "Java", "PHP", "Ruby", "VBA", "VBScript", "Swift", "Kotlin"]),
    OFFENSIVE_CODE_PLATFORMS: withOtherOption(["Windows", "Linux", "macOS", "Cross-Platform", "Cloud", "Identity", "SaaS", "Email", "Hypervisor", "Database", "Mobile", "Kubernetes", "Entra ID", "Active Directory"]),
    OPSEC_RISK: ["Low", "Medium", "High", "Critical"],
    PERMISSIONS_REQUIRED: withOtherOption(["User", "Authenticated User", "Administrator", "SYSTEM", "root"]),
    PLAYBOOK_SCOPES: withOtherOption(["External", "Internal", "Cloud", "Physical", "Social Engineering", "Web Application", "Identity", "Email", "Kubernetes", "Multi-Stage"]),
    PRIMARY_PLATFORMS: withOtherOption(["Windows", "Linux", "macOS", "Cloud", "Identity", "SaaS", "Email", "Hypervisor", "Database", "Mobile", "Kubernetes", "Entra ID", "Active Directory"]),
    PROTECTED_PLATFORMS: withOtherOption(["Windows", "Linux", "macOS", "Cloud", "Network", "Container", "Identity", "SaaS", "Email", "Hypervisor", "Database", "Mobile", "Kubernetes", "Entra ID", "Active Directory"]),
    PROTOCOL_AUTHENTICATION_METHODS: withOtherOption(["Kerberos", "NTLM", "Certificates", "Tokens", "API Keys", "OAuth", "OpenID Connect", "Basic Auth", "Digest", "LDAP Bind", "RADIUS", "TACACS+", "MFA", "Passkeys/FIDO2"]),
    PROTOCOL_FAMILIES: withOtherOption(["Application", "Network", "Authentication", "Industrial", "Cloud API", "Identity", "Directory", "Management", "Messaging", "Remote Access", "Web", "File Sharing", "Database", "Service Discovery"]),
    REFERENCE_RESOURCE_TYPES: withOtherOption(["Book", "Article", "Blog Post", "Whitepaper", "Documentation", "Video", "Research Paper", "Conference Talk", "Cheat Sheet", "RFC/Standard", "Vendor Documentation", "GitHub Repository"]),
    SEVERITY_LEVELS: ["Critical", "High", "Medium", "Low"],
    SHELL_ENVIRONMENTS: withOtherOption(["PowerShell", "pwsh", "CMD", "Bash", "sh", "ZSH", "Fish", "AWS CLI", "Azure CLI", "SQL Shell", "KQL", "Graph PowerShell", "kubectl", "Terraform CLI"]),
    STUDY_RESOURCE_TYPES: withOtherOption(["Course", "Certification", "Lab", "Book", "Article", "Playlist", "Workshop", "Practice Range", "CTF", "Assessment", "Module", "Bootcamp"]),
    TACTICS: ["Reconnaissance", "Resource Development", "Initial Access", "Execution", "Persistence", "Privilege Escalation", "Defense Evasion", "Credential Access", "Discovery", "Lateral Movement", "Collection", "Command and Control", "Exfiltration", "Impact"],
    TARGET_PLATFORMS: withOtherOption(["Windows", "Linux", "macOS", "Cloud", "Network", "Container", "Identity", "SaaS", "Email", "Hypervisor", "Database", "Mobile", "Kubernetes", "Entra ID", "Active Directory"])
};

const CONTENT_TYPE_PROPERTY_SCHEMAS = {
    "Attack Surface": [
        { name: "platforms", type: "list[text]", required: true, promptOnCreate: true, prompt: "Platforms", allowedValues: CONTROLLED_VALUES.ATTACK_SURFACE_PLATFORMS },
        { name: "deployment-models", type: "list[text]", required: false, promptOnCreate: true, prompt: "Deployment models", allowedValues: CONTROLLED_VALUES.ATTACK_SURFACE_DEPLOYMENT_MODELS },
        { name: "authentication-methods", type: "list[text]", required: false, promptOnCreate: true, prompt: "Authentication methods", allowedValues: CONTROLLED_VALUES.ATTACK_SURFACE_AUTHENTICATION_METHODS },
        { name: "related-controls", type: "list[link]", required: true, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Related security controls", targetContentTypes: ["Security Control"] },
        { name: "related-tradecraft", type: "list[link]", required: true, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Related tradecraft notes", targetContentTypes: ["Tradecraft"] },
        { name: "related-vulnerabilities", type: "list[link]", required: true, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Related vulnerabilities", targetContentTypes: ["Vulnerability"] },
        { name: "related-tools", type: "list[link]", required: true, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Related tools", targetContentTypes: ["Tool"] }
    ],
    "Basic": [],
    "Biography": [
        { name: "organizations", type: "list[text|link]", required: false, prompt: "Organizations", example: "[[TrustedSec]], SpecterOps" },
        { name: "roles", type: "list[text]", required: false, prompt: "Roles", example: "'Researcher', 'Professor'" },
        { name: "active-from", type: "date", required: false, prompt: "Active from date" },
        { name: "active-to", type: "date", required: false, prompt: "Active to date" },
        { name: "related-tradecraft", type: "list[link]", required: true, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Related tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "related-tools", type: "list[link]", required: true, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Related tools", targetContentTypes: ["Tool"] }
    ],
    "Case Study": [
        { name: "engagement-type", type: "text", required: true, promptOnCreate: true, prompt: "Engagement type", allowedValues: CONTROLLED_VALUES.CASE_STUDY_ENGAGEMENT_TYPES },
        { name: "target-environment", type: "text", required: true, prompt: "Target environment", example: "'Internal Windows estate with legacy PKI'" },
        { name: "used-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Used tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "used-tools", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Used tools", targetContentTypes: ["Tool"] },
        { name: "exploited-vulnerabilities", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Exploited vulnerabilities", targetContentTypes: ["Vulnerability"] },
        { name: "start-date", type: "date", required: false, prompt: "Start date" },
        { name: "end-date", type: "date", required: false, prompt: "End date" }
    ],
    "Command": [
        { name: "execution-environments", type: "list[text]", required: true, promptOnCreate: true, prompt: "Execution environments", allowedValues: CONTROLLED_VALUES.COMMAND_EXECUTION_ENVIRONMENTS },
        { name: "target-environments", type: "list[text]", required: false, promptOnCreate: true, prompt: "Target environments", allowedValues: CONTROLLED_VALUES.COMMAND_TARGET_ENVIRONMENTS },
        { name: "shell-environments", type: "list[text]", required: false, promptOnCreate: true, prompt: "Shell environments", allowedValues: CONTROLLED_VALUES.SHELL_ENVIRONMENTS },
        { name: "permissions-required", type: "list[text]", required: false, promptOnCreate: true, prompt: "Permissions required", allowedValues: CONTROLLED_VALUES.PERMISSIONS_REQUIRED },
        { name: "used-in-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Used in tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "used-by-tools", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Used by tools", targetContentTypes: ["Tool"] },
        { name: "supports-remote", type: "boolean", required: false, prompt: "Supports remote usage" }
    ],
    "Idea": [
        { name: "status", type: "text", required: false, promptOnCreate: true, prompt: "Idea status", allowedValues: CONTROLLED_VALUES.IDEA_STATUS },
        { name: "related-tradecraft", type: "list[link]", required: true, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Related tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "related-tools", type: "list[link]", required: true, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Related tools", targetContentTypes: ["Tool"] },
        { name: "related-playbooks", type: "list[link]", required: true, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Related playbooks", targetContentTypes: ["Playbook"] }
    ],
    "Infrastructure": [
        { name: "infrastructure-type", type: "text", required: true, promptOnCreate: true, prompt: "Infrastructure type", allowedValues: CONTROLLED_VALUES.INFRASTRUCTURE_TYPES },
        { name: "platforms", type: "list[text]", required: false, promptOnCreate: true, prompt: "Platforms", allowedValues: CONTROLLED_VALUES.INFRASTRUCTURE_PLATFORMS },
        { name: "components", type: "list[text|link]", required: false, prompt: "Components", example: "Nginx Redirector, [[Cobalt Strike Team Server]]" },
        { name: "supports-playbooks", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Supported playbooks", targetContentTypes: ["Playbook"] },
        { name: "supports-tools", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Supported tools", targetContentTypes: ["Tool"] },
        { name: "supports-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Supported tradecraft", targetContentTypes: ["Tradecraft"] }
    ],
    "IOC": [
        { name: "ioc-type", type: "text", required: true, promptOnCreate: true, prompt: "IOC type", allowedValues: CONTROLLED_VALUES.IOC_TYPES },
        { name: "indicator-value", type: "text", required: true, prompt: "Indicator value", example: "185.199.110.153 or login.microsoftonline.com" },
        { name: "confidence", type: "text", required: false, promptOnCreate: true, prompt: "Confidence", allowedValues: CONTROLLED_VALUES.CONFIDENCE_LEVELS },
        { name: "associated-tools", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Associated tools", targetContentTypes: ["Tool"] },
        { name: "associated-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Associated tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "first-seen", type: "date", required: false, prompt: "First seen date" },
        { name: "last-seen", type: "date", required: false, prompt: "Last seen date" },
        { name: "active", type: "boolean", required: false, prompt: "Indicator still active" }
    ],
    "Lab Setup": [
        { name: "lab-purpose", type: "text", required: true, promptOnCreate: true, prompt: "Lab purpose", allowedValues: CONTROLLED_VALUES.LAB_PURPOSES },
        { name: "platforms", type: "list[text]", required: false, promptOnCreate: true, prompt: "Platforms", allowedValues: CONTROLLED_VALUES.LAB_PLATFORMS },
        { name: "difficulty", type: "text", required: false, promptOnCreate: true, prompt: "Difficulty", allowedValues: CONTROLLED_VALUES.DIFFICULTY_LEVELS },
        { name: "estimated-build-time", type: "text", required: false, prompt: "Estimated build time", example: "2 hours" },
        { name: "practices-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Practiced tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "uses-tools", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Used tools", targetContentTypes: ["Tool"] }
    ],
    "Offensive Code": [
        { name: "languages", type: "list[text]", required: true, promptOnCreate: true, prompt: "Languages", allowedValues: CONTROLLED_VALUES.LANGUAGES },
        { name: "entry-points", type: "list[text]", required: false, promptOnCreate: true, prompt: "Entry points", allowedValues: CONTROLLED_VALUES.ENTRY_POINTS },
        { name: "implements-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Implemented tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "targets-vulnerabilities", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Targeted vulnerabilities", targetContentTypes: ["Vulnerability"] },
        { name: "uses-protocols", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Used protocols", targetContentTypes: ["Protocol"] },
        { name: "opsec-risk", type: "text", required: false, promptOnCreate: true, prompt: "OPSEC risk", allowedValues: CONTROLLED_VALUES.OPSEC_RISK },
        { name: "platforms", type: "list[text]", required: false, promptOnCreate: true, prompt: "Platforms", allowedValues: CONTROLLED_VALUES.OFFENSIVE_CODE_PLATFORMS }
    ],
    "Playbook": [
        { name: "objective", type: "text", required: true, prompt: "Objective", example: "'Obtain domain admin from a low-privileged foothold'" },
        { name: "scope", type: "text", required: true, promptOnCreate: true, prompt: "Scope", allowedValues: CONTROLLED_VALUES.PLAYBOOK_SCOPES },
        { name: "required-tools", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Required tools", targetContentTypes: ["Tool"] },
        { name: "required-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Required tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "required-access", type: "list[text]", required: false, prompt: "Required access", example: "'Valid domain user credentials', 'VPN access'" },
        { name: "opsec-risk", type: "text", required: false, promptOnCreate: true, prompt: "OPSEC risk", allowedValues: CONTROLLED_VALUES.OPSEC_RISK },
        { name: "tested", type: "boolean", required: false, prompt: "Tested" },
        { name: "last-executed", type: "date", required: false, prompt: "Last executed date" }
    ],
    "Protocol": [
        { name: "protocol-family", type: "text", required: true, promptOnCreate: true, prompt: "Protocol family", allowedValues: CONTROLLED_VALUES.PROTOCOL_FAMILIES },
        { name: "ports", type: "list[number]", required: false, prompt: "Ports" },
        { name: "authentication-methods", type: "list[text]", required: false, promptOnCreate: true, prompt: "Authentication methods", allowedValues: CONTROLLED_VALUES.PROTOCOL_AUTHENTICATION_METHODS },
        { name: "abused-by-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Abused by tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "secured-by-controls", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Secured by controls", targetContentTypes: ["Security Control"] },
        { name: "related-tools", type: "list[link]", required: true, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Related tools", targetContentTypes: ["Tool"] }
    ],
    "Reference Material": [
        { name: "resource-type", type: "text", required: true, promptOnCreate: true, prompt: "Resource type", allowedValues: CONTROLLED_VALUES.REFERENCE_RESOURCE_TYPES },
        { name: "authors", type: "list[text]", required: false, prompt: "Authors", example: "Will Schroeder, Lee Christensen" },
        { name: "publisher", type: "text", required: false, prompt: "Publisher", example: "TrustedSec" },
        { name: "publication-date", type: "date", required: false, prompt: "Publication date" },
        { name: "covers-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Covered tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "covers-tools", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Covered tools", targetContentTypes: ["Tool"] },
        { name: "covers-platforms", type: "list[text]", required: false, prompt: "Covered platforms", example: "Windows, Active Directory" }
    ],
    "Security Control": [
        { name: "control-category", type: "text", required: true, promptOnCreate: true, prompt: "Control category", allowedValues: CONTROLLED_VALUES.CONTROL_CATEGORIES },
        { name: "protects-platforms", type: "list[text]", required: false, promptOnCreate: true, prompt: "Protected platforms", allowedValues: CONTROLLED_VALUES.PROTECTED_PLATFORMS },
        { name: "detects-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Detected tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "known-bypasses", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Known bypasses", targetContentTypes: ["Tradecraft"] },
        { name: "telemetry-sources", type: "list[text]", required: false, prompt: "Telemetry sources", example: "Sysmon Event ID 1, Windows Security 4688" },
        { name: "enforcement-points", type: "list[text]", required: false, prompt: "Enforcement points", example: "'Endpoint agent', 'Identity provider'" }
    ],
    "Study Resources": [
        { name: "resource-type", type: "text", required: true, promptOnCreate: true, prompt: "Resource type", allowedValues: CONTROLLED_VALUES.STUDY_RESOURCE_TYPES },
        { name: "provider", type: "text", required: false, prompt: "Provider", example: "OffSec" },
        { name: "status", type: "text", required: false, promptOnCreate: true, prompt: "Status", allowedValues: CONTROLLED_VALUES.CONTROLLED_PROVIDER_STATUS },
        { name: "covers-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Covered tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "covers-tools", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Covered tools", targetContentTypes: ["Tool"] },
        { name: "covers-platforms", type: "list[text]", required: false, prompt: "Covered platforms", example: "Windows, Azure" },
        { name: "completed-on", type: "date", required: false, prompt: "Completed on date" }
    ],
    "Tool": [
        { name: "tool-category", type: "text", required: true, prompt: "Tool category", example: "'Password Cracker'" },
        { name: "operating-platforms", type: "list[text]", required: false, promptOnCreate: true, prompt: "Operating platforms", allowedValues: CONTROLLED_VALUES.PRIMARY_PLATFORMS },
        { name: "target-platforms", type: "list[text]", required: false, promptOnCreate: true, prompt: "Target platforms", allowedValues: CONTROLLED_VALUES.TARGET_PLATFORMS },
        { name: "implements-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Implemented tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "used-in-playbooks", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Used in playbooks", targetContentTypes: ["Playbook"] },
        { name: "opsec-risk", type: "text", required: false, promptOnCreate: true, prompt: "OPSEC risk", allowedValues: CONTROLLED_VALUES.OPSEC_RISK },
        { name: "detection-difficulty", type: "text", required: false, promptOnCreate: true, prompt: "Detection difficulty", allowedValues: CONTROLLED_VALUES.DETECTION_DIFFICULTY },
        { name: "tested", type: "boolean", required: false, prompt: "Tested" }
    ],
    "Tradecraft": [
        { name: "attack-id", type: "text", required: false, prompt: "ATT&CK or framework ID", example: "T1059.001" },
        { name: "tactic", type: "text", required: true, promptOnCreate: true, prompt: "Tactic", allowedValues: CONTROLLED_VALUES.TACTICS },
        { name: "platforms", type: "list[text]", required: true, promptOnCreate: true, prompt: "Platforms", allowedValues: CONTROLLED_VALUES.TARGET_PLATFORMS },
        { name: "permissions-required", type: "list[text]", required: false, promptOnCreate: true, prompt: "Permissions required", allowedValues: CONTROLLED_VALUES.PERMISSIONS_REQUIRED },
        { name: "supports-remote", type: "boolean", required: false, prompt: "Supports remote" },
        { name: "data-sources", type: "list[text]", required: false, prompt: "Data sources", example: "Process, PowerShell Transcript, Windows Event Logs" },
        { name: "uses-tools", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Used tools", targetContentTypes: ["Tool"] },
        { name: "bypasses-controls", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Bypassed controls", targetContentTypes: ["Security Control"] },
        { name: "exploits-vulnerabilities", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Exploited vulnerabilities", targetContentTypes: ["Vulnerability"] },
        { name: "uses-protocols", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Used protocols", targetContentTypes: ["Protocol"] }
    ],
    "Vulnerability": [
        { name: "cve-id", type: "text", required: false, prompt: "CVE ID", example: "CVE-2021-34527" },
        { name: "cvss-score", type: "number", required: false, prompt: "CVSS score" },
        { name: "severity", type: "text", required: true, promptOnCreate: true, prompt: "Severity", allowedValues: CONTROLLED_VALUES.SEVERITY_LEVELS },
        { name: "affected-platforms", type: "list[text]", required: false, promptOnCreate: true, prompt: "Affected platforms", allowedValues: CONTROLLED_VALUES.AFFECTED_PLATFORMS },
        { name: "affects-attack-surfaces", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Affected attack surfaces", targetContentTypes: ["Attack Surface"] },
        { name: "exploited-by-tradecraft", type: "list[link]", required: false, promptOnCreate: true, allowEmptyOnCreate: true, prompt: "Exploited by tradecraft", targetContentTypes: ["Tradecraft"] },
        { name: "prerequisites", type: "list[text]", required: false, prompt: "Prerequisites", example: "'Authenticated access', 'Reachable RPC service'" }
    ]
};

//////////////////////////////////////////////////////////////////////////////////
//                              LOGGING UTILITY                                //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Enhanced logging utility with consistent formatting and debug levels
 */
const Logger = {
    /**
     * Debug-level logging for detailed operational info
     * @param {string} message - Debug message
     * @param {Object} data - Additional context data
     */
    debug: (message, data = {}) => {
        console.debug(`[DEBUG] ${message}`, data);
    },

    /**
     * Info-level logging for general operational flow
     * @param {string} message - Info message
     * @param {Object} data - Additional context data
     */
    info: (message, data = {}) => {
        console.info(`[INFO] ${message}`, data);
    },

    /**
     * Warning-level logging for non-critical issues
     * @param {string} message - Warning message
     * @param {Object} data - Additional context data
     */
    warn: (message, data = {}) => {
        console.warn(`[WARN] ${message}`, data);
    },

    /**
     * Error-level logging with full context
     * @param {string} message - Error description
     * @param {Error} error - Error object
     * @param {Object} data - Additional context data
     */
    error: (message, error, data = {}) => {
        console.error(`[ERROR] ${message}`, {
            error: error.message,
            stack: error.stack,
            timestamp: new Date().toISOString(),
            ...data
        });
    },
};

//////////////////////////////////////////////////////////////////////////////////
//                              UTILITY FUNCTIONS                              //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Shows a user notice with consistent formatting
 * @param {string} message - Message to display
 * @param {"error"|"success"|"suggestion"|"warning"} noticeType - Type of notice
 * @returns {void} - Shows a transient Obsidian notice.
 */
function showNotice(message, noticeType) {
	const emoji = EMOJI_NOTICE[noticeType.toUpperCase()] || "";
	new Notice(`${emoji} ${message}`);
}

/**
 * Displays an error notice using the vault's standard emoji and styling conventions.
 * @param {string} message - Error message to show to the user.
 * @returns {void} - Shows a transient Obsidian notice.
 */
function showError(message) { showNotice(message, "error"); }
/**
 * Displays a success notice using the vault's standard emoji and styling conventions.
 * @param {string} message - Success message to show to the user.
 * @returns {void} - Shows a transient Obsidian notice.
 */
function showSuccess(message) { showNotice(message, "success"); }
/**
 * Displays a suggestion notice using the vault's standard emoji and styling conventions.
 * @param {string} message - Suggestion message to show to the user.
 * @returns {void} - Shows a transient Obsidian notice.
 */
function showSuggestion(message) { showNotice(message, "suggestion"); }
/**
 * Displays a warning notice using the vault's standard emoji and styling conventions.
 * @param {string} message - Warning message to show to the user.
 * @returns {void} - Shows a transient Obsidian notice.
 */
function showWarning(message) { showNotice(message, "warning"); }

/**
 * Converts a relative template path into the Obsidian include-link syntax expected by Templater.
 * @param {string} relativePathWithoutExtension - Relative vault path without the `.md` extension.
 * @returns {string} - Obsidian include link for `tp.file.include`.
 */
function createTemplateIncludePath(relativePathWithoutExtension) {
    return `[[${relativePathWithoutExtension}]]`;
}

/**
 * Builds the full set of template include paths for a content type's component files.
 * @param {string} contentTypeName - Content type folder name under the content templates directory.
 * @returns {Object} - Object containing metadataTemplate, bodyTemplate, dataviewTemplate, and footerTemplate paths.
 */
function createContentTemplatePaths(contentTypeName) {
    const basePath = `${PATHS.CONTENT_TEMPLATES}/${contentTypeName}`;
    return {
        metadataTemplate: createTemplateIncludePath(`${basePath}/Metadata`),
        bodyTemplate: createTemplateIncludePath(`${basePath}/Body`),
        dataviewTemplate: createTemplateIncludePath(`${basePath}/Dataview`),
        footerTemplate: createTemplateIncludePath(`${basePath}/Footer`)
    };
}

/**
 * Resolves the effective schema for a content type using the built-in in-script schema definitions.
 * @param {string} contentTypeName - Content type name to resolve.
 * @returns {Object} - Resolved schema object with universal, type-specific, and combined properties.
 */
function getContentTypePropertySchema(contentTypeName) {
    return buildResolvedContentTypeSchema(contentTypeName);
}

/**
 * Builds the final schema used by the note-creation workflow from built-in definitions.
 * @param {string} contentTypeName - Content type whose schema should be resolved.
 * @returns {Object} - Resolved schema object with universal, typeSpecific, allProperties, and metadata.
 */
function buildResolvedContentTypeSchema(contentTypeName) {
    const typeSpecificProperties = CONTENT_TYPE_PROPERTY_SCHEMAS[contentTypeName] || [];

    if (!CONTENT_TYPE_PROPERTY_SCHEMAS[contentTypeName]) {
        Logger.warn("No content type schema found, defaulting to universal properties only", {
            contentTypeName,
            availableSchemas: Object.keys(CONTENT_TYPE_PROPERTY_SCHEMAS)
        });
    }

    return {
        universal: UNIVERSAL_CONTENT_PROPERTIES,
        typeSpecific: typeSpecificProperties,
        allProperties: [...UNIVERSAL_CONTENT_PROPERTIES, ...typeSpecificProperties]
    };
}

/**
 * Escapes a string so it can be safely interpolated into a regular expression.
 * @param {string} value - Raw string that may contain regex metacharacters.
 * @returns {string} - Regex-safe version of the input string.
 */
function escapeRegex(value) {
    return value.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

/**
 * Detects the dominant line ending used by a block of text.
 * @param {string} text - Text whose newline style should be inspected.
 * @returns {"\n"|"\r\n"} - Detected line ending, defaulting to LF.
 */
function detectLineEnding(text = "") {
    return text.includes("\r\n") ? "\r\n" : "\n";
}

/**
 * Serializes a single property value for safe YAML frontmatter output.
 * @param {*} value - Raw scalar value to serialize.
 * @param {string} propertyType - Property type hint used to format booleans, numbers, and links.
 * @returns {string} - YAML-safe scalar representation.
 */
function formatYamlScalar(value, propertyType = "text") {
    if (value === null || value === undefined || value === "") {
        return "";
    }

    if (propertyType === "boolean" || propertyType === "number") {
        return String(value);
    }

    if (typeof value === "string" && value.startsWith("[[")) {
        return `"${value}"`;
    }

    return String(value);
}

/**
 * Serializes an array of values into YAML list syntax.
 * @param {Array} values - Array of property values to serialize.
 * @param {string} propertyType - Property type hint used to format each list item.
 * @returns {string} - YAML list block ready for insertion into frontmatter.
 */
function formatYamlList(values, propertyType = "list[text]") {
    if (!Array.isArray(values) || values.length === 0) {
        return "  - ";
    }

    return values
        .map(value => `  - ${formatYamlScalar(value, propertyType)}`)
        .join("\n");
}

/**
 * Ensures that a scalar frontmatter property exists with the provided value, updating or inserting it as needed.
 * @param {string} metadata - Raw frontmatter block to modify.
 * @param {string} propertyName - Property name to insert or update.
 * @param {*} propertyValue - Value to serialize into the property.
 * @param {"\n"|"\r\n"} lineEnding - Newline style to preserve when mutating the block.
 * @returns {string} - Updated frontmatter block containing the requested property.
 */
function ensureMetadataProperty(metadata, propertyName, propertyValue, lineEnding = "\n") {
    const scalarPattern = new RegExp(`^${escapeRegex(propertyName)}:.*$`, "m");
    const serializedValue = formatYamlScalar(propertyValue, "text");

    if (scalarPattern.test(metadata)) {
        return metadata.replace(scalarPattern, `${propertyName}: ${serializedValue}`);
    }

    const closingDelimiter = metadata.lastIndexOf(`${lineEnding}---`);
    if (closingDelimiter === -1) {
        throw new Error(`Unable to insert metadata property "${propertyName}" because the closing frontmatter delimiter was not found`);
    }

    return [
        metadata.slice(0, closingDelimiter),
        `${lineEnding}${propertyName}: ${serializedValue}`,
        metadata.slice(closingDelimiter)
    ].join("");
}

/**
 * Determines whether a property should be prompted during note creation.
 * @param {Object} propertyDefinition - Normalized property definition from the resolved schema.
 * @returns {boolean} - True when the property should appear in the creation workflow.
 */
function shouldPromptOnCreate(propertyDefinition) {
    return propertyDefinition?.promptOnCreate === true || propertyDefinition?.required === true;
}

/**
 * Determines whether a prompted property may be left empty during note creation.
 * @param {Object} propertyDefinition - Normalized property definition from the resolved schema.
 * @returns {boolean} - True when the workflow may preserve placeholders instead of requiring a value.
 */
function canLeaveEmptyOnCreate(propertyDefinition) {
    return propertyDefinition?.allowEmptyOnCreate === true;
}

/**
 * Normalizes unknown thrown values into a readable error message string.
 * @param {*} error - Thrown error object or arbitrary value.
 * @returns {string} - Human-readable error message.
 */
function getErrorMessage(error) {
    if (error instanceof Error) {
        return error.message;
    }

    return String(error || "Unknown error");
}

/**
 * Detects whether an error represents a user cancellation rather than an unexpected workflow failure.
 * @param {*} error - Thrown error object or arbitrary value.
 * @returns {boolean} - True when the error message matches a known cancellation path.
 */
function isCancellationError(error) {
    const message = getErrorMessage(error);
    return message === "User cancelled input" || message === "Operation cancelled by user";
}

/**
 * Summarizes prompted-property results for completion notices and recovery content.
 * @param {Object} summary - Raw prompt summary containing prompted, provided, and preserved property arrays.
 * @returns {Object} - Normalized counts and property-name arrays for downstream messaging.
 */
function summarizePromptedPropertyOutcomes(summary = {}) {
    const provided = Array.isArray(summary.providedProperties) ? summary.providedProperties : [];
    const preserved = Array.isArray(summary.preservedPlaceholderProperties) ? summary.preservedPlaceholderProperties : [];
    const promptCount = Number.isFinite(summary.promptedPropertyCount) ? summary.promptedPropertyCount : provided.length + preserved.length;

    return {
        promptCount,
        providedCount: provided.length,
        preservedCount: preserved.length,
        providedProperties: provided,
        preservedPlaceholderProperties: preserved
    };
}

/**
 * Filters a resolved schema down to the type-specific properties that should prompt during creation.
 * @param {Object} schema - Resolved content type schema.
 * @returns {Object[]} - Array of type-specific property definitions that should prompt on create.
 */
function getCreateTimePromptProperties(schema) {
    return (schema?.typeSpecific || []).filter(property => shouldPromptOnCreate(property));
}

/**
 * Builds a quick lookup table from property name to property definition.
 * @param {Object} schema - Resolved schema containing `allProperties`.
 * @returns {Object} - Plain object keyed by property name.
 */
function buildSchemaLookup(schema) {
    return Object.fromEntries((schema?.allProperties || []).map(property => [property.name, property]));
}

/**
 * Determines whether a property type represents a wiki-link-aware field.
 * @param {string} propertyType - Schema property type string.
 * @returns {boolean} - True when the property type includes links.
 */
function isLinkProperty(propertyType = "") {
    return propertyType.includes("link");
}

/**
 * Validates whether a string matches the vault's expected date format.
 * @param {string} value - Candidate date string.
 * @returns {boolean} - True when the string matches `YYYY-MM-DD`.
 */
function isDateString(value) {
    return /^\d{4}-\d{2}-\d{2}$/.test(value);
}

/**
 * Validates whether a string matches the vault's expected date-time format.
 * @param {string} value - Candidate datetime string.
 * @returns {boolean} - True when the string matches the supported datetime pattern.
 */
function isDateTimeString(value) {
    return /^\d{4}-\d{2}-\d{2}[ T]\d{2}:\d{2}(:\d{2})?$/.test(value);
}

/**
 * Normalizes a value into Obsidian wiki-link syntax when appropriate.
 * @param {*} value - Raw property value supplied by the user or schema.
 * @returns {*} - Original value or normalized wiki-link string.
 */
function normalizeWikiLink(value) {
    if (typeof value !== "string") {
        return value;
    }

    const trimmedValue = value.trim();
    if (trimmedValue === "") {
        return trimmedValue;
    }

    if (trimmedValue.startsWith("[[") && trimmedValue.endsWith("]]")) {
        return `[[${trimmedValue.slice(2, -2).trim()}]]`;
    }

    return `[[${trimmedValue}]]`;
}

/**
 * Normalizes a prompted property value according to its schema definition before validation.
 * @param {Object} propertyDefinition - Schema definition describing the property.
 * @param {*} value - Raw prompted value to normalize.
 * @returns {*} - Normalized scalar or list value ready for validation.
 */
function normalizePropertyValue(propertyDefinition, value) {
    if (value === null || value === undefined) {
        return value;
    }

    if (Array.isArray(value)) {
        const normalizedList = value
            .map(item => normalizePropertyValue({ ...propertyDefinition, type: propertyDefinition.type.replace(/^list\[(.+)\]$/, "$1") }, item))
            .filter(item => item !== null && item !== undefined && item !== "");

        return [...new Set(normalizedList)];
    }

    if (propertyDefinition.type === "text" || propertyDefinition.type === "date" || propertyDefinition.type === "datetime") {
        if (typeof value === "string") {
            return value.trim();
        }
    }

    if (propertyDefinition.type === "number") {
        return typeof value === "number" ? value : Number(value);
    }

    if (propertyDefinition.type === "boolean") {
        if (typeof value === "boolean") {
            return value;
        }

        if (typeof value === "string") {
            const normalizedBoolean = value.trim().toLowerCase();
            if (normalizedBoolean === "true") return true;
            if (normalizedBoolean === "false") return false;
        }
    }

    if (isLinkProperty(propertyDefinition.type)) {
        return normalizeWikiLink(value);
    }

    return value;
}

/**
 * Validates a normalized property value against its schema definition.
 * @param {Object} propertyDefinition - Schema definition for the property being checked.
 * @param {*} value - Normalized value to validate.
 * @returns {Object} - Validation result object with isValid, error, suggestion, and canProceedAnyway.
 */
function validatePropertyValueAgainstDefinition(propertyDefinition, value) {
    const propertyLabel = propertyDefinition.prompt || propertyDefinition.name;

    if (propertyDefinition.required) {
        const isMissing = value === null || value === undefined || value === "" || (Array.isArray(value) && value.length === 0);
        if (isMissing) {
            throw new Error(`${propertyLabel} is required but no value was provided`);
        }
    }

    if (value === null || value === undefined || value === "") {
        return;
    }

    if (propertyDefinition.type.startsWith("list[")) {
        if (!Array.isArray(value)) {
            throw new Error(`${propertyLabel} must be a list`);
        }

        value.forEach(item => validatePropertyValueAgainstDefinition(
            { ...propertyDefinition, type: propertyDefinition.type.replace(/^list\[(.+)\]$/, "$1"), required: false },
            item
        ));
        return;
    }

    if (propertyDefinition.type === "boolean" && typeof value !== "boolean") {
        throw new Error(`${propertyLabel} must be true or false`);
    }

    if (propertyDefinition.type === "number" && (!Number.isFinite(value) || Number.isNaN(value))) {
        throw new Error(`${propertyLabel} must be a valid number`);
    }

    if (propertyDefinition.type === "date" && (typeof value !== "string" || !isDateString(value))) {
        throw new Error(`${propertyLabel} must use YYYY-MM-DD format`);
    }

    if (propertyDefinition.type === "datetime" && (typeof value !== "string" || !isDateTimeString(value))) {
        throw new Error(`${propertyLabel} must use YYYY-MM-DD HH:mm or YYYY-MM-DDTHH:mm format`);
    }

    if (isLinkProperty(propertyDefinition.type) && (typeof value !== "string" || !/^\[\[[^\]]+\]\]$/.test(value))) {
        throw new Error(`${propertyLabel} must be a valid Obsidian wiki-link`);
    }

    if (Array.isArray(propertyDefinition.allowedValues) && propertyDefinition.allowedValues.length > 0) {
        if (!propertyDefinition.allowedValues.includes(value)) {
            throw new Error(`${propertyLabel} must match one of the allowed values`);
        }
    }
}

/**
 * Normalizes and validates every prompted property before frontmatter assembly.
 * @param {Object} schema - Resolved schema containing all property definitions for the content type.
 * @param {Object} promptedProperties - Raw prompted-property map keyed by property name.
 * @returns {Object} - Validated and normalized prompted-property map.
 */
function normalizeAndValidatePromptedProperties(schema, promptedProperties = {}) {
    const schemaLookup = buildSchemaLookup(schema);
    const normalizedProperties = {};

    for (const [propertyName, propertyValue] of Object.entries(promptedProperties)) {
        const propertyDefinition = schemaLookup[propertyName];

        if (!propertyDefinition) {
            throw new Error(`No schema definition found for prompted property "${propertyName}"`);
        }

        const normalizedValue = normalizePropertyValue(propertyDefinition, propertyValue);
        validatePropertyValueAgainstDefinition(propertyDefinition, normalizedValue);
        normalizedProperties[propertyName] = normalizedValue;
    }

    return normalizedProperties;
}

/**
 * Guards against malformed frontmatter by checking that both opening and closing delimiters are present.
 * @param {string} metadata - Raw frontmatter text to inspect.
 * @returns {void} - Throws if frontmatter delimiters are missing or malformed.
 */
function assertFrontmatterIntegrity(metadata) {
    if (typeof metadata !== "string" || !metadata.startsWith("---")) {
        throw new Error("Metadata template is malformed: missing opening frontmatter delimiter");
    }

    const closingDelimiterIndex = metadata.indexOf("\n---", 3);
    if (closingDelimiterIndex === -1) {
        throw new Error("Metadata template is malformed: missing closing frontmatter delimiter");
    }
}

/**
 * Creates a reusable non-empty validation function for prompt inputs.
 * @param {string} promptLabel - Friendly property label used in returned validation messages.
 * @returns {Function} - Async validation function that returns a standard validation result object.
 */
function createNonEmptyValidation(promptLabel) {
    return async (input) => {
        if (!input || input.trim() === "") {
            return createValidationResult(
                false,
                `${promptLabel} cannot be empty`,
                `Provide a value for ${promptLabel.toLowerCase()}`
            );
        }

        return createValidationResult(true);
    };
}

/**
 * Builds consistent prompt text for scalar and list property collection.
 * @param {string} action - Leading action text such as "Enter" or "Select".
 * @param {Object} propertyDefinition - Property definition supplying prompt text and examples.
 * @param {boolean} listMode - Whether the prompt is being built for a list value.
 * @returns {string} - Fully formatted prompt string for Templater input dialogs.
 */
function formatPromptText(action, propertyDefinition, listMode = false) {
    const basePrompt = listMode
        ? `${action} ${propertyDefinition.prompt}`
        : `${action} ${propertyDefinition.prompt}`;

    if (!propertyDefinition.example) {
        return basePrompt;
    }

    return `${basePrompt} (e.g., ${propertyDefinition.example})`;
}

/**
 * Prompts the user for a required scalar property and validates the result.
 * @param {Object} propertyDefinition - Schema definition for the scalar property being collected.
 * @returns {Promise<*>} - Validated scalar property value.
 */
async function promptRequiredScalarProperty(propertyDefinition) {
    const { prompt, allowedValues, type } = propertyDefinition;

    if (Array.isArray(allowedValues) && allowedValues.length > 0) {
        while (true) {
            const selectedValue = await tp.system.suggester(
                allowedValues,
                allowedValues,
                false,
                `Select ${prompt}:`
            );

            if (selectedValue) {
                return selectedValue;
            }

            showWarning(`${prompt} is required`);
        }
    }

    const input = await retryWithValidation(
        formatPromptText("Enter", propertyDefinition),
        createNonEmptyValidation(prompt),
        VALIDATION_LIMITS.MAX_VALIDATION_ATTEMPTS
    );

    if (type === "number") {
        return Number(input);
    }

    if (type === "boolean") {
        return String(input).toLowerCase() === "true";
    }

    return input.trim();
}

/**
 * Reads a content note's `type` frontmatter value from a vault file object.
 * @param {TFile|Object} file - Obsidian file object to inspect.
 * @returns {string|null} - Content type string when available, otherwise null.
 */
function getContentNoteType(file) {
    const frontmatter = app.metadataCache.getFileCache(file)?.frontmatter;

    if (typeof frontmatter?.type === "string" && frontmatter.type.trim() !== "") {
        return frontmatter.type.trim();
    }

    return null;
}

/**
 * Gathers eligible content-note choices for a typed link property picker.
 * @param {Object} propertyDefinition - Property definition containing target content type filters.
 * @returns {string[]} - Ordered array of wiki-link strings eligible for selection.
 */
function getEligibleContentNoteChoices(propertyDefinition) {
    const targetContentTypes = propertyDefinition.targetContentTypes || [];

    if (!Array.isArray(targetContentTypes) || targetContentTypes.length === 0) {
        return [];
    }

    const normalizedContentPath = `${PATHS.CONTENT}/`;

    return app.vault
        .getMarkdownFiles()
        .filter(file => file.path.startsWith(normalizedContentPath))
        .filter(file => {
            const contentType = getContentNoteType(file);
            return targetContentTypes.includes(contentType);
        })
        .map(file => file.basename)
        .sort((a, b) => a.localeCompare(b));
}

/**
 * Summarizes which content note types are currently available in the vault for troubleshooting prompts.
 * @returns {string} - Comma-separated summary of available content note types.
 */
function summarizeAvailableContentTypes() {
    const normalizedContentPath = `${PATHS.CONTENT}/`;
    const typeCounts = {};

    app.vault
        .getMarkdownFiles()
        .filter(file => file.path.startsWith(normalizedContentPath))
        .forEach(file => {
            const contentType = getContentNoteType(file);
            if (!contentType) {
                return;
            }

            typeCounts[contentType] = (typeCounts[contentType] || 0) + 1;
        });

    return Object.entries(typeCounts)
        .sort((a, b) => a[0].localeCompare(b[0]))
        .map(([type, count]) => `${type} (${count})`)
        .join(", ");
}

/**
 * Builds a stable multi-select option list that preserves ordering while toggling selected state.
 * @param {string[]} allValues - Full ordered set of selectable values.
 * @param {string[]} selectedValues - Values currently selected by the user.
 * @returns {Object} - Object containing displayOptions and optionValues arrays for Templater suggesters.
 */
function buildStableMultiSelectOptions(allValues, selectedValues) {
    const displayOptions = [];
    const optionValues = [];

    allValues.forEach(value => {
        if (selectedValues.includes(value)) {
            displayOptions.push(`❌ ${value}`);
            optionValues.push(`REMOVE_${value}`);
            return;
        }

        displayOptions.push(`🔳 ${value}`);
        optionValues.push(value);
    });

    displayOptions.push("✅ Done");
    optionValues.push("__DONE__");

    return { displayOptions, optionValues };
}

/**
 * Prompts the user to select one or more linked notes for a typed link-list property.
 * @param {Object} propertyDefinition - Property definition describing target note types and prompt text.
 * @returns {Promise<string[]|null>} - Selected wiki-link values, or null when placeholders should be preserved.
 */
async function promptLinkedNoteListProperty(propertyDefinition) {
    const { prompt, targetContentTypes = [] } = propertyDefinition;
    const availableNotes = getEligibleContentNoteChoices(propertyDefinition);

    if (availableNotes.length === 0) {
        if (!canLeaveEmptyOnCreate(propertyDefinition)) {
            const availableTypeSummary = summarizeAvailableContentTypes() || "none";
            throw new Error(
                `No eligible notes were found in ${PATHS.CONTENT} for ${prompt}. ` +
                `Create at least one ${targetContentTypes.join(" or ")} note before creating this note. ` +
                `Currently available content note types: ${availableTypeSummary}.`
            );
        }

        const availableTypeSummary = summarizeAvailableContentTypes() || "none";
        showWarning(
            `No eligible notes were found for ${prompt}. Leaving the template placeholder in place. ` +
            `Currently available content note types: ${availableTypeSummary}.`
        );
        return null;
    }

    const selectedValues = [];

    while (true) {
        const { displayOptions, optionValues } = buildStableMultiSelectOptions(availableNotes, selectedValues);

        const selectedValue = await tp.system.suggester(
            displayOptions,
            optionValues,
            false,
            selectedValues.length === 0
                ? `Select one or more notes for ${prompt}:`
                : `Selected: ${selectedValues.join(", ")}. Add, remove, or finish ${prompt}:`
        );

        if (selectedValue === "__DONE__") {
            if (!canLeaveEmptyOnCreate(propertyDefinition) && selectedValues.length === 0) {
                showWarning(`${prompt} requires at least one linked note`);
                continue;
            }

            return selectedValues.length > 0 ? selectedValues : null;
        }

        if (!selectedValue) {
            if (!canLeaveEmptyOnCreate(propertyDefinition) && selectedValues.length === 0) {
                showWarning(`${prompt} requires at least one linked note`);
                continue;
            }

            return selectedValues.length > 0 ? selectedValues : null;
        }

        if (selectedValue.startsWith("REMOVE_")) {
            const valueToRemove = selectedValue.substring(7);
            const index = selectedValues.indexOf(valueToRemove);

            if (index > -1) {
                selectedValues.splice(index, 1);
                showSuccess(`Removed ${valueToRemove} from ${prompt}`);
            }

            continue;
        }

        selectedValues.push(selectedValue);

        if (selectedValues.length === availableNotes.length) {
            return selectedValues;
        }
    }
}

/**
 * Prompts the user for a required list property, supporting controlled vocabularies and free-write lists.
 * @param {Object} propertyDefinition - Schema definition for the list property being collected.
 * @returns {Promise<Array|null>} - Validated list value or null when allowed for placeholder preservation.
 */
async function promptRequiredListProperty(propertyDefinition) {
    const { prompt, allowedValues } = propertyDefinition;

    if (propertyDefinition.type === "list[link]" && Array.isArray(propertyDefinition.targetContentTypes)) {
        return await promptLinkedNoteListProperty(propertyDefinition);
    }

    if (Array.isArray(allowedValues) && allowedValues.length > 0) {
        const selectedValues = [];
        let continueSelecting = true;

        while (continueSelecting) {
            const { displayOptions, optionValues } = buildStableMultiSelectOptions(allowedValues, selectedValues);

            const selectedValue = await tp.system.suggester(
                displayOptions,
                optionValues,
                false,
                selectedValues.length === 0
                    ? `Select one or more values for ${prompt}:`
                    : `Selected: ${selectedValues.join(", ")}. Add, remove, or finish ${prompt}:`
            );

            if (selectedValue === "__DONE__") {
                if (selectedValues.length > 0) {
                    return selectedValues;
                }

                showWarning(`${prompt} requires at least one value`);
                continue;
            }

            if (!selectedValue) {
                showWarning(`${prompt} requires at least one value`);
                continue;
            }

            if (selectedValue.startsWith("REMOVE_")) {
                const valueToRemove = selectedValue.substring(7);
                const index = selectedValues.indexOf(valueToRemove);

                if (index > -1) {
                    selectedValues.splice(index, 1);
                    showSuccess(`Removed ${valueToRemove} from ${prompt}`);
                }

                continue;
            }

            selectedValues.push(selectedValue);
            if (selectedValues.length === allowedValues.length) {
                return selectedValues;
            }
        }
    }

    const input = await retryWithValidation(
        formatPromptText("Enter one or more comma-separated values for", propertyDefinition, true),
        createNonEmptyValidation(prompt),
        VALIDATION_LIMITS.MAX_VALIDATION_ATTEMPTS
    );

    return input
        .split(",")
        .map(value => value.trim())
        .filter(Boolean);
}

/**
 * Routes a required property prompt to the correct scalar or list collector.
 * @param {Object} propertyDefinition - Schema definition for the property being collected.
 * @returns {Promise<*>} - Prompted property value appropriate to the property type.
 */
async function promptRequiredPropertyValue(propertyDefinition) {
    Logger.debug("Prompting for required property", {
        propertyName: propertyDefinition.name,
        propertyType: propertyDefinition.type,
        prompt: propertyDefinition.prompt
    });

    if (propertyDefinition.type.startsWith("list[")) {
        return await promptRequiredListProperty(propertyDefinition);
    }

    return await promptRequiredScalarProperty(propertyDefinition);
}

/**
 * Prompts for the short Overview description used when creating primary and secondary category notes.
 * @param {string} noteType - Category note type constant.
 * @param {string} title - Category title for contextual prompt text.
 * @returns {Promise<string>} - Validated 1-2 sentence description for the category overview section.
 */
async function promptCategoryDescription(noteType, title) {
    const categoryLabel = noteType === NOTE_TYPES.PRIMARY ? "primary category" : "secondary category";
    const promptMessage = `Enter a 1-2 sentence description for "${title}"`;

    const input = await retryWithValidation(
        promptMessage,
        async (value) => {
            const trimmedValue = value.trim();
            if (!trimmedValue) {
                return {
                    isValid: false,
                    error: "Category description cannot be empty.",
                    suggestion: "Write a short overview that explains what this category covers."
                };
            }

            const sentences = trimmedValue
                .split(/[.!?]+/)
                .map(sentence => sentence.trim())
                .filter(Boolean);

            if (sentences.length < 1 || sentences.length > 2) {
                return {
                    isValid: false,
                    error: "Category descriptions should be 1-2 sentences.",
                    suggestion: "Keep the overview short and scannable for category landing pages."
                };
            }

            return { isValid: true };
        },
        VALIDATION_LIMITS.MAX_VALIDATION_ATTEMPTS
    );

    return input.trim();
}

/**
 * Prompts for a typed link property using the content-note picker workflow.
 * @param {Object} propertyDefinition - Schema definition for the typed link property.
 * @returns {Promise<string[]|null>} - Selected linked notes or null when placeholders should be preserved.
 */
async function promptTypedLinkPropertyValue(propertyDefinition) {
    Logger.debug("Prompting for typed link property", {
        propertyName: propertyDefinition.name,
        propertyType: propertyDefinition.type,
        prompt: propertyDefinition.prompt,
        targetContentTypes: propertyDefinition.targetContentTypes || []
    });

    return await promptLinkedNoteListProperty(propertyDefinition);
}

/**
 * Collects, normalizes, and summarizes all create-time prompted properties for a content note.
 * @param {Object} contentTypeConfig - Selected content type configuration object.
 * @returns {Promise<Object>} - Object containing resolved schema, normalized values, and prompt outcome summary.
 */
async function collectPromptedContentProperties(contentTypeConfig) {
    const schema = buildResolvedContentTypeSchema(contentTypeConfig.name);
    const promptProperties = getCreateTimePromptProperties(schema);
    const collectedValues = {};
    const providedProperties = [];
    const preservedPlaceholderProperties = [];

    Logger.info("Collecting prompted content properties", {
        contentTypeName: contentTypeConfig.name,
        promptPropertyCount: promptProperties.length,
        promptPropertyNames: promptProperties.map(property => property.name)
    });

    for (const propertyDefinition of promptProperties) {
        const propertyValue =
            propertyDefinition.type === "list[link]" && Array.isArray(propertyDefinition.targetContentTypes)
                ? await promptTypedLinkPropertyValue(propertyDefinition)
                : await promptRequiredPropertyValue(propertyDefinition);

        if (propertyValue !== null && propertyValue !== undefined) {
            collectedValues[propertyDefinition.name] = propertyValue;
            providedProperties.push(propertyDefinition.name);
            continue;
        }

        preservedPlaceholderProperties.push(propertyDefinition.name);
    }

    const normalizedValues = normalizeAndValidatePromptedProperties(schema, collectedValues);

    return {
        schema,
        values: normalizedValues,
        summary: {
            promptedPropertyCount: promptProperties.length,
            providedProperties,
            preservedPlaceholderProperties
        }
    };
}

/**
 * Applies prompted property values into a frontmatter template by replacing scalar and list placeholders.
 * @param {string} metadata - Raw metadata template content.
 * @param {Object} promptedProperties - Normalized prompted-property map keyed by property name.
 * @param {Object|null} schema - Resolved schema used to determine property formatting.
 * @returns {string} - Updated metadata block with prompted values inserted.
 */
function applyPromptedPropertyValues(metadata, promptedProperties = {}, schema = null) {
    assertFrontmatterIntegrity(metadata);

    const schemaLookup = buildSchemaLookup(schema);
    const lineEnding = detectLineEnding(metadata);
    let updatedMetadata = metadata;

    for (const [propertyName, propertyValue] of Object.entries(promptedProperties)) {
        const propertyDefinition = schemaLookup[propertyName];
        if (!propertyDefinition) {
            throw new Error(`No schema definition found for property "${propertyName}" during metadata customization`);
        }

        const escapedPropertyName = escapeRegex(propertyName);
        const isListValue = Array.isArray(propertyValue);

        if (isListValue) {
            const listPattern = new RegExp(`^${escapedPropertyName}:\\s*\\r?\\n(?:[ \\t]{2}-.*(?:\\r?\\n|$))+`, "m");
            if (!listPattern.test(updatedMetadata)) {
                throw new Error(`Unable to find list placeholder for property "${propertyName}" in metadata template`);
            }

            const formattedList = formatYamlList(propertyValue, propertyDefinition.type).replace(/\n/g, lineEnding);
            const replacement = `${propertyName}:${lineEnding}${formattedList}${lineEnding}`;
            updatedMetadata = updatedMetadata.replace(listPattern, replacement);
            continue;
        }

        const scalarPattern = new RegExp(`^${escapedPropertyName}:.*$`, "m");
        if (!scalarPattern.test(updatedMetadata)) {
            throw new Error(`Unable to find scalar placeholder for property "${propertyName}" in metadata template`);
        }

        updatedMetadata = updatedMetadata.replace(
            scalarPattern,
            `${propertyName}: ${formatYamlScalar(propertyValue, propertyDefinition.type)}`
        );
    }

    assertFrontmatterIntegrity(updatedMetadata);
    return updatedMetadata;
}

/**
 * Loads an optional template component and degrades gracefully when the file is missing.
 * @param {string|null} templatePath - Include path for the template component.
 * @param {string} componentName - Friendly component name used for logging and notices.
 * @returns {Promise<string>} - Template content string or an empty string when unavailable.
 */
async function loadOptionalTemplate(templatePath, componentName) {
    if (!templatePath) {
        Logger.debug(`Skipping optional ${componentName} template because no path was provided`);
        return "";
    }

    try {
        Logger.debug(`Loading optional ${componentName} template`, { templatePath });
        const content = await tp.file.include(templatePath);
        Logger.debug(`Optional ${componentName} template loaded (${content.length} chars)`);
        return content;
    } catch (error) {
        Logger.warn(`Optional ${componentName} template could not be loaded; continuing without it`, {
            templatePath,
            error: error.message
        });
        return "";
    }
}

/**
 * Creates a standardized validation result object
 * @param {boolean} isValid - Whether the validation passed
 * @param {string} error - Error message if validation failed
 * @param {string} suggestion - Helpful suggestion for fixing the error
 * @param {boolean} canProceedAnyway - Whether user can override this validation
 * @returns {Object} Validation result object with isValid, error, suggestion, canProceedAnyway properties
 */
function createValidationResult(isValid, error = null, suggestion = null, canProceedAnyway = false) {
    return { isValid, error, suggestion, canProceedAnyway };
}

/**
 * Creates a standardized file path based on note type and title
 * @param {string} noteType - Type of note (Primary, Secondary, Content, or Content Type)
 * @param {string} title - Note title
 * @returns {string} - Full file path
 */
function createDestinationPath(noteType, title) {
	try {
        // Validate noteType input; allow title input to be empty or null, as title validation occurs later
        if (!noteType || typeof noteType !== 'string') {
            throw new Error(`Invalid note type: expected string, got ${typeof noteType}`);
        }

		const pathMap = {
	        [NOTE_TYPES.PRIMARY]: `${PATHS.PRIMARY_CATEGORIES}/${title}.md`,
	        [NOTE_TYPES.SECONDARY]: `${PATHS.SECONDARY_CATEGORIES}/${title}.md`,
	        [NOTE_TYPES.CONTENT]: `${PATHS.CONTENT}/${title}.md`,
	        [NOTE_TYPES.CONTENT_TYPE]: `${PATHS.CONTENT_TEMPLATES}/${title}`
	    };
	    
	    const path = pathMap[noteType];
	    if (!path) {
            const validTypes = Object.keys(pathMap);
            throw new Error(`Unknown note type: "${noteType}". Valid types: ${validTypes.join(', ')}`);
	    }

        Logger.debug("Destination path created", { noteType, title, path });
        return path;

    } catch (error) {
        Logger.error("Failed to create destination path", error, { noteType, title });
        throw error; // Re-throw with context preserved
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                          VALIDATION FUNCTIONS                               //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Generic validation retry function with user-friendly error handling
 * @param {string} promptMessage - Message to show in the user prompt
 * @param {Function} validationFn - Async function that validates input and returns validation result object
 * @param {number} maxAttempts - Maximum number of validation attempts before fallback
 * @param {string|null} fallbackValue - Fallback value to use if all attempts fail (null for no fallback)
 * @returns {Promise<string>} - Validated user input or fallback value
 * @throws {Error} - If user cancels or no fallback available after max attempts
 */
async function retryWithValidation(promptMessage, validationFn, maxAttempts = VALIDATION_LIMITS.MAX_VALIDATION_ATTEMPTS, fallbackValue = null) {
	Logger.info(`Starting validation retry loop`, {
        promptMessage,
        maxAttempts,
        hasFallback: fallbackValue !== null
    });

    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
	    Logger.debug(`Validation attempt ${attempt}/${maxAttempts}`);
	    
        const remainingAttempts = maxAttempts - attempt + 1;
        const prompt = attempt < maxAttempts 
            ? `${promptMessage} (${remainingAttempts} attempts remaining):`
            : `Last chance - ${promptMessage}:`;
        
        const input = await tp.system.prompt(prompt);
        
        if (input === null) {
	        Logger.warn(`User cancelled input on attempt ${attempt}`);
            if (fallbackValue !== null) {
	            Logger.info(`Using fallback value: ${fallbackValue}`);
                showWarning(`Using fallback value: ${fallbackValue}`);
                return fallbackValue;
            }
            throw new Error("User cancelled input");
        }

		Logger.debug(`Validating input: "${input}"`);
        const validation = await validationFn(input);
        
        if (validation.isValid) {
	        Logger.info(`Validation successful on attempt ${attempt}`, { input });
			return input;
        }

		Logger.warn(`Validation failed on attempt ${attempt}`, {
            input,
            error: validation.error,
            suggestion: validation.suggestion,
            canProceedAnyway: validation.canProceedAnyway
        });
        
        showError(validation.error);
        if (validation.suggestion) {
            showSuggestion(validation.suggestion);
        }
        
        // Special handling for warnings that can be overridden
        if (validation.canProceedAnyway) {
	        Logger.debug("Offering user option to proceed anyway");
            const shouldProceed = await tp.system.suggester(
                ["✅ Use anyway (may cause issues)", "❌ Try different input"],
                [true, false],
                false,
                `Continue with "${input}"?`
            );
            
            if (shouldProceed) {
                Logger.warn(`User chose to proceed with problematic input: ${input}`);
				showWarning(`Proceeding with potentially problematic input: ${input}`);
                return input;
            }
        }
    }
    
    return await handleMaxAttemptsExceeded(promptMessage, fallbackValue, validationFn);
}

/**
 * Handles the case when maximum validation attempts are exceeded
 * @param {string} promptMessage - Original prompt message for context in final attempt
 * @param {string|null} fallbackValue - Fallback value to offer user (null if none available)
 * @returns {Promise<string>} - Final input value, fallback value, or throws error
 * @throws {Error} - If user cancels operation and no fallback available
 */
async function handleMaxAttemptsExceeded(promptMessage, fallbackValue, validationFn) {
	Logger.warn("Maximum validation attempts exceeded", {
        promptMessage,
        hasFallback: fallbackValue !== null,
        fallbackValue: fallbackValue
    });

    showError("Maximum validation attempts reached");

    const options = ["🔄 Try one more time"];
    const values = ["retry"];
    
    if (fallbackValue !== null) {
        options.push(`📁 Use fallback: ${fallbackValue}`);
        values.push("fallback");
    }
    
    options.push("❌ Cancel");
    values.push("cancel");

    Logger.debug("Presenting recovery options to user", {
        optionsCount: options.length,
        hasFallbackOption: fallbackValue !== null
    });

    const choice = await tp.system.suggester(options, values, false, "What would you like to do?");

    Logger.info("User selected recovery option", { 
        choice: choice || "none_selected",
        wasNull: choice === null
    });

    if (choice === "cancel") {
	    Logger.info("User chose to cancel operation");
        throw new Error("Operation cancelled by user");
    }
    
    if (choice === "fallback") {
        Logger.info(`User accepted fallback value: ${fallbackValue}`);
        showWarning(`Using fallback: ${fallbackValue}`);
        return fallbackValue;
    }
    
    // One final attempt
    Logger.info("User chose final retry attempt");
    const finalInput = await tp.system.prompt(`Final attempt - ${promptMessage}:`);
    if (finalInput === null) {
	    Logger.warn("User cancelled final attempt");
        if (fallbackValue !== null) {
            Logger.info(`Defaulting to fallback after cancellation: ${fallbackValue}`);
            showWarning(`Using fallback: ${fallbackValue}`);
            return fallbackValue;
        }
        Logger.info("No fallback available, operation cancelled");
        throw new Error("Operation cancelled by user");
    }

	Logger.info("Final attempt input received", { 
        input: finalInput,
        inputLength: finalInput.length
    });

	Logger.debug("Validating final attempt input");
    const finalValidation = await validationFn(finalInput);
    
    if (finalValidation.isValid) {
        Logger.info("Final attempt validation successful", { input: finalInput });
        return finalInput;
    }

	Logger.error("Final attempt validation failed", {
        input: finalInput,
        error: finalValidation.error,
        suggestion: finalValidation.suggestion
    });
    
    showError(`Final attempt failed validation: ${finalValidation.error}`);

	// All validation attempts exhausted
    Logger.info("All validation attempts exhausted, using fallback or failing");
    
    if (fallbackValue !== null) {
        Logger.info(`Using fallback after all attempts failed: ${fallbackValue}`);
        showWarning(`All attempts failed. Using fallback: ${fallbackValue}`);
        return fallbackValue;
    }
    
    Logger.error("No fallback available, operation failed completely");
    throw new Error("All validation attempts failed and no fallback available");
}

/**
 * Note title validation with comprehensive error handling and cross-platform compatibility
 * @param {string} title - Title to validate for use as filename
 * @param {string} destinationPath - Full file path where note would be created (for duplicate checking)
 * @param {boolean} isNote - Whether title is for a note file (true) or content type folder (false)
 * @returns {Promise<Object>} - Validation result object with isValid, error, suggestion, and canProceedAnyway properties
 */
async function validateNoteTitle(title, destinationPath, isNote) {
    const itemType = isNote ? "note" : "content type";
    const trimmedTitle = title.trim();

	Logger.debug(`Starting title validation`, {
        title,
        destinationPath,
        itemType,
        titleLength: title?.length || 0
    });

	try {
	    // Check for empty, undefined, or default titles
	    if (!title || title.includes('Untitled') || trimmedTitle === "") {
            Logger.warn(`Title validation failed: empty or default title`, { originalTitle: title });
			return createValidationResult(false, 
	            "Title cannot be empty or be the default 'Untitled' value",
	            `Provide a distinct ${itemType} title`
	        );
	    }
	
		// Check for leading dots
		if (title.startsWith('.')) {
            Logger.warn(`Title validation failed: starts with dot`, { title });
			return createValidationResult(false,
	            "Title cannot start with a dot (.) character",
	            `Try: "${trimmedTitle.substring(1) || 'Hidden File'}" or add a prefix`
	        );
	    }
	
		// Check for illegal filename characters
		illegalMatches = trimmedTitle.match(ILLEGAL_CHARS); 
	    if (illegalMatches) {
	        const uniqueChars = [...new Set(illegalMatches)];
            Logger.warn(`Title validation failed: illegal characters found`, {
                title,
                illegalChars: uniqueChars
            });
			return createValidationResult(false,
	            `Title contains characters that aren't allowed in filenames: ${uniqueChars.join(', ')}`,
	            "Replace these characters with spaces, hyphens, or underscores"
	        );
	    }
	
		// Check for Windows reserved names (case-insensitive)
	    if (RESERVED_NAMES.test(trimmedTitle)) {
            Logger.warn(`Title validation failed: Windows reserved name`, { title });
			return createValidationResult(false,
	            `"${trimmedTitle}" is reserved by Windows and cannot be used as a filename`,
	            `Try adding a prefix or suffix, like "${trimmedTitle} Notes" or "My ${trimmedTitle}"`
	        );
	    }
		
		// Check for excessively long titles (to encourage conciseness, the character limit is currently 100, which is well below the typical 255-byte filename limit of most OSs)
		if (title.length > VALIDATION_LIMITS.MAX_TITLE_LENGTH) {
            Logger.warn(`Title validation failed: too long`, { 
                title, 
                length: title.length, 
                maxLength: VALIDATION_LIMITS.MAX_TITLE_LENGTH 
            });
			return createValidationResult(false,
	            `Title is too long (${title.length} characters, maximum is ${VALIDATION_LIMITS.MAX_TITLE_LENGTH})`,
	            "Try shortening it or using abbreviations"
	        );
	    }
	    
		// Check for trailing dots and whitespace characters (problematic on Windows)
		if (title !== title.replace(/[. ]+$/, '')) {
            Logger.warn(`Title validation failed: trailing dots or spaces`, { title });
	        return createValidationResult(false,
	            "Title cannot end with dots or spaces (Windows compatibility issue)",
	            "Remove the trailing characters or replace with underscores"
	        );
	    }
	
		// Check for emojis mixed with text (might be intentional but worth flagging; can cause file explorer issues)
		if (EMOJI_REGEX.test(title)) {
            Logger.warn(`Title validation failed: contains emojis`, { title });
			return createValidationResult(false,
	            `Title contains both emojis and text: "${trimmedTitle}"`,
	            "Use text-only titles for consistency and cross-platform compatibility"
	        );
	    }
	
		// Check for invisible characters
		if (INVISIBLE_CHARS.test(title)) {
            Logger.warn(`Title validation failed: invisible characters detected`, { title });
			return createValidationResult(false,
	            "Title contains invisible characters that may cause file system issues",
	            "Please retype the title to remove any hidden characters"
	        );
	    }
	
	    // Check for duplicates
        Logger.debug(`Checking for existing file at path: ${destinationPath}`);
		const noteExists = await tp.file.exists(destinationPath);
	    if (noteExists) {
            Logger.warn(`Title validation failed: file already exists`, { 
                title, 
                destinationPath 
            });
			return createValidationResult(false,
	            `A ${itemType} with this title already exists in the destination directory`,
	            "Try adding a number, date, or descriptive suffix"
	        );
	    }

        Logger.info(`Title validation successful`, { title, itemType });
        return createValidationResult(true);

    } catch (error) {
        Logger.error("Title validation encountered unexpected error", error, {
            title,
            destinationPath,
            itemType
        });
        // Continue with validation as a warning rather than failing completely
        return createValidationResult(true);
    }
}

/**
 * Primary Category and Content Type search tag emoji validation with comprehensive error handling
 * @param {string} emoji - Emoji string to validate for use in Obsidian tags
 * @returns {Object} - Validation result object with isValid, error, suggestion, and canProceedAnyway properties
 */
function validateEmoji(emoji) {
	Logger.debug("Starting emoji validation", {
        emoji,
        emojiLength: emoji?.length || 0,
        codePoint: emoji ? emoji.codePointAt(0) : null
    });
    
	try {
	    // Check for empty or invalid input
	    if (!emoji || emoji.length === 0) {
		    Logger.warn("Emoji validation failed: empty input");
	        return createValidationResult(false, "Emoji cannot be empty", "Enter a single emoji character");
	    }
	    
	    // Check if input matches emoji regular expression
	    if (!EMOJI_REGEX.test(emoji)) {
		    Logger.warn("Emoji validation failed: not a valid emoji", { 
                emoji,
                regexTest: false
            });
	        return createValidationResult(false, "Not a valid emoji", "Try copying an emoji from your system's emoji picker");
	    }
	
	    // Check for Zero Width Joiner sequences (problematic for Obsidian tags)
	    const codepoints = Array.from(emoji).map(char => char.codePointAt(0));
	    const hasZWJ = codepoints.includes(0x200D);

		Logger.debug("Emoji codepoint analysis", {
            codepoints,
            hasZWJ,
            codepointCount: codepoints.length
        });

		if (hasZWJ) {
            Logger.warn("Emoji validation warning: contains ZWJ sequences", {
                emoji,
                codepoints
            });
	        return createValidationResult(false,
	            "Emoji contains Zero Width Joiner sequences that may cause tag display issues in Obsidian",
	            "Try a simpler emoji without profession/family modifiers (e.g., 👨‍💻 → 💻, 🕵️‍♂️ → 🕵️)",
	            true // Flag to indicate user can override this warning
	        );
	    }
	
		// Check for invisible characters
		if (INVISIBLE_CHARS.test(emoji)) {
            Logger.warn("Emoji validation failed: invisible characters detected", { emoji });
	        return createValidationResult(false,
	            "Input contains invisible characters that may cause tag issues in Obsidian",
	            "Enter a single emoji character"
	        );
	    }
	
		// Check if input contains whitespace characters
	    if (emoji !== emoji.trim()) { 
            Logger.warn("Emoji validation failed: contains whitespace", {
                emoji,
                trimmed: emoji.trim(),
                hasLeadingSpace: emoji !== emoji.trimStart(),
                hasTrailingSpace: emoji !== emoji.trimEnd()
            });
	        return createValidationResult(false, "Input cannot contain whitespace characters", "Enter a single emoji character");
	    }
	
		// Check if input contains a single visual character
		const segments = Array.from(SEGMENTER.segment(emoji));

        Logger.debug("Emoji segmentation analysis", {
            segmentCount: segments.length,
            segments: segments.map(s => s.segment)
        });

		if (segments.length > VALIDATION_LIMITS.MAX_TAG_LENGTH) {
            Logger.warn("Emoji validation failed: multiple characters", {
                emoji,
                segmentCount: segments.length
            });
	        return createValidationResult(false, "Input cannot contain multiple characters", "Enter a single emoji character");
	    }
	
		// Check if emoji is reserved for vault administration
		if (RESERVED_EMOJIS.has(emoji)) {
            Logger.warn("Emoji validation failed: reserved emoji", {
                emoji,
                reservedEmojis: Array.from(RESERVED_EMOJIS)
            });
	        return createValidationResult(false,
	            `"${emoji}" is a reserved emoji for vault administration`,
	            "Pick an emoji besides 🥇, 🥈, or ⚛️"
	        );
	    }

        Logger.info("Emoji validation successful", { emoji });
        return createValidationResult(true);
        
    } catch (error) {
        Logger.error("Emoji validation encountered unexpected error", error, {
            emoji,
            emojiLength: emoji?.length
        });
        // Return as valid to allow workflow to continue with warning
        Logger.warn("Proceeding with emoji validation despite error");
        return createValidationResult(true);
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                           CORE WORKFLOW FUNCTIONS                           //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Prompts user to select note type with validation loop to ensure selection
 * @returns {Promise<string>} - Selected note type constant from NOTE_TYPES (PRIMARY, SECONDARY, or CONTENT)
 * @throws {Error} - If user repeatedly cancels selection (handled internally with retry)
 */
async function selectNoteType() {
    Logger.info("Starting note type selection");
    
    const options = [NOTE_TYPES.PRIMARY, NOTE_TYPES.SECONDARY, NOTE_TYPES.CONTENT];
    const displayOptions = [
        "🥇 Primary Category (High-level topics)",
        "🥈 Secondary Category (Mid-level topics)", 
        "⚛️ Content/Atomic Note (Specific content)"
    ];

    Logger.debug("Presenting note type options", {
        optionCount: options.length,
        options: options
    });
    
    let selectedType = null;
    let attempts = 0;
    
    while (!selectedType) {
        attempts++;
        Logger.debug(`Note type selection attempt ${attempts}`);

        selectedType = await tp.system.suggester(
            displayOptions, 
            options, 
            false,
            "Select note type (required):"
        );
        
        if (!selectedType) {
            Logger.warn(`Note type not selected on attempt ${attempts}`);
            showError("Note type selection is required to continue");
        } else {
            Logger.info("Note type selected", {
                selectedType,
                attempts,
                selectionIndex: options.indexOf(selectedType)
            });
        }
    }
    
    return selectedType;
}

/**
 * Gets and validates note title using the generic retry function with type-specific validation
 * @param {string} noteType - Type of note being created (NOTE_TYPES.PRIMARY, NOTE_TYPES.SECONDARY, NOTE_TYPES.CONTENT, or NOTE_TYPES.CONTENT_TYPE)
 * @returns {Promise<string>} - Validated title that passes filename and uniqueness checks
 * @throws {Error} - If validation fails after max attempts and no fallback available
 */
async function getValidatedNoteTitle(noteType) {
    Logger.info(`Starting note title validation for ${noteType}`);
    
	const validationFn = async (title) => {
        Logger.debug("Validating title input", { title, noteType });

        const destinationPath = createDestinationPath(noteType, title);
        Logger.debug("Created destination path", { destinationPath });
        return await validateNoteTitle(title, destinationPath, true);
    };
    
	try {
        Logger.debug("Starting retry validation loop for note title");
        const title = await retryWithValidation(
            `Title for New ${noteType}`, 
            validationFn, 
            VALIDATION_LIMITS.MAX_VALIDATION_ATTEMPTS,
            tp.file.title
        );

        Logger.info(`Title validation completed successfully`, {
            noteType,
            title,
            titleLength: title.length,
            fallbackUsed: title === tp.file.title
        });
        
        showSuccess(`Title "${title}" is valid for ${noteType}`);
        return title;
    } catch (error) {
        Logger.error("Note title validation failed", error, {
            noteType,
            currentFileTitle: tp.file.title
        });
        showError("Note title validation failed");
        throw error;
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                            CATEGORY MANAGEMENT                              //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Retrieves all category files from a specific directory and returns sorted names
 * @param {string} folderPath - Relative path to category folder from vault root
 * @returns {Promise<string[]>} - Array of category names (basenames without .md extension), sorted alphabetically
 */
async function loadCategoriesFromFolder(folderPath) {
    Logger.debug("Loading categories from folder", { folderPath });
    
    try {
        const folder = app.vault.getAbstractFileByPath(folderPath);
        
        if (!folder) {
            Logger.warn("Folder not found", { folderPath });
            return [];
        }
        
        if (!folder.children) {
            Logger.warn("Folder has no children", { 
                folderPath,
                folderType: folder.constructor.name
            });
            return [];
        }

        Logger.debug("Scanning folder contents", {
            folderPath,
            totalChildren: folder.children.length
        });

        const mdFiles = folder.children.filter(file => file.extension === "md");
        const categories = mdFiles.map(file => file.basename).sort();

        Logger.info("Categories loaded successfully", {
            folderPath,
            totalFiles: folder.children.length,
            mdFiles: mdFiles.length,
            categories: categories.length,
            categoryNames: categories
        });

        return categories;

    } catch (error) {
        Logger.error("Failed to load categories from folder", error, {
            folderPath,
            vaultName: app.vault.getName()
        });
        return [];
    }
}

/**
 * Gets all primary categories from the primary categories folder
 * @returns {Promise<string[]>} - Array of primary category names, sorted alphabetically
 */
async function getPrimaryCategories() {
    return await loadCategoriesFromFolder(PATHS.PRIMARY_CATEGORIES);
}

/**
 * Gets all secondary categories from the secondary categories folder
 * @returns {Promise<string[]>} - Array of secondary category names, sorted alphabetically
 */
async function getSecondaryCategories() {
    return await loadCategoriesFromFolder(PATHS.SECONDARY_CATEGORIES);
}

/**
 * Interactive category selection with multi-select capability and formatted output
 * @param {string} categoryType - Type of categories to select ("primary" or "secondary")
 * @returns {Promise<string[]>} - Array of selected categories formatted as wiki links with quotes (e.g., ["[[Category 1]]", "[[Category 2]]"])
 */
async function selectCategories(categoryType) {
    Logger.info(`Starting ${categoryType} category selection`);

    const isSecondary = categoryType === "secondary";
    const availableCategories = isSecondary ? await getSecondaryCategories() : await getPrimaryCategories();
    const categoryLabel = isSecondary ? "SECONDARY" : "PRIMARY";
    
    try {
        Logger.debug("Available categories loaded", {
            categoryType,
            availableCount: availableCategories.length,
            categories: availableCategories
        });

        if (availableCategories.length === 0) {
            Logger.warn(`No ${categoryType} categories found`);
            return [];
        }
        
        const selectedCategories = [];
        let continueSelecting = true;
        let selectionRound = 0;
        
        while (continueSelecting && selectedCategories.length < availableCategories.length) {
            selectionRound++;
            Logger.debug(`Selection round ${selectionRound}`, {
                selectedSoFar: selectedCategories.length,
                remainingOptions: availableCategories.length - selectedCategories.length
            });

            const remainingCategories = availableCategories.filter(cat => !selectedCategories.includes(cat));

            if (remainingCategories.length === 0 && selectedCategories.length > 0) {
                Logger.debug("No remaining categories to select, but some already selected");
                break;
            }

            const stableOptions = buildStableMultiSelectOptions(availableCategories, selectedCategories);
            const displayOptions = stableOptions.displayOptions.slice(0, -1);
            const options = stableOptions.optionValues.slice(0, -1);

            options.push("DONE");
            displayOptions.push("✅ Done (Finish Selection)");
                        
            const promptText = selectedCategories.length === 0 
                ? `Select ${categoryLabel} category to link back to:` 
                : `Selected: ${selectedCategories.join(", ")}. Select another or choose Done:`;
            
            Logger.debug("Presenting category selection", {
                promptText,
                optionsAvailable: options.length,
                currentlySelected: selectedCategories
            });
            
            const selection = await tp.system.suggester(displayOptions, options, false, promptText);
            
            if (!selection) {
                Logger.info(`Category selection cancelled`, {
                    categoryType,
                    finalSelectionCount: selectedCategories.length,
                    selectionRounds: selectionRound
                });
                continueSelecting = false;
            } else if (selection === "DONE") {
                Logger.info(`Category selection completed`, {
                    categoryType,
                    finalSelectionCount: selectedCategories.length,
                    selectionRounds: selectionRound,
                    userChoseDone: selection === "DONE"
                });
                continueSelecting = false;
            } else if (selection.startsWith("REMOVE_")) {
                // Remove category
                const categoryToRemove = selection.substring(7); // Remove "REMOVE_" prefix
                const index = selectedCategories.indexOf(categoryToRemove);
                if (index > -1) {
                    selectedCategories.splice(index, 1);
                    Logger.debug(`Removed category: ${categoryToRemove}`, {
                        remainingCount: selectedCategories.length
                    });
                    showSuccess(`Removed category: ${categoryToRemove}`);
                }
            } else {
                // Add new category
                if (!selectedCategories.includes(selection)) {
                    selectedCategories.push(selection);
                    Logger.debug(`Category added to selection`, {
                        addedCategory: selection,
                        totalSelected: selectedCategories.length
                    });
                }
            }
        }

        // Format as proper wiki links
        const formattedCategories = selectedCategories.map(cat => `"[[${cat}]]"`);
        
        showSuccess(`Completed back-linking to ${categoryType} categories`);
        Logger.info(`Category selection process completed`, {
            categoryType,
            selectedCount: selectedCategories.length,
            formattedCategories
        });

        return formattedCategories;

    } catch (error) {
        Logger.error(`Failed to select ${categoryType} categories`, error, {
            categoryType,
            categoryLabel
        });
        return [];
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                              EMOJI MANAGEMENT                               //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Returns categorized emoji options organized by functional areas for red team activities
 * @returns {Object} - Object with category names as keys and arrays of {emoji, desc} objects as values
 */
function getCategorizedEmojis() {
    return {
        "🔥 Attack & Exploitation": [
            {emoji: "💥", desc: "Attack/Impact"}, 
            {emoji: "💣", desc: "Payload/Exploit"},
            {emoji: "⚔️", desc: "Tool/Weapon"},
            {emoji: "🎯", desc: "Target/Precision"},
            {emoji: "🔨", desc: "Brute Force"},
            {emoji: "💉", desc: "Code Injection"},
            {emoji: "🎣", desc: "Phishing"},
            {emoji: "🎭", desc: "Social Engineering"},
            {emoji: "🐚", desc: "Shell/Command Line"},
            {emoji: "🔴", desc: "Red Team Activity"},
            {emoji: "🟣", desc: "Purple Team Activity"},
            {emoji: "🔓", desc: "Unlocked/Broken Access Control"},
            {emoji: "🕳️", desc: "Vulnerability/Security Risk"}
        ],
        "🛡️ Defense & Security": [
            {emoji: "🛡️", desc: "Defense/Protection"},
            {emoji: "🔒", desc: "Access Control"},
            {emoji: "🔐", desc: "Encryption"},
            {emoji: "🔑", desc: "Authentication"},
            {emoji: "🧱", desc: "Firewall/Blocking"},
            {emoji: "👁️", desc: "Monitoring/Detection"},
            {emoji: "⚠️", desc: "Warning/Alert"},
            {emoji: "🔵", desc: "Blue Team Activity"},
            {emoji: "🚨", desc: "Incident Response"},
            {emoji: "👣", desc: "IOC/Artifact"},
            {emoji: "🗝️", desc: "Old Key/Cryptography"}
        ],
        "📊 Analysis & Intelligence": [
            {emoji: "📊", desc: "Data Analysis"},
            {emoji: "🔍", desc: "Investigation/OSINT"},
            {emoji: "📈", desc: "Metrics/Reporting"},
            {emoji: "📋", desc: "Audit/Checklist"},
            {emoji: "📝", desc: "Documentation/Notes"},
            {emoji: "😈", desc: "Threat Actor/APT"},
            {emoji: "✍️", desc: "Reporting/Writing"},
            {emoji: "🎒", desc: "Education/Training"},
            {emoji: "👤", desc: "Person/Silhouette"},
            {emoji: "💡", desc: "Idea/Thought"},
            {emoji: "🗺️", desc: "Mind Map/Graph"},
            {emoji: "✅", desc: "Checklist/Playbook"},
            {emoji: "🎓", desc: "Student/Training"},
            {emoji: "📕", desc: "Records"}
        ],
        "🌐 Infrastructure": [
            {emoji: "🌐", desc: "Network/Web"},
            {emoji: "🧠", desc: "Artificial Intelligence"},
            {emoji: "📡", desc: "Communication/C2"},
            {emoji: "💻", desc: "Client/Endpoint"},
            {emoji: "🖥️", desc: "Server/Infrastructure"},
            {emoji: "📱", desc: "Mobile/Device"},
            {emoji: "🗄️", desc: "Database/Storage"},
            {emoji: "📶", desc: "Wireless/RF"},
            {emoji: "☁️", desc: "Cloud/Third-Party"},
            {emoji: "🏦", desc: "Vault/Secured Data"},
            {emoji: "🏗️", desc: "Infrastructure/Construction"},
            {emoji: "🧪", desc: "Lab Setup"}
        ],
        "⚙️ Technical": [
            {emoji: "🔧", desc: "Configuration/Tool"},
            {emoji: "🛠️", desc: "Toolkit/Development"},
            {emoji: "🤖", desc: "Automation/Script"},
            {emoji: "💾", desc: "Data/Persistence"},
            {emoji: "📦", desc: "Package/Bundle"},
            {emoji: "⚡", desc: "Performance/Speed"},
            {emoji: "💲", desc: "Command/Shell"},
            {emoji: "⚙️", desc: "Gear/Internals"}
        ]
    };
}

/**
 * Emoji selector with categorized options and utility functions
 * @param {string} itemName - Name of item being tagged (used for context in prompt display)
 * @returns {Promise<string>} - Selected emoji character or default emoji if selection fails
 */
async function selectEmoji(itemName = "item") {
    Logger.info("Starting emoji selection", { itemName });
    try {
	    const categories = getCategorizedEmojis();
	    const options = [];
	    const displayOptions = [];

        Logger.debug("Building emoji selection options", {
            categoryCount: Object.keys(categories).length,
            totalEmojis: Object.values(categories).reduce((sum, cat) => sum + cat.length, 0)
        });

	    // Build categorized options
	    for (const [categoryName, emojis] of Object.entries(categories)) {
	        // Add category separator
	        displayOptions.push(`--- ${categoryName} ---`);
	        options.push(`${EMOJI_SELECTION.TYPES.SEPARATOR_PREFIX}${categoryName}`);
	        
	        // Add emoji options
	        emojis.forEach(item => {
	            displayOptions.push(`${item.emoji} ${item.desc}`);
	            options.push(item.emoji);
	        });
	    }
	    
	    // Add utility options
	    displayOptions.push(EMOJI_SELECTION.DISPLAY_TEXT.UTILITIES);
	    options.push(`${EMOJI_SELECTION.TYPES.UTILITIES}Utilities`);
	    displayOptions.push(EMOJI_SELECTION.DISPLAY_TEXT.MANUAL);
	    options.push(EMOJI_SELECTION.TYPES.MANUAL);
	    displayOptions.push(EMOJI_SELECTION.DISPLAY_TEXT.RANDOM);
	    options.push(EMOJI_SELECTION.TYPES.RANDOM);

        Logger.debug("Presenting emoji selection interface", {
            totalOptions: options.length,
            displayOptions: displayOptions.length
        });

	    const selection = await tp.system.suggester(
	        displayOptions,
	        options,
	        false,
	        `Select emoji for "${itemName}":`
	    );

        Logger.debug("Emoji selection made", {
            selection: selection || "none",
            itemName
        });
	
		// Handle user pressing escape key or exiting dropdown menu
	    if (!selection) {
            Logger.warn(`Emoji selection cancelled, using default`, {
                itemName,
                defaultEmoji: EMOJI_SELECTION.DEFAULT_EMOJI
            });
		    showWarning(`Emoji selection cancelled by user, using default emoji "${EMOJI_SELECTION.DEFAULT_EMOJI}"`);
	        return EMOJI_SELECTION.DEFAULT_EMOJI;
		}
	
		// Handle user selecting a category separator
		if (selection.startsWith(EMOJI_SELECTION.TYPES.UTILITIES)) {
            Logger.warn(`Category header selected, using default`, {
                selection,
                defaultEmoji: EMOJI_SELECTION.DEFAULT_EMOJI
            });
		    showWarning(`User selected a category header, using default emoji "${EMOJI_SELECTION.DEFAULT_EMOJI}"`);
		    return EMOJI_SELECTION.DEFAULT_EMOJI;
		}
	
		// Handle manual entry option
	    if (selection === EMOJI_SELECTION.TYPES.MANUAL) {
            Logger.info("User chose manual emoji entry");
            const manualEmoji = await getValidatedEmoji();
            return manualEmoji;
	    }
	
		// Handle randomized emoji choice option
	    if (selection === EMOJI_SELECTION.TYPES.RANDOM) {
	        const allEmojis = Object.values(categories).flat().map(e => e.emoji);
            const randomEmoji = allEmojis[Math.floor(Math.random() * allEmojis.length)] || EMOJI_SELECTION.DEFAULT_EMOJI;
            Logger.info("Random emoji selected", {
                randomEmoji,
                totalEmojiPool: allEmojis.length
            });
            return randomEmoji;
	    }

        Logger.info("Emoji selection completed", {
            selectedEmoji: selection,
            itemName,
            selectionMethod: "categorized"
        });

	    return selection;
	    
    } catch (error) {
        Logger.error("Emoji selection failed", error, {
            itemName,
            fallbackEmoji: EMOJI_SELECTION.DEFAULT_EMOJI
        });
        return EMOJI_SELECTION.DEFAULT_EMOJI;
    }
}

/**
 * Gets and validates emoji using the generic retry function with emoji-specific validation
 * @returns {Promise<string>} - Validated emoji character or fallback default emoji
 */
async function getValidatedEmoji() {
    Logger.info("Starting validated emoji input process");
    
    try {
        Logger.debug("Beginning emoji validation retry loop");
        const emoji = await retryWithValidation(
            "Search Tag Emoji for New Content Type",
            validateEmoji,
            VALIDATION_LIMITS.MAX_VALIDATION_ATTEMPTS,
            EMOJI_SELECTION.DEFAULT_EMOJI
        );

        const usedFallback = emoji === EMOJI_SELECTION.DEFAULT_EMOJI;

        Logger.info(`Emoji validation process completed`, {
            finalEmoji: emoji,
            usedFallback,
            validationSuccessful: !usedFallback
        });
        
        if (!usedFallback) {
            showSuccess(`Emoji "${emoji}" is a valid search tag`);
        }
        
        return emoji;
        
    } catch (error) {
        showError("Emoji validation failed");
        showWarning(`Using fallback emoji: ${EMOJI_SELECTION.DEFAULT_EMOJI}`);
        return EMOJI_SELECTION.DEFAULT_EMOJI;
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                           CONTENT TYPE MANAGEMENT                           //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Scans available content type templates and extracts their metadata including emojis
 * @returns {Promise<Object[]>} - Array of content type configuration objects with name, emoji, displayName, and searchTag properties
 */
async function getAvailableContentTypes() {
    Logger.info("Starting content types scan");

	try {
	    const templatesFolder = app.vault.getAbstractFileByPath(PATHS.CONTENT_TEMPLATES);
	    
	    if (!templatesFolder) {
            Logger.warn("Content templates folder not found", {
                expectedPath: PATHS.CONTENT_TEMPLATES
            });
	        return [];
	    }

        if (!templatesFolder.children) {
            Logger.warn("Content templates folder has no children", {
                folderPath: PATHS.CONTENT_TEMPLATES,
                folderType: templatesFolder.constructor.name
            });
            return [];
        }

        Logger.debug("Scanning content type templates", {
            templatesPath: PATHS.CONTENT_TEMPLATES,
            totalItems: templatesFolder.children.length
        });

        // Filter to only folders upfront
        const contentTypeFolders = templatesFolder.children.filter(item => 
            item.children && item.children.length > 0
        );
        
        Logger.debug("Found content type folders", {
            totalItems: templatesFolder.children.length,
            validFolders: contentTypeFolders.length,
            folderNames: contentTypeFolders.map(f => f.name)
        });

        // Batch metadata file path creation
        const metadataOperations = contentTypeFolders.map(folder => ({
            folder,
            metadataPath: `${PATHS.CONTENT_TEMPLATES}/${folder.name}/Metadata.md`
        }));

        // Batch check for metadata file existence
        const existenceChecks = await Promise.allSettled(
            metadataOperations.map(async ({ metadataPath }) => {
                const file = app.vault.getAbstractFileByPath(metadataPath);
                return { metadataPath, exists: !!file, file };
            })
        );

        Logger.debug("Metadata file existence check completed", {
            totalChecks: existenceChecks.length,
            successful: existenceChecks.filter(r => r.status === 'fulfilled').length
        });

        // Batch metadata content reading for existing files
        const metadataReads = [];
        const folderMetadataMap = new Map();

        existenceChecks.forEach((result, index) => {
            if (result.status === 'fulfilled' && result.value.exists) {
                const { folder } = metadataOperations[index];
                const { file } = result.value;
                folderMetadataMap.set(folder.name, file);
                metadataReads.push(
                    app.metadataCache.getFileCache(file)?.frontmatter || null
                );
            } else {
                const { folder } = metadataOperations[index];
                folderMetadataMap.set(folder.name, null);
                metadataReads.push(null);
            }
        });

        Logger.debug("Metadata content extraction completed", {
            metadataFilesFound: metadataReads.filter(m => m !== null).length,
            totalFolders: contentTypeFolders.length
        });

        // Process all content types in parallel-safe manner
        const contentTypes = [];
        let emojiExtractionFailures = 0;

        for (let i = 0; i < contentTypeFolders.length; i++) {
            const folder = contentTypeFolders[i];
            const frontmatter = metadataReads[i];
            const typeName = folder.name;
            let emoji = EMOJI_SELECTION.DEFAULT_EMOJI;

            try {
                if (frontmatter?.tags?.length > 0) {
                    const contentTag = frontmatter.tags.find(tag => 
                        typeof tag === 'string' &&
                        !tag.includes('Primary_Category') && 
                        !tag.includes('Secondary_Category')
                    );
                    
                    if (contentTag) {
                        const emojiMatch = contentTag.match(EMOJI_REGEX);
                        if (emojiMatch) {
                            emoji = emojiMatch[0];
                        }
                    }
                }
            } catch (error) {
                emojiExtractionFailures++;
                Logger.warn(`Emoji extraction failed for ${typeName}`, {
                    error: error.message,
                    typeName,
                    usingDefault: EMOJI_SELECTION.DEFAULT_EMOJI
                });
            }

            contentTypes.push({
                name: typeName,
                emoji: emoji,
                displayName: `${emoji} ${typeName}`,
                searchTag: `${emoji}${typeName.replace(/\s+/g, '_')}`
            });
        }
	    
        const sortedContentTypes = contentTypes.sort((a, b) => a.name.localeCompare(b.name));
        
        Logger.info("Content types scan completed", {
            totalFoldersScanned: contentTypeFolders.length,
            validContentTypes: contentTypes.length,
            emojiExtractionFailures,
            contentTypeNames: sortedContentTypes.map(ct => ct.name)
        });
        
        return sortedContentTypes;
        
    } catch (error) {
        Logger.error("Content types scan failed", error, {
            templatesPath: PATHS.CONTENT_TEMPLATES
        });
        return [];
    }
}

/**
 * Interactive content type selection or creation
 * @returns {Promise<Object>} - Selected/created content type configuration
 */
async function selectContentType() {
    Logger.info("Starting content type selection");

	try {
	    const availableTypes = await getAvailableContentTypes();

        Logger.debug("Available content types loaded for selection", {
            typeCount: availableTypes.length,
            typeNames: availableTypes.map(t => t.name)
        });

	    const options = [...availableTypes, "NEW_CONTENT_TYPE"];
	    const displayOptions = [
	        ...availableTypes.map(type => type.displayName),
	        "➕ Create New Content Type"
	    ];

        Logger.debug("Presenting content type selection interface", {
            totalOptions: options.length,
            existingTypes: availableTypes.length,
            hasCreateNewOption: true
        });

	    let selectedType = null;
	    while (!selectedType) {
	        selectedType = await tp.system.suggester(
	            displayOptions,
	            options,
	            false,
	            "Select Content Type or create new one:"
	        );
	        
	        if (!selectedType) {
                Logger.warn("No content type selected, defaulting to Basic");
                const basicType = availableTypes.find(type => type.name === "Basic");
                
                if (basicType) {
                    Logger.info("Found Basic content type for fallback", basicType);
                    showWarning("Using Basic content type");
                    return basicType;
                } else {
                    Logger.error("Basic content type not found in available types", {
                        availableTypes: availableTypes.map(t => t.name)
                    });
                    throw new Error("No Basic content type available and no selection made");
                }
	        }
	    }
	    
        const isNewType = selectedType === "NEW_CONTENT_TYPE";
        
        Logger.info("Content type selection made", {
            selectionType: isNewType ? "create_new" : "existing",
            selectedTypeName: isNewType ? null : selectedType.name
        });
        
        if (isNewType) {
            Logger.info("New content type selected");
            showSuccess("New content type selected");
            const newType = await createNewContentType();
            return newType;
        } else {
            Logger.info("Existing content type selected", selectedType);
            showSuccess("Existing content type selected");
            return selectedType;
        }

    } catch (error) {
        Logger.error("Content type selection failed", error);
        throw error;
    }
}

/**
 * Creates a new content type template structure with metadata, body, and footer files
 * @returns {Promise<Object>} - New content type configuration object with name, emoji, displayName, and searchTag properties
 */
async function createNewContentType() {
    Logger.info("====== Phase 4: New Content Type Creation Started ======");

	try {
        Logger.debug("Phase 4.1: Getting validated content type name");
	    const typeName = await getValidatedNoteTitle("Content Type");

        Logger.info("Content type name validated", {
            typeName,
            nameLength: typeName.length
        });

        Logger.debug("Phase 4.2: Getting emoji search tag for content type");
	    const selectedEmoji = await selectEmoji(typeName);

        Logger.info("Content type emoji selected", {
            typeName,
            selectedEmoji,
            emojiCodePoint: selectedEmoji.codePointAt(0)
        });

        Logger.debug("Phase 4.3: Selecting query tier for new content type");
        const queryTier = await selectQueryTierForContentType(typeName);

        Logger.info("Content type query tier selected", {
            typeName,
            queryTier
        });

        Logger.debug("Phase 4.4: Creating content type template structure");
	    await createTemplateStructure(typeName, selectedEmoji, queryTier);

        const newContentType = {
            name: typeName,
            emoji: selectedEmoji,
            displayName: `${selectedEmoji} ${typeName}`,
            searchTag: `${selectedEmoji}${typeName.replace(/\s+/g, '_')}`,
            queryTier
        };
        
        Logger.info("====== Phase 4: New Content Type Creation Completed Successfully ======", newContentType);
        
        return newContentType;
        
    } catch (error) {
        Logger.error("New content type creation failed", error, {
            phase: "unknown" // Could be enhanced to track current phase
        });
        throw error;
    }
}

/**
 * Prompts the user to choose the starter Dataview query tier for a new custom content type.
 * @param {string} typeName - Name of the new content type for prompt context.
 * @returns {Promise<string>} - Selected query tier constant from QUERY_TIERS.
 */
async function selectQueryTierForContentType(typeName) {
    const options = [QUERY_TIERS.NONE, QUERY_TIERS.LIGHTWEIGHT, QUERY_TIERS.RICH];
    const displayOptions = [
        "📝 None - Create an empty Related Notes section",
        "🔎 Lightweight - Seed typed relationships and same classification headings",
        "🕸️ Rich - Seed same classification, typed relationships, and reverse relationships headings"
    ];

    const selectedTier = await tp.system.suggester(
        displayOptions,
        options,
        false,
        `Select Dataview query tier for "${typeName}":`
    );

    if (!selectedTier) {
        Logger.warn("No query tier selected for new content type; defaulting to None", {
            typeName,
            defaultTier: QUERY_TIERS.NONE
        });
        return QUERY_TIERS.NONE;
    }

    return selectedTier;
}

/**
 * Creates template folder structure and files for new content type based on Basic template
 * @param {string} typeName - Content type name for folder and file naming
 * @param {string} emoji - Selected emoji for search tags and identification
 * @param {string} queryTier - Selected Dataview query tier used to seed starter query scaffolding.
 * @returns {Promise<void>} - Creates the template folder and files for the new content type.
 * @throws {Error} - If template creation fails (folder creation, file copying, or customization)
 */
async function createTemplateStructure(typeName, emoji, queryTier) {
	// Obsidian handles cross-platform file separators automatically
    const basePath = `${PATHS.CONTENT_TEMPLATES}/${typeName}`;

    Logger.info(`Creating template structure for new content type`, {
        typeName,
        emoji,
        queryTier,
        basePath
    });

    try {
        Logger.debug(`Creating base folder: ${basePath}`);
        await app.vault.createFolder(basePath);
	    Logger.debug("Base folder created successfully");    
	    
        const templateFiles = [
            { source: "Metadata.md", target: "Metadata.md" },
            { source: "Footer.md", target: "Footer.md" },
            { source: "Body.md", target: "Body.md" }
        ];

		let filesCreated = 0;
        for (const fileInfo of templateFiles) {
            const sourceFilePath = `${PATHS.BASIC_TEMPLATE}/${fileInfo.source}`;
            const targetFilePath = `${basePath}/${fileInfo.target}`;

            Logger.debug(`Processing template file: ${fileInfo.source}`);

            const sourceFile = app.vault.getAbstractFileByPath(sourceFilePath);
            if (!sourceFile) {
                Logger.warn(`Source template file not found: ${sourceFilePath}`);
                continue;
            }
            
            let content = await app.vault.read(sourceFile);
            Logger.debug(`Read source file (${content.length} chars)`);
            
            // Customize metadata file
            if (fileInfo.source === "Metadata.md") {
                Logger.debug("Customizing metadata template");
                content = content
                    .replace(
                        'tags:\n  - 📝Basic',
                        `tags:\n  - ${emoji}${typeName.replace(/\s+/g, '_')}`
                    )
                    .replace(
                        'type: Basic',
                        `type: ${typeName}`
                    );
                Logger.debug("Metadata customization completed");
            }
            
            // Create the file
            await app.vault.create(targetFilePath, content);
            filesCreated++;
            Logger.debug(`Created file: ${targetFilePath}`);
        }

        const dataviewTargetPath = `${basePath}/Dataview.md`;
        await app.vault.create(dataviewTargetPath, createStarterDataviewTemplate(queryTier));
        filesCreated += 2;
        Logger.debug(`Created file: ${dataviewTargetPath}`);
        Logger.debug(`Created file: ${schemaTargetPath}`);

        Logger.info(`Template structure created successfully`, {
            typeName,
            emoji,
            filesCreated,
            basePath
        });

        showSuccess(`Created new content type: ${emoji} ${typeName}`);
        
    } catch (error) {
        Logger.error("Template structure creation failed", error, {
            typeName,
            emoji,
            basePath
        });
        throw error;
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                              NOTE BUILDING                                  //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Note Configuration Builder - eliminates complex parameter objects
 */
class NoteConfigBuilder {
    /**
     * Initializes a builder with the base note type and title used throughout note creation.
     * @param {string} noteType - Selected note type constant.
     * @param {string} title - Validated title for the note being created.
     * @returns {void} - Stores the initial mutable configuration state on the builder instance.
     */
    constructor(noteType, title) {
        this.config = { 
            noteType, 
            title,
            primaryCategories: [],
            secondaryCategories: []
        };
    }
    
    /**
     * Stores selected primary category links on the in-progress configuration.
     * @param {string[]} categories - Primary category wiki links.
     * @returns {NoteConfigBuilder} - Builder instance for fluent chaining.
     */
    withPrimaryCategories(categories) {
        this.config.primaryCategories = categories || [];
        return this;
    }
    
    /**
     * Stores selected secondary category links on the in-progress configuration.
     * @param {string[]} categories - Secondary category wiki links.
     * @returns {NoteConfigBuilder} - Builder instance for fluent chaining.
     */
    withSecondaryCategories(categories) {
        this.config.secondaryCategories = categories || [];
        return this;
    }
    
    /**
     * Stores the selected emoji used for category tagging or custom content type identification.
     * @param {string} emoji - Selected emoji character.
     * @returns {NoteConfigBuilder} - Builder instance for fluent chaining.
     */
    withEmoji(emoji) {
        this.config.emoji = emoji;
        return this;
    }
    
    /**
     * Stores the short Overview description collected for category notes.
     * @param {string} categoryDescription - 1-2 sentence description for the category.
     * @returns {NoteConfigBuilder} - Builder instance for fluent chaining.
     */
    withCategoryDescription(categoryDescription) {
        this.config.categoryDescription = categoryDescription || "";
        return this;
    }
    
    /**
     * Stores the selected content type configuration for a content note.
     * @param {Object} contentTypeConfig - Selected content type configuration object.
     * @returns {NoteConfigBuilder} - Builder instance for fluent chaining.
     */
    withContentType(contentTypeConfig) {
        this.config.contentType = contentTypeConfig;
        return this;
    }

    /**
     * Stores normalized prompted property values collected during content-note creation.
     * @param {Object} promptedProperties - Prompted property map keyed by property name.
     * @returns {NoteConfigBuilder} - Builder instance for fluent chaining.
     */
    withPromptedProperties(promptedProperties) {
        this.config.promptedProperties = promptedProperties || {};
        return this;
    }

    /**
     * Stores a summary of prompt outcomes for later notices and fallback recovery content.
     * @param {Object|null} promptedPropertySummary - Summary object returned from prompt collection.
     * @returns {NoteConfigBuilder} - Builder instance for fluent chaining.
     */
    withPromptedPropertySummary(promptedPropertySummary) {
        this.config.promptedPropertySummary = promptedPropertySummary || null;
        return this;
    }
    
    /**
     * Finalizes the configuration object by attaching derived paths and template references.
     * @returns {Object} - Complete note configuration object ready for content assembly.
     */
    build() {
        return this._addPathsAndTemplates(this.config);
    }
    
    /**
     * Adds destination paths, template references, and workflow defaults based on note type.
     * @param {Object} config - Partially built note configuration.
     * @returns {Object} - Enriched configuration with destination and template metadata.
     */
    _addPathsAndTemplates(config) {
        const pathConfigurationStrategies = {
            [NOTE_TYPES.PRIMARY]: () => ({
                destination: `${PATHS.PRIMARY_CATEGORIES}/`,
                metadataTemplate: PATHS.PRIMARY_TEMPLATE_META,
                bodyTemplate: PATHS.PRIMARY_TEMPLATE_BODY
            }),
            [NOTE_TYPES.SECONDARY]: () => ({
                destination: `${PATHS.SECONDARY_CATEGORIES}/`,
                metadataTemplate: PATHS.SECONDARY_TEMPLATE_META,
                bodyTemplate: PATHS.SECONDARY_TEMPLATE_BODY
            }),
            [NOTE_TYPES.CONTENT]: () => {
                if (!config.contentType || !config.contentType.name) {
                    throw new Error("Content type is required for content notes but was not provided");
                }
                const templatePaths = createContentTemplatePaths(config.contentType.name);
                return {
                    destination: `${PATHS.CONTENT}/`,
                    noteStatus: NOTE_STATUS.DRAFT,
                    contentTypeSchema: buildResolvedContentTypeSchema(config.contentType.name),
                    ...templatePaths
                };
            }
        };
        
        const strategy = pathConfigurationStrategies[config.noteType];
        if (!strategy) {
            throw new Error(`Unsupported note type: ${config.noteType}`);
        }
        
        const pathConfig = strategy();
        return { ...config, ...pathConfig };
    }
}

/**
 * Loads all required templates for a note configuration from Obsidian template files
 * @param {Object} config - Note configuration object containing template paths
 * @returns {Promise<Object>} - Template content object with metadata, body, and optional dataview/footer properties
 * @throws {Error} - If any required template file cannot be loaded
 */
async function loadNoteTemplates(config) {
	Logger.info("Loading note templates", {
        metadataTemplate: config.metadataTemplate,
        bodyTemplate: config.bodyTemplate,
        dataviewTemplate: config.dataviewTemplate || 'none',
        footerTemplate: config.footerTemplate || 'none'
    });

    try {
        Logger.debug("Loading metadata template");
		const templates = {
            metadata: await tp.file.include(config.metadataTemplate)
        };
        Logger.debug(`Metadata template loaded (${templates.metadata.length} chars)`);

        Logger.debug("Loading body template");
        templates.body = await tp.file.include(config.bodyTemplate);
        Logger.debug(`Body template loaded (${templates.body.length} chars)`);

        templates.dataview = await loadOptionalTemplate(config.dataviewTemplate, "dataview");
        templates.footer = await loadOptionalTemplate(config.footerTemplate, "footer");

        Logger.info("All templates loaded successfully", {
            templateCount: Object.keys(templates).length,
            totalChars: Object.values(templates).reduce((sum, t) => sum + t.length, 0)
        });

        return templates;
    } catch (error) {
        Logger.error("Template loading failed", error, {
            metadataTemplate: config.metadataTemplate,
            bodyTemplate: config.bodyTemplate,
            dataviewTemplate: config.dataviewTemplate,
            footerTemplate: config.footerTemplate
        });
        throw new Error(`Failed to load templates: ${error.message}`);
    }
}

/**
 * Customizes metadata template based on note configuration using early returns for type-specific logic
 * @param {string} metadata - Raw metadata template content from template file
 * @param {Object} config - Note configuration object with noteType, emoji, title, primaryCategories, secondaryCategories
 * @returns {string} - Customized metadata with replaced placeholders and category links
 */
function customizeMetadata(metadata, config) {
    Logger.debug("Starting metadata customization", {
        metadataLength: metadata?.length || 0,
        noteType: config?.noteType,
        hasConfig: !!config
    });

    if (!metadata || !config) {
        Logger.warn("Missing metadata or config for customization", {
            hasMetadata: !!metadata,
            hasConfig: !!config
        });
        return metadata;
	};
    
    let customizedMetadata = metadata;
    let transformationsApplied = 0;
    
    if (config.noteType === NOTE_TYPES.PRIMARY && config.emoji) {
        Logger.debug("Applying PRIMARY category metadata customization", {
            emoji: config.emoji,
            title: config.title
        });

        const originalPattern = 'tags:\n  - 🥇Primary_Category\n  - ADD_NEW_PRIMARY_CATEGORY_EMOJI';
        const replacement = `tags:\n  - 🥇Primary_Category\n  - ${config.emoji}${config.title.replace(/\s+/g, '_')}`;
        
        customizedMetadata = customizedMetadata.replace(originalPattern, replacement);
        transformationsApplied++;
        
        Logger.debug("PRIMARY category metadata transformation applied", {
            originalPattern,
            replacement,
            transformationCount: transformationsApplied
        });
        
        return customizedMetadata;
    }
    
    if (config.noteType === NOTE_TYPES.SECONDARY) {
        Logger.debug("Applying SECONDARY category metadata customization", {
            primaryCategoriesCount: config.primaryCategories?.length || 0
        });
        
        customizedMetadata = customizeCategoriesMetadata(customizedMetadata, config.primaryCategories, 'primary');
        transformationsApplied++;
        
        Logger.debug("SECONDARY category metadata customization completed", {
            transformationsApplied
        });
        
        return customizedMetadata;
    }
    
    if (config.noteType === NOTE_TYPES.CONTENT) {
        Logger.debug("Applying CONTENT note metadata customization", {
            primaryCategoriesCount: config.primaryCategories?.length || 0,
            secondaryCategoriesCount: config.secondaryCategories?.length || 0,
            promptedPropertyCount: Object.keys(config.promptedProperties || {}).length
        });
        
        customizedMetadata = customizeCategoriesMetadata(customizedMetadata, config.primaryCategories, 'primary');
        transformationsApplied++;
        
        customizedMetadata = customizeCategoriesMetadata(customizedMetadata, config.secondaryCategories, 'secondary');
        transformationsApplied++;

        customizedMetadata = applyPromptedPropertyValues(
            customizedMetadata,
            config.promptedProperties,
            config.contentTypeSchema
        );
        transformationsApplied++;

        const lineEnding = detectLineEnding(customizedMetadata);
        customizedMetadata = ensureMetadataProperty(
            customizedMetadata,
            "note-status",
            config.noteStatus || NOTE_STATUS.DRAFT,
            lineEnding
        );
        transformationsApplied++;

        Logger.debug("CONTENT note metadata customization completed", {
            transformationsApplied
        });
        
        return customizedMetadata;
    }
    
    Logger.debug("No metadata customization needed", {
        noteType: config.noteType,
        supportedTypes: [NOTE_TYPES.PRIMARY, NOTE_TYPES.SECONDARY, NOTE_TYPES.CONTENT]
    });
    
    return customizedMetadata;
}

/**
 * Helper function to customize category metadata sections for primary and secondary category links
 * @param {string} metadata - Metadata content to modify
 * @param {string[]} categories - Array of category names formatted as wiki links
 * @param {string} categoryType - Type of categories being processed ("primary" or "secondary")
 * @returns {string} - Metadata with category placeholders replaced by actual category links
 */
function customizeCategoriesMetadata(metadata, categories, categoryType) {
    Logger.debug(`Starting ${categoryType} categories metadata customization`, {
        categoriesCount: categories?.length || 0,
        categoryType,
        metadataLength: metadata?.length || 0
    });
    
    if (!categories || categories.length === 0) {
        Logger.debug(`No ${categoryType} categories to process`);
        return metadata;
	}
    
    const replacementPatterns = {
        'primary': {
            pattern: /primary(?:-|\s)categories:\r?\n  - Add link\(s\) \[\[\]\] back to related PRIMARY categories/,
            replacement: `primary-categories:\n  - ${categories.join('\n  - ')}`
        },
        'secondary': {
            pattern: /secondary(?:-|\s)categories:\r?\n  - Add link\(s\) \[\[\]\] back to related SECONDARY categories/,
            replacement: `secondary-categories:\n  - ${categories.join('\n  - ')}`
        }
    };
    
    const config = replacementPatterns[categoryType];

    if (!config) {
        Logger.warn(`Unknown category type for metadata customization`, {
            categoryType,
            supportedTypes: Object.keys(replacementPatterns)
        });
        return metadata;
    }
    
    const originalLength = metadata.length;
    const customizedMetadata = metadata.replace(config.pattern, config.replacement);
    const replacementMade = customizedMetadata !== metadata;
    
    Logger.debug(`${categoryType} categories metadata customization completed`, {
        categoryType,
        categoriesProcessed: categories.length,
        replacementMade,
        originalLength,
        newLength: customizedMetadata.length,
        categories: categories
    });
    
    if (!replacementMade) {
        Logger.warn(`Pattern not found for ${categoryType} categories replacement`, {
            expectedPattern: config.pattern,
            metadataPreview: metadata.substring(0, 200) + "..."
        });
    }
    
    return customizedMetadata;
}

/**
 * Injects a prompted description into the Overview section of a primary or secondary category body template.
 * @param {string} body - Raw body template content for the category note.
 * @param {Object} config - Note configuration containing noteType and categoryDescription.
 * @returns {string} - Updated body template with the Overview placeholder replaced when applicable.
 */
function customizeCategoryBody(body, config) {
    if (![NOTE_TYPES.PRIMARY, NOTE_TYPES.SECONDARY].includes(config.noteType)) {
        return body;
    }

    const categoryDescription = (config.categoryDescription || "").trim();
    if (!categoryDescription) {
        return body;
    }

    const placeholderPattern =
        config.noteType === NOTE_TYPES.PRIMARY
            ? /<!-- Brief description of primary category -->/
            : /<!-- Brief description of secondary category -->/;

    if (placeholderPattern.test(body)) {
        return body.replace(placeholderPattern, categoryDescription);
    }

    return body.replace(/^## Overview\s*$/m, `## Overview\n\n${categoryDescription}`);
}

/**
 * Normalizes assembled note sections so divider insertion does not create accidental extra blank lines.
 * @param {string} sectionContent - Raw content section to prepare for final note assembly.
 * @returns {string} - Trimmed section content ready to be joined with the standard divider.
 */
function normalizeAssembledSection(sectionContent) {
    if (typeof sectionContent !== "string") {
        return "";
    }

    return sectionContent.trim();
}

/**
 * Builds the complete note content from templates and configuration with proper assembly
 * @param {Object} config - Complete note configuration object with all required properties
 * @returns {Promise<string>} - Fully assembled note content ready for file creation
 * @throws {Error} - If template loading or content assembly fails
 */
async function buildNoteContent(config) {
    Logger.info("====== Phase 3: Note Content Build Started ======", {
        noteType: config.noteType,
        title: config.title,
        hasContentType: !!config.contentType
    });

    try {
        Logger.debug("Phase 3.1: Loading note templates");
        const templates = await loadNoteTemplates(config);

        Logger.debug("Phase 3.2: Customizing metadata");
        const customizedMetadata = customizeMetadata(templates.metadata, config);
        const customizedBody = customizeCategoryBody(templates.body, config);

        Logger.debug("Metadata customization results", {
            originalLength: templates.metadata.length,
            customizedLength: customizedMetadata.length,
            changesMade: customizedMetadata !== templates.metadata
        });

        Logger.debug("Phase 3.3: Assembling final content");
		const finalContent = assembleNoteContent(customizedMetadata, { ...templates, body: customizedBody }, config);

        Logger.info("====== Phase 3: Note Content Build Completed Successfully ======", {
            finalContentLength: finalContent.length,
            noteType: config.noteType,
            title: config.title
        });
        
        return finalContent;
        
    } catch (error) {
        Logger.error("Note content build failed", error, {
            noteType: config.noteType,
            title: config.title,
            configKeys: Object.keys(config)
        });
        throw error;
    }
}

/**
 * Assembles the final note content from all components with proper dividers and formatting
 * @param {string} customizedMetadata - Processed metadata with all placeholders replaced
 * @param {Object} templates - Template content object with metadata, body, and optional dataview/footer
 * @param {Object} config - Note configuration object for type-specific assembly logic
 * @returns {string} - Complete note content with header, body, footer, and timestamp
 */
function assembleNoteContent(customizedMetadata, templates, config) {
    Logger.debug("Starting note content assembly", {
        noteType: config.noteType,
        title: config.title,
        templateComponents: Object.keys(templates),
        metadataLength: customizedMetadata.length
    });

    const pageTitle = `# [[${config.title}]]`;
    const header = customizedMetadata + pageTitle;
    const contentParts = [header];

    Logger.debug("Header component created", {
        headerLength: header.length,
        titleLength: pageTitle.length
    });

    let componentsAdded = 1; // Header is always added

    // Add content based on note type
    switch (config.noteType) {
        case NOTE_TYPES.CONTENT:
            Logger.debug("Assembling CONTENT note components");
            contentParts.push(templates.body);
            if (templates.dataview?.trim()) {
                contentParts.push(templates.dataview);
                componentsAdded++;
            }
            if (templates.footer?.trim()) {
                contentParts.push(templates.footer);
                componentsAdded++;
            }
            contentParts.push(TIMESTAMP);
            componentsAdded += 2;
            Logger.debug("CONTENT note components added", {
                bodyLength: templates.body?.length || 0,
                dataviewLength: templates.dataview?.length || 0,
                footerLength: templates.footer?.length || 0,
                timestampLength: TIMESTAMP.length
            });
            break;
        case NOTE_TYPES.PRIMARY:
        case NOTE_TYPES.SECONDARY:
            Logger.debug(`Assembling ${config.noteType} note components`);
            contentParts.push(templates.body, TIMESTAMP);
            componentsAdded += 2;
            Logger.debug(`${config.noteType} note components added`, {
                bodyLength: templates.body?.length || 0,
                timestampLength: TIMESTAMP.length
            });
            break;
        default:
            Logger.warn(`Unknown/unexpected note type in assembly: ${config.noteType}`);
            contentParts.push(templates.body, TIMESTAMP);
            componentsAdded += 2;
    }
    
    const finalContent = contentParts
        .map(normalizeAssembledSection)
        .filter(Boolean)
        .join(DIVIDER);
    
    Logger.info("Note content assembly completed", {
        noteType: config.noteType,
        componentsAdded,
        totalParts: contentParts.length,
        finalContentLength: finalContent.length,
        dividerLength: DIVIDER.length,
        averageComponentSize: Math.round(finalContent.length / componentsAdded)
    });
    
    return finalContent;
}

//////////////////////////////////////////////////////////////////////////////////
//                         WORKFLOW ORCHESTRATION                              //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Builds note configuration based on user selections for note type and related options
 * @returns {Promise<Object>} - Complete note configuration object ready for content generation
 * @throws {Error} - If user cancels configuration process or validation fails
 */
async function buildNoteConfiguration() {
	Logger.info("====== Phase 1: Note Configuration Build Started ======");
    let noteType;
    let title;

	try {
		Logger.debug("Phase 1.1: Selecting note type");
	    noteType = await selectNoteType();
	    Logger.info(`Selected note type: ${noteType}`);

		Logger.debug("Phase 1.2: Saving validated note title");
	    title = await getValidatedNoteTitle(noteType);
	    Logger.info(`Validated title: "${title}"`);

        Logger.debug("Phase 1.3: Creating type-specific configuration", { noteType, title });
	    const config = await createConfigurationForNoteType(noteType, title);

		Logger.info("====== Phase 1: Note Configuration Build Completed Successfully ======", {
            noteType: config.noteType,
            title: config.title,
            hasPrimaryCategories: config.primaryCategories?.length > 0,
            hasSecondaryCategories: config.secondaryCategories?.length > 0,
            hasContentType: !!config.contentType
        });

		return config;
    } catch (error) {
	    Logger.error("Note configuration build failed", error, { noteType, title });
        throw error;
    }
}

/**
 * Creates configuration for specific note type using builder pattern with type-specific workflows
 * @param {string} noteType - Note type constant (NOTE_TYPES.PRIMARY, NOTE_TYPES.SECONDARY, or NOTE_TYPES.CONTENT)
 * @param {string} title - Validated note title
 * @returns {Promise<Object>} - Note configuration object with all required properties for the specified type
 * @throws {Error} - If unsupported note type or configuration process fails
 */
async function createConfigurationForNoteType(noteType, title) {
    Logger.info("Starting note configuration creation", {
        noteType,
        title,
        titleLength: title.length
    });

	try {
	    const builder = new NoteConfigBuilder(noteType, title);
        Logger.debug("Note config builder initialized", {
            noteType,
            title
        });

	    const configStrategies = {
	        [NOTE_TYPES.PRIMARY]: async () => {
                Logger.debug("Executing PRIMARY category configuration strategy");
	            const emoji = await selectEmoji(title);
                const categoryDescription = await promptCategoryDescription(noteType, title);

                Logger.debug("PRIMARY configuration completed", { emoji, hasCategoryDescription: !!categoryDescription });
	            return builder
                    .withEmoji(emoji)
                    .withCategoryDescription(categoryDescription);
	        },
	        [NOTE_TYPES.SECONDARY]: async () => {
                Logger.debug("Executing SECONDARY category configuration strategy");
	            const primaryCategories = await selectCategories("primary");
                const categoryDescription = await promptCategoryDescription(noteType, title);

                Logger.debug("SECONDARY configuration completed", {
                    primaryCategoriesCount: primaryCategories.length,
                    hasCategoryDescription: !!categoryDescription
                });
	            return builder
                    .withPrimaryCategories(primaryCategories)
                    .withCategoryDescription(categoryDescription);
	        },
	        [NOTE_TYPES.CONTENT]: async () => {
                Logger.debug("Executing CONTENT note configuration strategy");
		        const primaryCategories = await selectCategories("primary");
		        const secondaryCategories = await selectCategories("secondary");
		        const contentType = await selectContentType();
                const promptedPropertyCollection = await collectPromptedContentProperties(contentType);

                Logger.debug("CONTENT configuration completed", {
                    primaryCategoriesCount: primaryCategories.length,
                    secondaryCategoriesCount: secondaryCategories.length,
                    contentTypeName: contentType.name,
                    promptedPropertyCount: Object.keys(promptedPropertyCollection.values).length
                });

                return builder
                    .withPrimaryCategories(primaryCategories)
                    .withSecondaryCategories(secondaryCategories)
                    .withContentType(contentType)
                    .withPromptedProperties(promptedPropertyCollection.values)
                    .withPromptedPropertySummary(promptedPropertyCollection.summary);
	        }
	    };
	    
	    const strategy = configStrategies[noteType];
	    if (!strategy) {
            Logger.error("Unsupported note type in configuration", {
                noteType,
                supportedTypes: Object.keys(configStrategies)
            });
            throw new Error(`Unsupported note type: ${noteType}`);
	    }

        Logger.debug(`Executing configuration strategy for ${noteType}`);
	    const configuredBuilder = await strategy();

        Logger.debug("Building final configuration object");
        const finalConfig = configuredBuilder.build();
        
        Logger.info("Note configuration creation completed", {
            noteType: finalConfig.noteType,
            title: finalConfig.title,
            destination: finalConfig.destination,
            hasEmoji: !!finalConfig.emoji,
            hasPrimaryCategories: finalConfig.primaryCategories?.length > 0,
            hasSecondaryCategories: finalConfig.secondaryCategories?.length > 0,
            hasContentType: !!finalConfig.contentType,
            configKeys: Object.keys(finalConfig)
        });
        
        return finalConfig;
		
    } catch (error) {
        Logger.error("Note configuration creation failed", error, {
            noteType,
            title
        });
        throw error;
    }
}

/**
 * Moves the current note to its destination directory based on note type and configuration
 * @param {Object} config - Note configuration object containing destination path and title
 * @returns {Promise<void>} - Moves the active file into its final destination folder.
 * @throws {Error} - If file move operation fails (permissions, path issues, etc.)
 */
async function moveNoteToDestination(config) {
	Logger.info("====== Phase 2: Note Relocation Started ======");
	const destinationPath = `${config.destination}${config.title}`;
    const destinationFolder = config.destination.endsWith("/")
        ? config.destination.slice(0, -1)
        : config.destination;

	Logger.debug(`Phase 2.1: Moving note to destination`, {
        from: tp.file.path(true),
        to: destinationPath
    });
	
	try {
        if (!app.vault.getAbstractFileByPath(destinationFolder)) {
            await app.vault.createFolder(destinationFolder);
        }
        await tp.file.move(destinationPath);
        Logger.debug(`File successfully moved to: ${destinationPath}`);
    	Logger.info("====== Phase 2: Note Relocation Completed Successfully ======");
    } catch (error) {
        Logger.error("File move operation failed", error, {
            sourceFile: tp.file.path(true),
            destinationPath: destinationPath,
            noteType: config.noteType,
            title: config.title
        });
        throw new Error(`Failed to move note: ${error.message}`);
    }
}

/**
 * Shows end-of-workflow notices so users understand what was filled, what was preserved, and where the note was saved
 * @param {Object} config - Final note configuration
 * @returns {void} - Displays completion notices tailored to the note type and prompt outcomes.
 */
function showCompletionNotices(config) {
    const destinationMessage = `${config.noteType} "${config.title}" created in ${config.destination}`;
    showSuccess(destinationMessage);

    if (config.noteType !== NOTE_TYPES.CONTENT) {
        return;
    }

    const summary = summarizePromptedPropertyOutcomes(config.promptedPropertySummary);
    if (summary.preservedCount > 0) {
        showWarning(
            `Left ${summary.preservedCount} prompted propert${summary.preservedCount === 1 ? "y" : "ies"} as template placeholders: ` +
            `${summary.preservedPlaceholderProperties.join(", ")}`
        );
    }

    showSuccess(
        `Prompted ${summary.promptCount} content propert${summary.promptCount === 1 ? "y" : "ies"}; ` +
        `filled ${summary.providedCount} and set note-status to ${config.noteStatus || NOTE_STATUS.DRAFT}.`
    );
}

/**
 * Creates fallback content when the main workflow fails to ensure user gets usable note
 * @param {Error|string} error - Failure encountered during note creation
 * @param {Object|null} config - Partial or completed note configuration when available
 * @returns {string} Safe fallback note content with error context and timestamps
 */
function createFallbackContent(error, config = null) {
    const fallbackTitle = config?.title || tp.file.title || "New Note";
    const errorMessage = getErrorMessage(error);
    const wasCancelled = isCancellationError(error);

    if (config?.noteType === NOTE_TYPES.CONTENT) {
        const contentTypeName = config.contentType?.name || "Unknown";
        const safeSummary = summarizePromptedPropertyOutcomes(config.promptedPropertySummary);
        const preservedSection = safeSummary.preservedCount > 0
            ? `- Prompted placeholders preserved: ${safeSummary.preservedPlaceholderProperties.join(", ")}\n`
            : "";

        return `---
aliases:
tags:
primary-categories:
secondary-categories:
type: "${contentTypeName}"
note-status: "${NOTE_STATUS.DRAFT}"
---
# [[${fallbackTitle}]]

## Overview

> [!warning] Note Creation ${wasCancelled ? "Cancelled" : "Failed"}
> The automated creation workflow ${wasCancelled ? "was cancelled" : "failed"} before the note could be fully assembled.
> Review the properties and body content below before marking this note as ready.

## Recovery Details

- Reason: ${errorMessage}
- Intended destination: ${config.destination || PATHS.CONTENT}/
- Content type: ${contentTypeName}
- Prompted properties filled: ${safeSummary.providedCount}
${preservedSection}
---

${TIMESTAMP}`;
    }

    return `# ${fallbackTitle}

## Overview

> [!warning] Note Creation ${wasCancelled ? "Cancelled" : "Failed"}
> The automated creation workflow ${wasCancelled ? "was cancelled" : "failed"} before the note could be fully assembled.
> Reason: ${errorMessage}

---

${TIMESTAMP}`;
}

/**
 * Main execution function - orchestrates the entire note creation workflow
 * @returns {Promise<string>} - Complete personalized note content or fallback content if workflow fails
 * @throws {Error} - If note creation fails (note configuration, file creation/deletion, appending data to file, etc.)
 */
async function executeNoteCreation() {   
    Logger.info("=== Adversary Simulation Note Creation Started ===", {
        timestamp: new Date().toISOString(),
        currentFile: tp.file.title,
        vaultName: app.vault.getName()
    });
    let config = null;
    
    try {     
	    Logger.debug("Phase 1: Building note configuration"); 
	    config = await buildNoteConfiguration();

        Logger.debug("Phase 2: Building note content");
        const noteContent = await buildNoteContent(config);

        Logger.debug("Phase 3: Moving note to destination");
        await moveNoteToDestination(config);
        showCompletionNotices(config);
        
        Logger.info("=== Adversary Simulation Note Creation Completed Successfully ===", {
            noteType: config.noteType,
            title: config.title,
            contentLength: noteContent.length,
            finalPath: `${config.destination}${config.title}.md`
        });
        
        return noteContent.trimEnd();
        
    } catch (error) {
        Logger.error("Note creation workflow failed", error, {
            phase: "unknown", // Could be enhanced to track current phase
            currentFile: tp.file.title
        });
        if (isCancellationError(error)) {
            showWarning("Note creation was cancelled. Leaving a safe draft in the current file.");
        } else {
            showError("Note creation failed. Leaving a safe draft in the current file. Check console for details.");
        }

		Logger.info("Generating fallback content");
        return createFallbackContent(error, config);
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                                  EXECUTION                                  //
//////////////////////////////////////////////////////////////////////////////////

// Execute the main workflow and return the generated content
const generatedContent = await executeNoteCreation();
%><%* 
tR += `${generatedContent}`;
%>
