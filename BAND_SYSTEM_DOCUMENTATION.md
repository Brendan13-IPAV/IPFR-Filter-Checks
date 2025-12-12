# IP Navigator Review Tool - Band System Implementation

## Overview
The review tool now uses a **relevance band system** instead of raw score grouping. Strategies are automatically distributed into 5 bands based on their calculated scores, and users can move strategies between bands with automatic balancing.

---

## Relevance Bands

### Band Distribution (Configurable in Code)
- **Highly Likely**: Top 20% of strategies (minimum 3)
- **Likely**: Next 20%
- **Possible**: Next 20%
- **Unlikely**: Next 20%
- **Not Applicable**: Strategies with score = 0, or manually moved here

### Visual Indicators
- **Green badge**: Highly Likely
- **Blue badge**: Likely
- **Orange badge**: Possible
- **Red badge**: Unlikely
- **Grey badge**: Not Applicable

---

## Key Features

### 1. Automatic Band Balancing
When a user moves a strategy UP to a band that's at capacity:
- The **lowest-scoring strategy** in the target band automatically moves down
- Manually moved cards show a **green highlight + "MOVED" badge**
- Auto-moved cards show a **subtle grey highlight + "AUTO" badge**

### 2. Ghost Placeholders
Each band (except N/A) shows **dashed outline placeholders** for empty slots, making it clear:
- How many strategies should be in each band
- That users can't overload the top bands
- The target distribution

### 3. Random Order Within Bands
Strategies are displayed in **random order** within each band, not by score. This emphasizes that all strategies in a band are equally relevant.

### 4. Score Display
Actual calculated scores are shown on each card for reference, even though grouping is by band.

### 5. Movement Controls
- **Up Arrow (▲)**: Move to higher relevance band
- **Down Arrow (▼)**: Move to lower relevance band
- Arrows are **disabled** at boundaries (can't go higher than Highly Likely, lower than N/A)

### 6. N/A Special Handling
Moving a strategy to "Not Applicable" requires:
- A written categorical reason
- Example reasons: "Requires trade mark registration which client doesn't have", "Strategy only applies to patents"

### 7. Batch Submission
- All moves accumulate as "pending changes"
- **"Submit Changes (X pending)"** button shows count
- One click submits all changes
- Each movement is recorded as a separate row in Google Sheets

---

## Google Sheets Data Structure

### Columns Sent for Each Movement:
1. **Timestamp**: When submitted
2. **User_Name**: From login
3. **Situation_Filter**: e.g., "Enforcement"
4. **IP_Rights_Filter**: e.g., "Patent"
5. **Accordion_Filters**: JSON array of active accordion filters
6. **Strategy_ID**: Strategy ID number
7. **Strategy_Title**: Strategy name
8. **Old_Band**: e.g., "Likely"
9. **New_Band**: e.g., "Highly Likely"
10. **Old_Score**: Original calculated score
11. **New_Score**: Approximate target score for new band
12. **Move_Type**: "manual", "automatic", or "flag"
13. **Reason**: Text reason (for N/A moves and flags)
14. **Session_ID**: Unique session identifier
15. **Strategies_Version**: Version of strategies data
16. **Metadata_Version**: Version of metadata
17. **Filters_Version**: Version of filter options
18. **Presets_Version**: Version of accordion presets
19. **Data_Load_Timestamp**: When data was loaded

---

## Configuration

### Adjusting Band Percentages (in code)
```javascript
const BAND_CONFIG = {
    highlyLikely: { percentage: 0.20, minCount: 3, label: 'Highly Likely', cssClass: 'highly-likely' },
    likely: { percentage: 0.20, minCount: 0, label: 'Likely', cssClass: 'likely' },
    possible: { percentage: 0.20, minCount: 0, label: 'Possible', cssClass: 'possible' },
    unlikely: { percentage: 0.20, minCount: 0, label: 'Unlikely', cssClass: 'unlikely' },
    notApplicable: { label: 'Not Applicable', cssClass: 'not-applicable' }
};
```

**To change band sizes:**
1. Edit the `percentage` values (must sum to ≤ 1.0)
2. Set `minCount` to ensure minimum strategies in a band
3. Save and refresh the page

---

## User Workflow

### Reviewing Strategies
1. **Log in** with name and password
2. **Apply filters** (situation type, IP rights, accordion options)
3. **Review bands** - strategies are displayed in relevance groups
4. **Move strategies** using ▲/▼ arrows if they're in the wrong band
5. **Flag issues** using the 🚩 button for general feedback
6. **Submit all changes** when done reviewing

### Understanding Visual Feedback
- **No border**: Original position
- **Green border + "MOVED"**: You moved this here
- **Grey border + "AUTO"**: System moved this to maintain balance
- **Dashed outline**: Empty slot (target not reached)
- **Green ✓ Flagged**: You've submitted feedback for this strategy

---

## Setup Instructions

### 1. Update Google Apps Script
Replace your existing Apps Script with the provided `google-apps-script.js` file:
- Copy the entire script
- Paste into your Google Apps Script project
- Deploy as Web App with "Anyone" access
- Copy the deployment URL
- Update `TELEMETRY_CONFIG.sheetURL` in the HTML file

### 2. Verify Data Files
Ensure all JSON files have version numbers:
```json
{
  "version": "1.0.0",
  "lastUpdated": "2024-12-12",
  "strategies": [...]
}
```

### 3. Test the System
1. Open the HTML file in a browser
2. Log in with the password
3. Apply some filters
4. Move a strategy up or down
5. Check that:
   - Another strategy auto-moves to balance
   - Both show visual indicators
   - Submit button shows "(2 pending)"
6. Click Submit and verify data in Google Sheets

---

## Differences from Previous Version

### Removed Features
- ✗ +/- 0.1 score adjustment buttons
- ✗ Manual score input fields
- ✗ "N/A" toggle button
- ✗ CSV export functionality
- ✗ Score-based grouping

### New Features
- ✓ Relevance band system
- ✓ Automatic balancing with visual feedback
- ✓ Ghost placeholders showing target counts
- ✓ Random order within bands
- ✓ Batch submission of changes
- ✓ Mandatory reason for N/A moves
- ✓ Clear distinction between manual/auto moves

---

## Troubleshooting

### Strategies not moving
- Check that the arrow isn't disabled (greyed out)
- Verify you're not at a boundary (top/bottom band)

### Submit button stays disabled
- Ensure you've moved at least one strategy
- The button shows "(X pending)" when active

### Google Sheets not receiving data
- Verify the Apps Script URL is correct
- Check that deployment is set to "Anyone" access
- Look in browser console for errors

### Bands seem unbalanced
- This is normal - percentages are targets, not strict rules
- With small numbers of strategies, rounding affects distribution
- Ghost placeholders show the target count

---

## Notes for Analysts

When reviewing the Google Sheets data:
- **manual** moves are intentional user decisions - high priority to investigate
- **automatic** moves are system-generated for balance - lower priority
- **flag** entries are general feedback, not band movements
- Multiple entries for the same strategy+session indicate competing opinions
- Empty **Reason** field = movement between scored bands
- Filled **Reason** field = movement to N/A, or flag feedback

Focus analysis on:
1. Strategies with multiple conflicting moves
2. Manual moves with consistent direction across users
3. N/A moves with similar reasons (may indicate missing filter logic)
