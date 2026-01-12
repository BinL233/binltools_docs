## Introduction
File Formatting Tool (FFT) is a Microsoft Word add-in designed to improve efficiency, consistency, and accuracy when formatting documents for health authority submissions. 

FFT is helpful for regulatory documents that:
- Under eCTD structure
- Contain extensive tables, figures, and cross-references
- Frequent revisions and re-formatting

Three Advantages:
- Works directly inside Word
-  Applies to any Word documents
-  Keyboard-driven shortcuts

Once FFT is opened in Word, it appears as a task pane on the right side of the document.

## Icons
Before diving into the FFT Word, let's take a step into the pane discovery.

### Announcements
Updated information will be posted in the announcements section, including but not limited to version updates, issues detected/solved, new feature releases, etc.

### Shortcuts Status
Visual check for the Shortcuts function. The details of the Shortcuts function will be covered in section 2.1.

### Settings
Designed to help users get oriented, access documentation, and get in touch with the shortcut function before applying any formatting. 
Click the Settings icon in the top-right corner of the FFT pane. 

From here, users can manage:
- About - check documentation (User Guide), version and contact developer.
- Notification - enable or disable the notification for each action.
- Shortcuts - customize shortcut keys based on preference or reset to default. See 2.1 for more details.

💡 Tips: 
- Clear all the track changes and comments before formatting: File → Info → Check for Issues → Inspect Document → Remove All (Comments, Revisions, and Versions)

### Configure, Format and Generate
Configure, Format, and Generate are the three fundamental processes of FFT, which help a document go from disorderly to ready for submission.

## Step 1 Configure
Step 1 Configure defines how FFT will prepare the document before formatting begins. 
This step allows users to load a predefined FFT template, control layout elements, and set page margins.

### 1.1 Select Module
At the top of Step 1, select the appropriate module or category from the dropdown list (for example, Module 2.5 – Clinical Overview).
The predefined styles are automatically added to the document and are ready for use in the following steps.

### 1.2 Set Up Layout
Provided flexibility in the template load and document layout setup.

#### 1.2.1 Load Templates
The principle of FFT template configuration is to deploy the preset styles (named "FFT XXX" in the Styles) and field into a Word document, and it only needs to be deployed once. 
The document needs to be opened several times for editing, but users do not have to deploy the FFT styles each time. 
Therefore, the design of the Load Templates helps the users decide whether to reload the template.

- ON (recommended for first use):
   Loads FFT-defined header, footer, and styles
   Initializes the document with the selected module template

- OFF (recommended when reopening documents):
   Keeps the existing FFT configuration
   Allows users to proceed directly to Step 2

💡 Tips: 
- If the document already contains an FFT template, turning this option OFF prevents unnecessary reloading
- If a wrong FFT template been deployed in the document, users need to delete all the FFT Styles manually in Styles. (Styles - Manage Styles - Import/Export - Select all the FFT Styles - Delete)

#### 1.2.2 Load Header / Load Footer
These options allow fine control over whether FFT should overwrite existing headers and footers.

- Load Header
   ON: FFT applies the predefined header from the selected module
   OFF: Keeps the document’s existing header unchanged

- Load Footer
   ON: FFT applies the predefined footer from the selected module
   OFF: Keeps the document’s existing footer unchanged

💡 Tips: 

Turn OFF these options if
- The document already contains an approved or validated header/footer
- Users are working on a late-stage document and want to preserve layout

#### 1.2.3 Page Margin Settings
Users can manually define page margins (in inches):
- Top Margin (default: 1)
- Bottom Margin (default: 0.67)
- Left Margin (default: 1.1)
- Right Margin (default: 0.9)

The specified margins will be applied when formatting begins.

## Step 2 Format
Step 2 Format is where FFT applies formatting to document content.
This step becomes available only after Step 1 Configure has been completed and submitted.

Step 2 contains three independent formatting sections:
- 2.1 Styles – headings and paragraphs
- 2.2 Fix Tables – table cell formatting
- 2.3 Fix Symbols – symbol normalization

All actions in Step 2 can be undone using Ctrl + Z.

### 2.1 Styles
Section 2.1 is used to apply predefined FFT styles to headings and paragraphs.

