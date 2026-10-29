<div align="center">

# REW MCP Server

**AI-powered room acoustics analysis for studio monitoring**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)
[![Node.js](https://img.shields.io/badge/node-%3E%3D20-brightgreen?style=flat-square&logo=node.js)](https://nodejs.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-blue?style=flat-square&logo=typescript)](https://www.typescriptlang.org/)
[![CI](https://img.shields.io/github/actions/workflow/status/fl0zan0/rew-mcp/ci.yml?style=flat-square&label=CI)](https://github.com/fl0zan0/rew-mcp/actions)
[![Tests](https://img.shields.io/badge/tests-884_passing-brightgreen?style=flat-square)](https://github.com/fl0zan0/rew-mcp/actions)

A [Model Context Protocol](https://modelcontextprotocol.io) server that connects Claude to [Room EQ Wizard](https://www.roomeqwizard.com), turning raw measurements into actionable acoustics guidance.

[Getting Started](#-getting-started) · [Tools](#-tools) · [Plugin](#-claude-code-plugin) · [Docs](#-documentation) · [Contributing](#-contributing)

</div>

> [!NOTE]
> **This is a fork of [koltyj/rew-mcp](https://github.com/koltyj/rew-mcp).** The `rew-mcp` package on npm (1.0.0) predates upstream fixes and does not work with Claude Code (tool names contain dots). This fork tracks upstream `main` and adds a fix for REW's `/measurements` response format (see [CHANGELOG](CHANGELOG.md)). Install it from GitHub as shown below, not from npm.

---

## What is this?

REW MCP Server lets an LLM control Room EQ Wizard, analyze your measurements, and coach you through room treatment and speaker placement — step by step, one recommendation at a time.

**Instead of staring at graphs, you get answers:**

> *"You have a 12 dB null at 83 Hz caused by speaker boundary interference. Your monitors are 0.97 m from the rear wall — move them to 1.4 m or 0.6 m to shift the cancellation frequency out of the critical listening range. GLM cannot fix this; it's a physics problem."*

### Key capabilities

- **Guided calibration** — Mic gain staging, SPL calibration to 85 dB reference, L/R/Sub measurement sessions
- **Plain language analysis** — Room modes, SBIR, reflections, symmetry issues explained with causes and fixes
- **GLM transparency** — What Genelec's DSP corrected vs what it physically cannot
- **Placement optimization** — One change at a time, measure after each, validate the improvement
- **27 MCP tools** — Full programmatic control of REW's API: measurements, signal generator, SPL meter, RTA, EQ, and more

---

## Getting Started

### Requirements

- **Node.js 20+**
- **[Room EQ Wizard](https://www.roomeqwizard.com)** running with API enabled (`-api` flag or Preferences)
- **Measurement microphone** (calibrated preferred — UMIK-1, UMIK-2, etc.)

### Install

<table>
<tr>
<td width="50%">

**Claude Desktop**

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "rew-mcp": {
      "command": "npx",
      "args": ["-y", "github:fl0zan0/rew-mcp"]
    }
  }
}
```

</td>
<td width="50%">

**Claude Code**

```bash
claude mcp add rew-mcp -- npx -y github:fl0zan0/rew-mcp
```

</td>
</tr>
<tr>
<td>

**Cursor**

[![Install in Cursor](https://img.shields.io/badge/Cursor-Install_MCP-black?style=flat-square&logo=cursor)](https://cursor.com/install-mcp?name=rew-mcp&config=eyJjb21tYW5kIjoibnB4IiwiYXJncyI6WyIteSIsImdpdGh1YjpmbDB6YW4wL3Jldy1tY3AiXX0%3D)

Or: Settings > Features > MCP Servers > `npx -y github:fl0zan0/rew-mcp`

</td>
<td>

**VS Code**

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_MCP-0098FF?style=flat-square&logo=visualstudiocode)](https://insiders.vscode.dev/redirect/mcp/install?name=rew-mcp&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22github%3Afl0zan0%2Frew-mcp%22%5D%7D)

</td>
</tr>
</table>

The first launch takes a little longer: npm clones the repository and builds it.

---

## Example Workflow

```
You:    "Help me calibrate my studio monitors"

Claude: Connects to REW → checks your mic levels → calibrates to 85 dB SPL
        → measures Left, Right, Sub → analyzes the room → finds issues:

        1. 14 dB mode at 62 Hz (axial mode, fixable with bass trapping)
        2. 8 dB SBIR null at 120 Hz (move speakers 15 cm from rear wall)
        3. 3 dB L/R asymmetry above 2 kHz (side wall reflection)

        → guides you through fixes one at a time
        → re-measures after each change
        → validates improvement: "62 Hz peak reduced from 14 dB to 4 dB ✓"
```

---

## Tools

### Offline Analysis

| Tool | What it does |
|------|-------------|
| `rew_analyze_room` | Full-room analysis with prioritized, fixability-ranked recommendations |
| `rew_analyze_room_modes` | Detect peaks, nulls, and correlate with theoretical room modes |
| `rew_analyze_decay` | RT60, EDT, and ringing analysis (ISO 3382) |
| `rew_analyze_impulse` | Early reflection detection and path estimation |
| `rew_compare_measurements` | Before/after, L/R symmetry, multi-position comparison |
| `rew_compare_to_target` | Score response against flat, Harman, REW room, or custom curves |
| `rew_interpret_with_glm_context` | What GLM can fix vs what requires physical changes |
| `rew_average_measurements` | Spatial averaging (RMS, vector, hybrid methods) |
| `rew_optimize_room` | One-at-a-time placement optimization with validation |
| `rew_ingest_measurement` | Parse REW text exports for offline analysis |

### REW API Control

| Tool | What it does |
|------|-------------|
| `rew_api_connect` | Connect to REW's REST API |
| `rew_api_audio` | Configure input/output devices and sample rate |
| `rew_api_check_levels` | Verify mic gain (clipping/optimal/low zone feedback) |
| `rew_api_calibrate_spl` | Semi-automated SPL calibration to target level |
| `rew_api_measure` | Trigger sweeps, SPL readings, or configure measurement |
| `rew_api_measure_workflow` | Orchestrated setup + level check + calibration + measurement |
| `rew_api_measurement_session` | Stateful L/R/Sub measurement sequence |
| `rew_api_generator` | Signal generator: pink noise, sweeps, tones |
| `rew_api_spl_meter` | Live SPL metering (A/C/Z weighting, Slow/Fast/Impulse) |
| `rew_api_rta` | Real-time analyzer control and capture |
| `rew_api_list_measurements` | List loaded measurements |
| `rew_api_get_measurement` | Fetch measurement data by UUID |
| `rew_api_import` | Import measurement files into REW |
| `rew_api_measurement_commands` | Execute per-measurement REW commands |
| `rew_api_measurement_eq` | Manage per-measurement EQ, filters, and targets |
| `rew_api_eq` | Global EQ defaults, house curves |
| `rew_api_groups` | Measurement group management |

### Prompts & Resources

<details>
<summary><strong>MCP Prompts</strong> — Pre-built workflow templates</summary>

| Prompt | Description |
|--------|-------------|
| `rew_calibration_full` | Complete end-to-end calibration workflow |
| `rew_gain_staging` | Standalone mic gain and level calibration |
| `rew_measurement_workflow` | Session-aware L/R/Sub measurement sequence |
| `rew_optimization_workflow` | Iterative placement optimization loop |

</details>

<details>
<summary><strong>MCP Resources</strong> — Dynamic session and measurement data</summary>

| URI Template | Description |
|--------------|-------------|
| `session://{session_id}` | Session state, step, and captured measurements |
| `measurement://{measurement_id}` | Stored frequency response data and metadata |
| `recommendations://{session_id}` | Active optimization recommendations |
| `history://{session_id}` | Measurement history and summaries |

</details>

---

## Claude Code Plugin

For [Claude Code](https://docs.anthropic.com/en/docs/claude-code) users, the included plugin adds guided workflows, domain knowledge, and quality automation on top of the MCP server.

```bash
# From a cloned repo
claude --plugin-dir ./plugin

# Or installed globally from GitHub (installs both the MCP server and plugin)
npm install -g github:fl0zan0/rew-mcp
claude --plugin-dir $(npm root -g)/rew-mcp/plugin
```

### Commands

| Command | Description |
|---------|-------------|
| `/rew:calibrate` | Full calibration workflow — levels, SPL, L/R/Sub, analysis, optimization |
| `/rew:analyze` | Analyze measurements with prioritized recommendations |
| `/rew:optimize` | Iterative placement optimization cycle |
| `/rew:status` | Session state and next recommended step |

### Skills & Automation

| Component | What it provides |
|-----------|-----------------|
| **room-acoustics** skill | Room modes, SBIR, treatment, GLM knowledge, target curves |
| **rew-workflows** skill | Tool chaining patterns, workflow sequencing, best practices |
| **measurement-reviewer** agent | Proactive data quality review after measurements |
| **workflow-enforcement** hook | Soft-warns when tools are called out of recommended order |

See [`plugin/README.md`](plugin/README.md) for full details.

---

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/architecture.md) | System design and data flow |
| [GLM Context](docs/glm-context.md) | How Genelec GLM works and its limitations |
| [Analysis Rules](docs/analysis-rules.md) | Detection algorithms and thresholds |
| [File Formats](docs/file-formats.md) | REW export format specifications |
| [Resources](docs/resources.md) | MCP resource URIs and payload shapes |
| [Examples](docs/examples.md) | End-to-end measurement and analysis workflows |

---

## Development

```bash
npm install          # Install dependencies
npm run build        # Build TypeScript
npm test             # Run tests (884 passing)
npm run test:coverage # With coverage report
npm run lint         # ESLint
npm run dev          # Watch mode
```

## Contributing

Contributions welcome. See [Contributing Guide](CONTRIBUTING.md) and [Code of Conduct](CODE_OF_CONDUCT.md).

```bash
git checkout -b feature/your-feature
npm test
git commit -m 'feat: add your feature'
git push origin feature/your-feature
# Open a Pull Request
```

## License

[MIT](LICENSE) © Kolton Jacobs

---

<div align="center">

**[Changelog](CHANGELOG.md)** · **[Report Bug](https://github.com/koltyj/rew-mcp/issues)** · **[Request Feature](https://github.com/koltyj/rew-mcp/issues)**

</div>
