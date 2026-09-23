## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Build 42 compatibility / B41-to-B42 migration
- [ ] Breaking change
- [ ] Documentation update

## Changes Made
- List specific changes
- Be concise

## Mod Structure
Build 42 requires versioned folders (`42/`, `42.1/`, ...) and/or a `common/` folder — a flat root `media/` is the Build 41 layout and will not load in B42.
- [ ] Files are placed in a valid Build 42 structure (`common/` and/or a version folder)
- [ ] Changes are for the legacy Build 41 layout only (no Build 42 impact)

## Testing
How was this tested?
- [ ] Tested in-game
- [ ] Tested with other mods
- [ ] Tested on Build 42 stable (42.20.x)
- [ ] Tested on Build 42 unstable / beta
- [ ] Tested on Build 41 (legacy, 41.78.x beta branch)

## Screenshots (if applicable)

## Related Issues
Fixes #(issue number)

## Checklist
- [ ] Items/recipes use the current script format (`ItemType`, `craftRecipe`) for Build 42
- [ ] Translation files use the correct format for the target build: JSON (Build 42, no language suffix in filename, UTF-8, `%%` for literal `%`) or legacy Lua `.txt` (Build 41)
- [ ] My code follows the project style
- [ ] I have tested these changes
- [ ] I have updated documentation if needed