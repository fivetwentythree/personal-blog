Of course. This is a classic issue when a site is hosted in a subdirectory. The font path needs to be handled by Hugo to be correct.

The problem is that the CSS is looking for the font at the root of the domain (e.g., `your-site.com/fonts/...`) but it actually lives at `your-site.com/personal-blog/fonts/...`.

Here is the correct and simplest way to fix this by leveraging Hugo's asset pipeline properly. We will put the font definition back in the theme's main CSS file where it can be processed by Hugo.

### Step-by-Step Instructions

#### Task 1: Fix the Font Definition in the Theme's CSS

1.  **File to Edit:** `themes/etch/assets/css/main.css`
2.  **Action:** Find the existing `@font-face` rule at the top of the file and replace it with this corrected and optimized version. This ensures Hugo generates the correct path and adds a performance enhancement.

    ```css
    @font-face {
        font-family: 'Departure Mono';
        src: url('{{ "/fonts/departure-mono.woff2" | absURL }}') format('woff2');
        font-weight: normal;
        font-style: normal;
        font-display: swap;
    }
    ```
    *   `{{ "/fonts/departure-mono.woff2" | absURL }}` is the key part. Hugo will replace this with the full, correct URL to your font file.
    *   `font-display: swap;` is a performance best practice that prevents text from being invisible while the font loads.

#### Task 2: Remove the Conflicting Font Definition

1.  **File to Edit:** `assets/css/custom.css`
2.  **Action:** Delete the entire `@font-face` block from the top of this file. It is now defined correctly in `main.css` and keeping it here will cause conflicts.

    **Remove this entire block from `assets/css/custom.css`:**
    ```css
    @font-face {
        font-family: 'Departure Mono';
        src: url('/fonts/departure-mono.woff2') format('woff2'),
             url('/fonts/departure-mono.woff') format('woff');
        font-weight: normal;
        font-style: normal;
    }
    ```

#### Task 3: Confirm the Title Style (No Change Needed)

The theme already tries to apply the font to the title.

*   In `themes/etch/assets/css/main.css`, this rule exists:
    ```css
    header#banner h2 {
        display: inline;
        font-size: 2.1rem;
        margin: 0 8px 0 0;
        font-family: 'Departure Mono', monospace; /* <-- This applies the font */
    }
    ```
You don't need to change this. Once the font is loading correctly (from Step 1), this rule will automatically work as intended.

---

### Summary for an AI Agent

1.  **In `themes/etch/assets/css/main.css`:** Replace the `@font-face` block with:
    ```css
    @font-face {
        font-family: 'Departure Mono';
        src: url('{{ "/fonts/departure-mono.woff2" | absURL }}') format('woff2');
        font-weight: normal;
        font-style: normal;
        font-display: swap;
    }
    ```

2.  **In `assets/css/custom.css`:** Delete the entire `@font-face` block.

After these changes, rebuild your site, and the title "Lochana Perera" will render in the Departure Mono font.