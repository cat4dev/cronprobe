# CronProbe

> A bash wrapper that intercepts command execution and alerts Slack, Discord, Ntfy, or custom webhooks on failure with the exact error output.

### What is it?
`cronprobe` is a single-file script that wraps your existing cron jobs. It runs your command, intercepts the exit code, and based on the result, either discards the output or sends an alert.

**Important standard cron behavior changes:** 
Normally, the cron daemon emails you any stdout or stderr produced by a running job. `cronprobe` **swallows all stdout and stderr** while executing. 
- If your job finishes successfully (exit code `0`), CronProbe silently discards the output and exits.
- If your job fails (non-zero exit code), CronProbe captures the output, sends it to your Slack/Discord/Ntfy webhook, **and** prints it to the console so native cron failure emails still work.

### Who is it for?
Operators running bash scripts, DB backups, or Python scrapers on Virtual Private Servers (VPS) who need immediate alerts and log context for script crashes. It is strictly a wrapper and does *not* provide root-cause analysis beyond the captured text.

### Security & Safety Warnings ⚠️
* **Direct Webhook Routing:** Error data is routed directly from your server to your Slack/Discord/Ntfy channel via `curl`. There is no middleman service. The temporary file used to buffer local output is removed from your server after execution.
* **Secret Leakage:** On failure, CronProbe captures and transmits both your **command arguments** and the last 50 lines of **stdout/stderr output**. **Do not pass passwords, API keys, or database credentials as plaintext command arguments.** Also, ensure your application logic does not print secrets to standard output or error upon crashing. Use environment variables instead.
* **Webhook Timeouts:** If the webhook endpoint is unreachable or times out, CronProbe stops cleanly. It preserves the original script's exit code and will not hang the parent process.

### Limitations
* `cronprobe` is designed for commands that output plain text.
* Slack limits single message payloads. `cronprobe` truncates log output to the final ~50 lines (2,000 characters). It will not capture the entirety of large log streams.
* **It does not monitor missed runs.** `cronprobe` is not a heartbeat monitor. If your server powers off or the cron daemon halts, the wrapper will not execute, and you will not receive a failure alert.

### Installation
Download the script to your target server and make it executable:
```bash
sudo curl -L -o /usr/local/bin/cronprobe https://raw.githubusercontent.com/cat4dev/cronprobe/main/cronprobe
sudo chmod +x /usr/local/bin/cronprobe
```

### Usage 

Prefix your standard cron entries with `cronprobe` and pass your webhook URL.

**Before:**
```crontab
0 4 * * * /usr/bin/python3 /opt/scrapers/sync.py
```

**After:**
```crontab
0 4 * * * cronprobe --webhook="https://hooks.slack.com/services/..." -- /usr/bin/python3 /opt/scrapers/sync.py
```

*(Optionally, you can set the `CRONPROBE_WEBHOOK` environment variable in your crontab instead of passing the CLI argument).*

### Supported Webhooks
`cronprobe` attempts to route the payload format automatically based on your URL:
- **Slack:** URLs containing `hooks.slack.com`
- **Discord:** URLs containing `discord.com`
- **Ntfy:** URLs containing `ntfy.sh` (sent as plain text)
- **Generic:** Any other URL receives a standardized JSON payload: `{"status": "failed", "host": "...", "command": "...", "exit_code": 1, "duration_seconds": 5, "output": "..."}`. Use this to route to Zapier, Make.com, n8n, or your own custom backend.

*You can explicitly override the detection using `--slack`, `--discord`, `--ntfy`, or `--generic`.*

### Future Scope
CronProbe is built as a utility to solve one specific problem: capturing the output of crashed cron jobs without altering existing infrastructure. There is no intent to expand this script into a platform with a web dashboard, user accounts, or active missed-job heartbeat detection.
