<%* 
/**
 * Adversary Simulation Zettelkasten - Automated Note and Category Creation System
 * 
 * This script automates the creation of hierarchical notes for a red team reference guide:
 * - Primary Categories (🥇): High-level topics like "Penetration Test", "Red Team"
 * - Secondary Categories (🥈): Mid-level topics like "Active Directory", "Post-Exploitation" 
 * - Content Notes (⚛️): Atomic notes with specific content types like "Tools", "TTPs", "Payloads"
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
 */
function showNotice(message, noticeType) {
	const emoji = EMOJI_NOTICE[noticeType.toUpperCase()] || "";
	new Notice(`${emoji} ${message}`);
}

function showError(message) { showNotice(message, "error"); }
function showSuccess(message) { showNotice(message, "success"); }
function showSuggestion(message) { showNotice(message, "suggestion"); }
function showWarning(message) { showNotice(message, "warning"); }

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
        const availableCategories = isSecondary ? await getSecondaryCategories() : await getPrimaryCategories();
        
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
            
            if (remainingCategories.length === 0) {
                Logger.debug("No remaining categories to select");
                break;
            }
            
            const options = [...remainingCategories];
            const displayOptions = [...remainingCategories];
            
            // Add "Done" option after first selection
            if (selectedCategories.length > 0) {
                options.push("DONE");
                displayOptions.push("✅ Done (Finish Selection)");
            }
            
            const promptText = selectedCategories.length === 0 
                ? `Select ${categoryLabel} category to link back to:` 
                : `Selected: ${selectedCategories.join(", ")}. Select another or choose Done:`;
            
            Logger.debug("Presenting category selection", {
                promptText,
                optionsAvailable: options.length,
                currentlySelected: selectedCategories
            });
            
            const selection = await tp.system.suggester(displayOptions, options, false, promptText);
            
            if (selection === "DONE" || !selection) {

                Logger.info(`Category selection completed`, {
                    categoryType,
                    finalSelectionCount: selectedCategories.length,
                    selectionRounds: selectionRound,
                    userChoseDone: selection === "DONE"
                });
                continueSelecting = false;
            } else {
                selectedCategories.push(selection);
                Logger.debug(`Category added to selection`, {
                    addedCategory: selection,
                    totalSelected: selectedCategories.length
                });
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

        Logger.debug("Phase 4.3: Creating content type template structure");
	    await createTemplateStructure(typeName, selectedEmoji);
	    
        const newContentType = {
            name: typeName,
            emoji: selectedEmoji,
            displayName: `${selectedEmoji} ${typeName}`,
            searchTag: `${selectedEmoji}${typeName.replace(/\s+/g, '_')}`
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
 * Creates template folder structure and files for new content type based on Basic template
 * @param {string} typeName - Content type name for folder and file naming
 * @param {string} emoji - Selected emoji for search tags and identification
 * @throws {Error} - If template creation fails (folder creation, file copying, or customization)
 */
async function createTemplateStructure(typeName, emoji) {
	// Obsidian handles cross-platform file separators automatically
    const basePath = `${PATHS.CONTENT_TEMPLATES}/${typeName}`;

    Logger.info(`Creating template structure for new content type`, {
        typeName,
        emoji,
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
    constructor(noteType, title) {
        this.config = { 
            noteType, 
            title,
            primaryCategories: [],
            secondaryCategories: []
        };
    }
    
    withPrimaryCategories(categories) {
        this.config.primaryCategories = categories || [];
        return this;
    }
    
    withSecondaryCategories(categories) {
        this.config.secondaryCategories = categories || [];
        return this;
    }
    
    withEmoji(emoji) {
        this.config.emoji = emoji;
        return this;
    }
    
    withContentType(contentTypeConfig) {
        this.config.contentType = contentTypeConfig;
        return this;
    }
    
    build() {
        return this._addPathsAndTemplates(this.config);
    }
    
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
                return {
                    destination: `${PATHS.CONTENT}/`,
                    metadataTemplate: `[[04 - Templates/04 - Content/${config.contentType.name}/Metadata]]`,
                    bodyTemplate: `[[04 - Templates/04 - Content/${config.contentType.name}/Body]]`,
                    footerTemplate: `[[04 - Templates/04 - Content/${config.contentType.name}/Footer]]`
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
 * @param {Object} config - Note configuration object containing template paths (metadataTemplate, bodyTemplate, footerTemplate)
 * @returns {Promise<Object>} - Template content object with metadata, body, and optionally footer properties
 * @throws {Error} - If any required template file cannot be loaded
 */
async function loadNoteTemplates(config) {
	Logger.info("Loading note templates", {
        metadataTemplate: config.metadataTemplate,
        bodyTemplate: config.bodyTemplate,
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

        if (config.footerTemplate) {
            Logger.debug("Loading footer template");
            templates.footer = await tp.file.include(config.footerTemplate);
            Logger.debug(`Footer template loaded (${templates.footer.length} chars)`);
        }

        Logger.info("All templates loaded successfully", {
            templateCount: Object.keys(templates).length,
            totalChars: Object.values(templates).reduce((sum, t) => sum + t.length, 0)
        });

        return templates;
    } catch (error) {
        Logger.error("Template loading failed", error, {
            metadataTemplate: config.metadataTemplate,
            bodyTemplate: config.bodyTemplate,
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
            secondaryCategoriesCount: config.secondaryCategories?.length || 0
        });
        
        customizedMetadata = customizeCategoriesMetadata(customizedMetadata, config.primaryCategories, 'primary');
        transformationsApplied++;
        
        customizedMetadata = customizeCategoriesMetadata(customizedMetadata, config.secondaryCategories, 'secondary');
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
            pattern: /primary categories:\r?\n  - Add link\(s\) \[\[\]\] back to related PRIMARY categories/,
            replacement: `primary categories:\n  - ${categories.join('\n  - ')}`
        },
        'secondary': {
            pattern: /secondary categories:\r?\n  - Add link\(s\) \[\[\]\] back to related SECONDARY categories/,
            replacement: `secondary categories:\n  - ${categories.join('\n  - ')}`
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

        Logger.debug("Metadata customization results", {
            originalLength: templates.metadata.length,
            customizedLength: customizedMetadata.length,
            changesMade: customizedMetadata !== templates.metadata
        });

        Logger.debug("Phase 3.3: Assembling final content");
		const finalContent = assembleNoteContent(customizedMetadata, templates, config);

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
 * @param {Object} templates - Template content object with metadata, body, and optional footer
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
            contentParts.push(templates.body, templates.footer, TIMESTAMP);
            componentsAdded += 3;
            Logger.debug("CONTENT note components added", {
                bodyLength: templates.body?.length || 0,
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
    
    const finalContent = contentParts.join(DIVIDER);
    
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

	try {
		Logger.debug("Phase 1.1: Selecting note type");
	    const noteType = await selectNoteType();
	    Logger.info(`Selected note type: ${noteType}`);

		Logger.debug("Phase 1.2: Saving validated note title");
	    const title = await getValidatedNoteTitle(noteType);
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

                Logger.debug("PRIMARY configuration completed", { emoji });
	            return builder.withEmoji(emoji);
	        },
	        [NOTE_TYPES.SECONDARY]: async () => {
                Logger.debug("Executing SECONDARY category configuration strategy");
	            const primaryCategories = await selectCategories("primary");

                Logger.debug("SECONDARY configuration completed", {
                    primaryCategoriesCount: primaryCategories.length
                });
	            return builder.withPrimaryCategories(primaryCategories);
	        },
	        [NOTE_TYPES.CONTENT]: async () => {
                Logger.debug("Executing CONTENT note configuration strategy");
		        const primaryCategories = await selectCategories("primary");
		        const secondaryCategories = await selectCategories("secondary");
		        const contentType = await selectContentType();

                Logger.debug("CONTENT configuration completed", {
                    primaryCategoriesCount: primaryCategories.length,
                    secondaryCategoriesCount: secondaryCategories.length,
                    contentTypeName: contentType.name
                });

	            return builder
	                .withPrimaryCategories(primaryCategories)
	                .withSecondaryCategories(secondaryCategories)
	                .withContentType(contentType);
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
 * @throws {Error} - If file move operation fails (permissions, path issues, etc.)
 */
async function moveNoteToDestination(config) {
	Logger.info("====== Phase 2: Note Relocation Started ======");
	const destinationPath = `${config.destination}${config.title}`;

	Logger.debug(`Phase 2.1: Moving note to destination`, {
        from: tp.file.path(true),
        to: destinationPath
    });
	
	try {
        await tp.file.move(destinationPath);
        Logger.debug(`File successfully moved to: ${destinationPath}`);
    	Logger.info("====== Phase 2: Note Relocation Completed Successfully ======");
        showSuccess(`Note moved to ${config.noteType} directory`);
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
 * Creates fallback content when the main workflow fails to ensure user gets usable note
 * @returns {string} Basic note content with error message and timestamp
 */
function createFallbackContent() {
    const fallbackTitle = tp.file.title || "New Note";
    return `# ${fallbackTitle}

## Overview

*Note creation encountered an error. Please try again or create manually.*

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
    
    try {     
	    Logger.debug("Phase 1: Building note configuration"); 
        const config = await buildNoteConfiguration();

        Logger.debug("Phase 2: Moving note to destination");
        await moveNoteToDestination(config);

        Logger.debug("Phase 3: Building note content");
        const noteContent = await buildNoteContent(config);
        
        Logger.info("=== Adversary Simulation Note Creation Completed Successfully ===", {
            noteType: config.noteType,
            title: config.title,
            contentLength: noteContent.length,
            finalPath: `${config.destination}${config.title}.md`
        });
        
        showSuccess(`${config.noteType} "${config.title}" created successfully!`);
        return noteContent.trimEnd();
        
    } catch (error) {
        Logger.error("Note creation workflow failed", error, {
            phase: "unknown", // Could be enhanced to track current phase
            currentFile: tp.file.title
        });
        showError("Note creation failed. Check console for details.");

		Logger.info("Generating fallback content");
        return createFallbackContent();
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
