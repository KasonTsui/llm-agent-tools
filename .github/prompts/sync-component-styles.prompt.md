---
agent: agent
model: Gemini 3 Pro (Preview) (copilot)
description: Sync Component Style and Structure
---
# Sync Component Style and Structure

I have updated the `${{section_name}}` (e.g., card-header) of `${{source_component}}`.

Please sync the style and structure of `${{section_name}}` in the following components to match `${{source_component}}` (both HTML and SCSS).

## Source
- **Component**: `${{source_component}}`
## Targets
- ${list_of_target_components}

## Requirements
1.  **Match Style & Structure**: Ensure the HTML structure and SCSS classes match the source component.
2.  **Coding Style**: Keep the coding style consistent with the source.
3.  **Preserve Content**:
    - **Icons**: Leave the same FontAwesome icon (`faicon`) as currently exists in the target (do not copy the source's icon).
    - **Buttons**: Some headers may not have buttons. If a target has a button, keep it. If it doesn't, don't add one.
    - **Titles**: Keep the existing titles.