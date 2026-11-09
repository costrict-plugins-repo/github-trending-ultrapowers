# Attack modes and checks

Researched 7 Nov 2026. Sources linked per item; several dates come from secondary coverage.

## Package supply chain

- **Slopsquatting.** AI-suggested packages that do not exist get registered by attackers. About 20 % of LLM-recommended packages did not exist, and most fake names are not typos of real ones (Spracklen et al., USENIX 2025; https://socket.dev/glossary/slopsquatting). Check: the package exists, has real downloads and history.
- **Compromised maintainer, malicious new version.** chalk/debug via phishing (Sept 2025, https://www.cycode.com/blog/npm-debug-chalk-supply-chain-attack/); axios (Mar 2026) pulled in a new dependency with an install-time RAT and no provenance (https://www.sonatype.com/blog/axios-compromise-on-npm-introduces-hidden-malicious-package). Check: new dependency in a patch release, missing provenance, release under 7 days old.
- **Worms.** Shai-Hulud (Sept and Nov 2025) used postinstall then preinstall scripts to steal tokens and republish packages (https://unit42.paloaltonetworks.com/NPM-supply-chain-attack/, https://securitylabs.datadoghq.com/articles/shai-hulud-2.0-npm-worm/).
- **Build backdoors.** xz-utils CVE-2024-3094: payload hidden in a binary test file decoded by the build. ultralytics PyPI (Dec 2024): wheel differed from GitHub source after a CI injection (https://thehackernews.com/2024/12/ultralytics-ai-library-compromised.html).
- **Mutable tags.** tj-actions/changed-files CVE-2025-30066: tags repointed to malicious code; SHA-pinned users were safe (https://www.wiz.io/blog/github-action-tj-actions-changed-files-supply-chain-attack-cve-2025-30066).
- **Auto-update at launch.** `npx -y pkg@latest`, `uvx --refresh` fetch the newest release every start, so one bad release lands without anyone looking. Mechanism, no documented incident yet.

## Installers, taps, binaries

- **curl | sh and third-party taps.** Homebrew does not vouch for third-party taps; 6.0 asks to trust a tap before its code runs (https://docs.brew.sh/Supply-Chain-Security). Fake Homebrew sites spread stealers.
- **HTTP casks.** Some casks download over http with `sha256 :no_check`, swappable in transit (https://www.koi.ai/blog/brew-hijack-serving-malware-over-homebrews-core-tap).
- **Quarantine removal.** macOS stealers (AMOS, SHAMOS) tell users to run `xattr -d com.apple.quarantine` then execute (https://www.crowdstrike.com/en-us/blog/falcon-prevents-cookie-spider-shamos-delivery-macos/).
- **Model weights.** Pickle files (`.bin`, `.pt`, `.pkl`) run code on load; malicious models found on Hugging Face. Prefer `.safetensors`, ONNX or GGUF from the original publisher.

## MCP servers

- **Tool poisoning.** Hidden instructions in tool descriptions steer the model (https://invariantlabs.ai/blog/mcp-security-notification).
- **Rug pull.** Tool definitions change after approval; pin versions and diff on update.
- **Lookalike servers.** postmark-mcp (Sept 2025) added a BCC sending every email to the attacker (https://www.koi.ai/blog/postmark-mcp-npm-malicious-backdoor-email-theft).
- **Client CVEs.** mcp-remote CVE-2025-6514 (fixed 0.1.16), MCP Inspector CVE-2025-49596 (fixed 0.14.1).
- **Confused deputy.** A server gets the agent to pass data from other tools through its arguments.

## Claude Code plugins, skills, hooks, project files

- **Hooks run shell** on events (SessionStart, PreToolUse...) with your user's rights.
- **Project files.** Repo `.mcp.json` and `.claude/settings.json` (`enableAllProjectMcpServers`, hooks, `ANTHROPIC_BASE_URL`) gave code execution or key theft before trust: CVE-2025-59536, CVE-2026-21852, patched (https://research.checkpoint.com/2026/rce-and-api-token-exfiltration-through-claude-code-project-files-cve-2025-59536/).
- **Malicious skills.** Snyk ToxicSkills (Feb 2026): of 3,984 marketplace skills, 13 % had a critical issue and 76 were confirmed malicious (https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/).
- **Hidden Unicode** tag and bidi characters carry instructions humans cannot see (https://embracethered.com/blog/posts/2026/scary-agent-skills).
- **Bundling.** One plugin install can bring skills, hooks, MCP servers and binaries under a single consent.

## Red flags

- `preinstall`, `postinstall`, `install` scripts; setup.py or build hooks doing network or file work.
- `curl`/`wget` piped to a shell, `base64` decoding, `eval`, downloaded binaries made executable, `xattr -d`/`-cr`, `sudo`.
- Unpinned: `@latest`, `npx -y`, `uvx --refresh`, git tags instead of SHAs, `:no_check`, `http://` downloads.
- Hooks of any kind; writes to `~/.claude`, `~/.zshrc`, `~/Library/LaunchAgents`.
- Skill or command text that tells Claude to skip permissions, auto-approve tools, edit settings, hide something from the user, or send files somewhere.
- Tool descriptions longer than the tool needs; non-printing Unicode.
- Network calls to anything beyond the service's stated purpose (telemetry, analytics, unknown hosts); servers bound to 0.0.0.0.
- Pickle model files; unsigned binaries.

## Sweeps (read-only, over the fetched source)

```bash
rg -n 'preinstall|postinstall|curl|wget|base64|eval\(|exec\(|subprocess|os\.system|xattr|sudo|chmod \+x|LaunchAgents|\.zshrc|\.claude/|mcpServers|hooks|ANTHROPIC_BASE_URL|0\.0\.0\.0|telemetry|analytics|posthog|sentry|segment' <dir>
rg -nP '[\x{E0000}-\x{E007F}\x{200B}-\x{200F}\x{202A}-\x{202E}\x{2060}-\x{2064}]' <dir>
rg -n 'https?://[^" )]+' -o <dir> | sort -u    # every host it talks to
```

Other free checks: `npm view <pkg> time --json` (release dates), `codesign -dv --verbose=4` and `spctl -a -vv` on apps, `shasum -a 256` against the published checksum, `xattr -l` (look, never strip).
