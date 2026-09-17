# Case Study: A Dashboard Setting Silently Produced a Malformed Launch Flag

## Situation

Enabling Valheim's native "portal anything" world modifier (lets ores and restricted items pass through portals) through AMP's dashboard setting didn't appear to take effect after a restart.

## Investigation

Checking the actual running server process's command line (not just trusting the dashboard showed the setting as "on") showed a malformed `-modifier casual` launch flag — missing a required category keyword the game actually expects. The dashboard had accepted `"casual"` as a valid value without validating it against the real expected format.

Reading AMP's own Valheim template definition (`valheimconfig.json`) directly on the box showed the real expected value: the full string `"portals casual"` (or `"portals hard"` / `"portals veryhard"`), since AMP does a raw `-modifier {value}` substitution rather than composing the flag itself.

## Root cause

The management UI didn't validate the setting's value against what the underlying game process actually required, so an incomplete value was silently accepted and produced a broken launch flag.

## Corrective action

Set the value to the full string `"portals casual"`, restarted, and confirmed `-modifier portals casual` in the live process's command line — verifying against the actual running process rather than the dashboard's saved value a second time.

## Lessons

- A saved setting and a setting that's actually working are two different claims — the only way to know which one you have is to check the live process, not the UI that configured it.
- When a dashboard abstracts over a raw command-line flag, reading the platform's own template/source for that abstraction beats guessing at the expected format.
