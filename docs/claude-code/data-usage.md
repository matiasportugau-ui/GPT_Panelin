# Data usage

> Learn about Anthropic's data usage policies for Claude Code.

## Data policies

### Data training policy

**Consumer users (Free, Pro, and Max plans)**:
You have the choice to allow your data to be used to improve future Claude models. Training will use data from Free, Pro, and Max accounts when this setting is on (including when you use Claude Code from these accounts).

**Commercial users** (Team and Enterprise plans, API, 3rd-party platforms, and Claude Gov): Anthropic does not train generative models using code or prompts sent to Claude Code under commercial terms, unless the customer has chosen to provide their data for model improvement (for example, the [Development Partner Program](https://support.claude.com/en/articles/11174108-about-the-development-partner-program)).

### Data retention

Anthropic retains Claude Code data based on your account type and preferences.

**Consumer users (Free, Pro, and Max plans)**:

* Users who allow data use for model improvement: 5-year retention period
* Users who don't allow data use for model improvement: 30-day retention period
* Privacy settings can be changed at any time at [claude.ai/settings/data-privacy-controls](https://claude.ai/settings/data-privacy-controls)

**Commercial users (Team, Enterprise, and API)**:

* Standard: 30-day retention period
* Zero data retention: Available with appropriately configured API keys
* Local caching: Claude Code clients may store sessions locally for up to 30 days to enable session resumption (configurable)

Learn more about data retention practices in the [Privacy Center](https://privacy.anthropic.com/).

## Data access

For all first party users, data is logged for both local Claude Code and remote Claude Code sessions. For remote Claude Code, Claude accesses the repository where you initiate your Claude Code session. Claude does not access repositories that you have connected but have not started a session in.

## Local Claude Code: Data flow and dependencies

Claude Code is installed from [NPM](https://www.npmjs.com/package/@anthropic-ai/claude-code). Claude Code runs locally. In order to interact with the LLM, Claude Code sends data over the network. This data includes all user prompts and model outputs. The data is encrypted in transit via TLS and is not encrypted at rest.

Claude Code is built on Anthropic's APIs. For details regarding the API's security controls, including API logging procedures, please refer to compliance artifacts offered in the [Anthropic Trust Center](https://trust.anthropic.com).

### Cloud execution: Data flow and dependencies

When using [Claude Code on the web](claude-code-on-the-web.md), sessions run in Anthropic-managed virtual machines instead of locally. In cloud environments:

* **Code and data storage**: Your repository is cloned to an isolated VM. Code and session data are subject to the retention and usage policies for your account type
* **Credentials**: GitHub authentication is handled through a secure proxy; your GitHub credentials never enter the sandbox
* **Network traffic**: All outbound traffic goes through a security proxy for audit logging and abuse prevention
* **Session data**: Prompts, code changes, and outputs follow the same data policies as local Claude Code usage

## Telemetry services

Claude Code connects from users' machines to the Statsig service to log operational metrics such as latency, reliability, and usage patterns. This logging does not include any code or file paths. To opt out, set the `DISABLE_TELEMETRY` environment variable.

Claude Code connects from users' machines to Sentry for operational error logging. To opt out, set the `DISABLE_ERROR_REPORTING` environment variable.

When users run the `/bug` command, a copy of their full conversation history including code is sent to Anthropic.

## Default behaviors by API provider

| Service | Claude API | Vertex API | Bedrock API |
|---------|-----------|------------|-------------|
| **Statsig (Metrics)** | Default on. `DISABLE_TELEMETRY=1` to disable. | Default off. | Default off. |
| **Sentry (Errors)** | Default on. `DISABLE_ERROR_REPORTING=1` to disable. | Default off. | Default off. |
| **Claude API (`/bug` reports)** | Default on. `DISABLE_BUG_COMMAND=1` to disable. | Default off. | Default off. |

All environment variables can be checked into `settings.json` (see [Settings](settings.md)).

## Related resources

* [Claude Code on the web](claude-code-on-the-web.md)
* [Settings reference](settings.md)
* [Security](security.md)
* [Anthropic Trust Center](https://trust.anthropic.com)
* [Privacy Center](https://privacy.anthropic.com/)
