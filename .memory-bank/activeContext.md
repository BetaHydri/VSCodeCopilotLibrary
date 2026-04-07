# Active context

## Current work focus

The project reached functional completeness on February 24, 2026. Ongoing expansions include new skills, agents, and documentation improvements. As of March 31, 2026, the project has 8 agents, 11 instruction files (+1 CLI model routing reference), 16 skills, and 5 prompts.

## Recent changes (March 31, 2026)

### Skill updates (2)

| Skill | Date | Change |
|---|---|---|
| `dsc-troubleshooting` | March 31 | +60 lines: WinMgmt restart hang via Invoke-LabCommand (reboot from host), phantom Busy LCM state after force reboot, new section 4.3.1 — ApplyAndAutoCorrect consistency timer interfering with long-running SetScript |
| `mecm-dsc-deployment` | March 31 | +114 lines: DSC auto-correct interference during SCCM installs (SetupWpf detection), file copy double-hop on Server 2025 (robocopy UNC hang), stale CM_S00 database blocking installation |

## Previous changes (March 24–29, 2026)

### New skills (1)

| Skill | Date | Description |
|---|---|---|
| `pdf-to-markdown` | March 29 | Convert PDF files to Markdown using .NET-native PDF parsing in PowerShell — no external tools required. Handles zlib/deflate streams, hex-encoded text operators, Y-coordinate line reconstruction, ISO-8859-1 encoding for German PDFs |

## Earlier changes (March 19–23, 2026)

### New skills (1)

| Skill | Date | Description |
|---|---|---|
| `outlook-calendar-export` | March 23 | Export Outlook calendar entries to Markdown via COM automation: recurring appointments, date range filtering, UTF-8 no-BOM encoding, index generation |

## Earlier changes (March 12–18, 2026)

### New skills (3)

| Skill | Date | Description |
|---|---|---|
| `mecm-dsc-deployment` | March 13 | Deploy and troubleshoot MECM/SCCM via DSC using ConfigMgrCBDsc, CommonTasks, and UpdateServicesDsc in DscWorkshop/Datum environments |
| `winrm-troubleshooting` | March 14 | Debug WinRM connectivity failures: service state recovery, listener config, HTTP.sys conflicts, auth failures, PowerShell Direct fallback |
| `pandoc-docx-export` | March 16 | Export Markdown to DOCX via pandoc with Lua filters for landscape tables, custom column widths, emoji, Mermaid diagrams |

### Skill updates (6)

| Skill | Date | Change |
|---|---|---|
| `datum-configuration` | March 12 | General updates |
| `dsc-troubleshooting` | March 12, 16 | Updated; enhanced PsDscRunAsCredential diagnostics and SCCM 2509 compatibility |
| `mecm-dsc-deployment` | March 16–17 | Enhanced remote runspace behavior, SCCM 2509 post-install troubleshooting, PsDscRunAsCredential handling for ConfigMgrCBDsc |
| `automatedlab-deployment` | March 16 | Added RSAT installation workaround for client OSes using scheduled tasks |
| `pester-patterns` | March 13 | Updated alongside MECM skill |
| `pandoc-docx-export` | March 18 | Content-aware column optimization for table formatting |

### Instruction updates (3)

| File | Date | Change |
|---|---|---|
| `pester.instructions.md` | March 13 | Emphasize using `Start-Process` for detached Pester execution |
| `git.instructions.md` | March 14 | Added AI-assisted commit strategy with attribution and branch naming conventions |
| `markdown.instructions.md` | March 16 | Added guidelines against using ASCII box art for structured data |

### Agent updates

- March 13: Added timestamped response requirement (`[YYYY-MM-DD HH:mm UTC]`) to all 8 agents
- March 14: Software Engineer Agent updated with AI-assisted commit strategy

### README updated

- Added 3 new skills to the Available Skills table (mecm-dsc-deployment, pandoc-docx-export, winrm-troubleshooting), bringing the total from 12 to 15.

## Previously undocumented changes (March 4–8, 2026)

The following were committed before the memory bank was created but were never captured:

### New agents (5)

| Agent | Date | Description |
|---|---|---|
| `legal-researcher.agent.md` | March 4 | German legal research and statement drafting (Mietrecht, BGB) |
| `Technical Troubleshooter Agent.agent.md` | March 5 | Systematic problem diagnosis using hypothetico-deductive method, Google SRE-inspired 6-phase workflow |
| `QC Inspector Agent.agent.md` | March 8 | Quality control inspection for Oil & Gas, Energy, and Industrial sectors |
| `Training Content Writer.agent.md` | March 8 | Generic training and workshop content creation with Bloom's taxonomy |
| `DevOps Training Writer.agent.md` | March 8 | DevOps-specialized training (inherits from Training Content Writer) |

### New instruction

| File | Date | Description |
|---|---|---|
| `copilot-model-selection-instructions.md` | March 8 | Model selection rules for Copilot CLI — 4-tier routing (Executors, Implementers, Tech Leads, Architects) with delegation policy |

## Next steps

- Consider adding more skills (e.g., Docker, CI/CD patterns, Azure DevOps)
- Consider adding language instructions for Python, JavaScript, TypeScript
- Consider creating a GitHub Actions or Azure Pipelines CI to lint the instruction files
- Monitor VS Code updates for new Copilot extensibility features
- Memory Bank maintenance is ongoing

## Active decisions and considerations

- **Model choice**: Claude Opus 4.6 is the current default for agents. Model selection instructions provide tiered routing for CLI usage.
- **OneDrive path**: Uses `~/OneDrive/MyCopilot/` which assumes the standard OneDrive folder location.
- **No CI/CD**: This is a configuration repository. Markdown linting could be added.

## Important patterns and preferences

- **Instruction files are comprehensive, not minimal** — Each covers the full breadth of best practices for its language/domain.
- **Agents use zero-confirmation policies** — All agents are designed to execute autonomously without asking for permission.
- **Skills use trigger phrases** — `USE FOR` and `DO NOT USE FOR` in descriptions help Copilot decide when to load them.
- **Setup script is non-destructive** — It merges settings rather than replacing them, and creates backups.
- **Agents organized into core SDLC pipeline + supplementary** — 4 core agents (Software Engineer, Security & QA, Technical Writer, Technical Troubleshooter) + 4 supplementary domain-specific agents.