Each style is mapped to:
- A button in the FFT pane
- A keyboard shortcut, shown in grey parentheses on the button

Both methods apply identical formatting.

#### 2.1.1 Available Styles
- Normal [X] – Body text, justified
- Normal 2 [C] – Body text, left-aligned
- Heading 1–5 [1–5] – Section headings
- Table Title [6] – Table captions
- Figure Title [7] – Figure captions
- Bullet Point [B] – Bulleted lists
- Numbering [V] – Numbered lists

#### 2.1.2 Method A – Formatting Using Keyboard Shortcuts (Recommended)
The Keyboard-driven shortcuts is one of the most powerful features of FFT. 
It allows users to format paragraph entirely by keyboard, significantly improve efficiency and eliminate repetitive formatting tasks.

Three Key principles:
- Pre-defined Word Styles - all shortcuts applied one key to one style, and aligned with health authority eCTD expectations.
- Sequential Traversal - one paragraph at a time, auto-skip all the table and figure, focus on plain-text only.
- Shortcut Mode - active only when the floating window is in green with "Shortcut On" and apply only to the navigated paragraph.

#### Step-by-Step Paragraph Formatting
1. Place the Cursor

   Click anywhere inside the paragraph that needs to be formatted. 

   The cursor must be inside plain text, not inside a table or figure.

2. Activate Shortcut Mode

   Move the cursor to the FFT pane and click once.

   The floating indicator will change from: “Shortcut Off” (grey) → “Shortcut On” (green)

   This confirms that shortcuts are active.

3. Navigate to the current paragraph

   Press [N] on the keyboard. (Think about "Navigate")

   FFT will select the paragraph containing the cursor and highlight it in grey.

4. Apply the Desired Style

   Press the shortcut key corresponding to the target style (as shown in grey parentheses on the style buttons).

   The style is applied immediately, and the selection remains on the same paragraph.

5. Move to the Next paragraph

   Press [→] to move to the next paragraph.

   Repeat step 4 and 5 until the entire document is formatted.

#### Default Shortcuts
- [N] – Navigate to the currently selected location
- [←] - Move to the previous paragraph
- [→] - Move to the next paragraph
- [X] - Normal (justify)
- [C] - Normal 2 (align left)
- [1~5] - Heading 1 to Heading 5
- [6] - Table title
- [7] - Figure title
- [B] - Bullets
- [V] - Numbering

💡 Tips:
- FFT automatically skips tables and figures during shortcut navigation.
- Shortcuts Keys can be customized in the Settings.

#### 2.1.2 Method B – Formatting Using Style Buttons
Users may also format content using the mouse:
- Select a paragraph using the cursor.
- Click the desired style button in the FFT pane.

Example: Select a table caption and click Table Title [6] to apply the predefined table title style.

### 2.2 Fix Tables
Section 2.2 is designed specifically for table formatting. Tables are excluded from shortcut traversal because:
- Table structures vary significantly between documents
- Accurate formatting requires explicit user-defined scope

#### 2.2.1 Table Formatting Principle
Users define a target area within a table by selecting:
- The top-left cell
- The bottom-right cell

FFT formats only the cells inside this defined range.

#### 2.2.2 Step-by-Step Table Formatting
1. Define the Start of the Target Area

   Click inside the top-left cell of the area to be formatted.

   Click Submit.

2. Define the End and Apply Style

   Click inside the bottom-right cell of the target area.
   
   Set the desired Text Size (default: 10Pt).

   Choose the appropriate cell type.

#### 2.2.3 Cell Style Options
Cells without indentation (more reliable)
- Header Cells
- Data Cells (Text)
- Data Cells (Numerals)

This option fully normalizes formatting, including indentation.

Cells with indentation
- Header Cells
- Data Cells (Text)
- Data Cells (Numerals)

This option preserves existing indentation and is recommended for hierarchical tables.

Example:

Use “Cells with indentation” when indentation conveys hierarchy or categorization.

### 2.3 Normalize Symbols
Section 2.3 normalizes full-width (Chinese) symbols into ASCII (half-width) symbols. One click for the entire document.

Examples:
- （ ） → ( )
- ， → ,
- 。 → .
- ： → :

This action applies to the entire document and cannot be limited to a selected range.