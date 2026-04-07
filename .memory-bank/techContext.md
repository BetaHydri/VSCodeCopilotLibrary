# Tech context

## Technology stack

| Layer | Technology | Purpose |
|---|---|---|
| IDE | VS Code | Primary development environment |
| AI assistant | GitHub Copilot (Claude Opus 4.6) | Code generation, review, documentation |
| Sync | OneDrive | Cross-machine file synchronization |
| Setup script | PowerShell 5.1+ | Automated VS Code configuration |
| Version control | Git | Repository management |

## VS Code settings configured

### File location settings

```jsonc
"chat.agentFilesLocations":        { "~/OneDrive/MyCopilot/Agents": true }
"chat.instructionsFilesLocations": { "~/OneDrive/MyCopilot/Instructions": true }
"chat.agentSkillsLocations":       { "~/OneDrive/MyCopilot/Skills": true }
"chat.promptFilesLocations":       { "~/OneDrive/MyCopilot/Prompts": true }
```

### Feature flags

| Setting | Value | Purpose |
|---|---|---|
| `chat.includeApplyingInstructions` | `true` | Auto-apply instruction files by glob match |
| `chat.includeReferencedInstructions` | `true` | Follow Markdown links in instruction files |
| `github.copilot.chat.agent.thinkingTool` | `true` | Enable agent reasoning tool |
| `github.copilot.chat.search.semanticTextResults` | `true` | Semantic search in agent mode |
| `github.copilot.chat.agent.maxRequests` | `500` | Maximum tool-call requests per agent turn (default: 5) |

### Model preferences

| Setting | Value |
|---|---|
| `gitlens.ai.vscode.model` | `copilot:claude-opus-4.6-fast` |
| `github.copilot.advanced.model` | `claude-opus-4.6-fast` |

## Agents inventory

### Core SDLC Pipeline Agents

| Agent name | ID | Role | Tools count | Handoffs |
|---|---|---|---|---|
| Software Engineer | `software-engineer` | Development & implementation | 21 | security-reviewer, technical-writer |
| Security & QA | `security-reviewer` | Security audits & production readiness | 17 | software-engineer |
| Technical Writer | `technical-writer` | Articles & documentation | 13 | security-reviewer |
| Technical Troubleshooter | `technical-troubleshooter` | Problem diagnosis & resolution | — | software-engineer |

### Supplementary Domain-Specific Agents

| Agent name | ID | Role |
|---|---|---|
| Legal Researcher (DE) | `legal-researcher` | German legal research & statement drafting |
| QC Inspector | `qc-inspector` | Quality control inspection for Oil & Gas / Energy / Industrial |
| Training Content Writer | `training-writer` | Generic training & workshop content creation |
| DevOps Training Writer | `devops-training-writer` | DevOps-specialized training (inherits from Training Content Writer) |

## Instructions inventory

| File | Applies to | Approximate size |
|---|---|---|
| `powershell.instructions.md` | `*.ps1, *.psm1, *.psd1` | ~1,233 lines |
| `markdown.instructions.md` | `*.md` | ~1,426 lines |
| `yaml.instructions.md` | `*.yml, *.yaml` | ~974 lines |
| `csharp.instructions.md` | `*.cs, *.csx` | ~1,260 lines |
| `changelog.instructions.md` | `CHANGELOG.md` and variants | ~1,137 lines |
| `versioning.instructions.md` | `GitVersion.yml, *.psd1, CHANGELOG.md` | ~1,228 lines |
| `sampler.instructions.md` | `build.yaml, build.ps1, RequiredModules.psd1, ...` | ~150 lines |
| `copilot-model-selection-instructions.md` | Copilot CLI model routing | ~120 lines |
| `pester.instructions.md` | `*.Tests.ps1, *.tests.ps1` | ~620 lines |
| `git.instructions.md` | `.gitconfig, .gitignore, .gitattributes, COMMIT_EDITMSG` | ~380 lines |
| `json.instructions.md` | `*.json, *.jsonc` | ~350 lines |
| `azurepipelines.instructions.md` | `azure-pipelines.yml, .azuredevops/*.yml` | ~500 lines |

## Skills inventory

| Skill | Directory | Purpose |
|---|---|---|
| `automatedlab-deployment` | `Skills/automatedlab-deployment/` | Build and deploy Hyper-V lab environments using AutomatedLab |
| `datum-configuration` | `Skills/datum-configuration/` | Comprehensive reference for Datum hierarchical DSC configuration data module |
| `dsc-troubleshooting` | `Skills/dsc-troubleshooting/` | Debug and troubleshoot PowerShell DSC resource failures on target nodes |
| `german-legal-research` | `Skills/german-legal-research/` | Legal research for German tenancy/rental law and civil law |
| `grammar-check` | `Skills/grammar-check/` | Identify grammar, logical, and flow errors in text |
| `outlook-email-export` | `Skills/outlook-email-export/` | Export and extract emails from Outlook via COM automation |
| `pester-patterns` | `Skills/pester-patterns/` | Ready-to-use Pester 5 test patterns and recipes |
| `sampler-build-debug` | `Skills/sampler-build-debug/` | Debug Sampler builds and Pester 5 test failures |
| `sampler-framework` | `Skills/sampler-framework/` | Comprehensive Sampler PowerShell module build framework reference |
| `sampler-migration` | `Skills/sampler-migration/` | Migrate legacy PowerShell modules to Sampler framework |
| `mecm-dsc-deployment` | `Skills/mecm-dsc-deployment/` | Deploy and troubleshoot MECM/SCCM via DSC using ConfigMgrCBDsc in DscWorkshop/Datum environments |
| `pandoc-docx-export` | `Skills/pandoc-docx-export/` | Export Markdown to DOCX via pandoc with Lua filters, landscape tables, custom column widths |
| `send-outlook-email` | `Skills/send-outlook-email/` | Send emails via the Outlook COM API from PowerShell |
| `winrm-troubleshooting` | `Skills/winrm-troubleshooting/` | Debug WinRM connectivity failures, service recovery, auth issues, PowerShell Direct fallback |
| `outlook-calendar-export` | `Skills/outlook-calendar-export/` | Export Outlook calendar entries to Markdown via COM automation with recurring appointment support |

## Prompts inventory

| Prompt | File | Purpose |
|---|---|---|
| Code Review | `Prompts/CodeReview.prompt.md` | PowerShell security code review with SARIF + Markdown output |
| Lab Deploy | `Prompts/LabDeploy.prompt.md` | Deploy AutomatedLab environments via natural language |
| PR Description | `Prompts/PRDescription.prompt.md` | Generate PR descriptions from branch diff |
| Module Scaffold | `Prompts/ModuleScaffold.prompt.md` | Scaffold new Sampler-based PowerShell module projects |
| Refactor | `Prompts/Refactor.prompt.md` | Structured multi-phase code refactoring workflow |

## Development setup

1. Sign into OneDrive so `~/OneDrive/MyCopilot/` syncs
2. Run `Setup-CopilotSettings.ps1` in PowerShell
3. Restart VS Code
4. Verify via Copilot Chat diagnostics (right-click → Diagnostics)

## Dependencies

- **VS Code** with GitHub Copilot extension
- **OneDrive** signed in and syncing
- **PowerShell 5.1+** for setup script
- **Git** for version control
