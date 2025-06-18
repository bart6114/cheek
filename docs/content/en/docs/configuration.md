---
title: Configuration
---

Everything about how you want the scheduler to function is defined in a schedule specification written in YAML.

## Basic Configuration

Create a schedule specification using the example below:

```yaml
tz_location: Europe/Brussels # optionally set timezone to adhere to
jobs:
  foo:
    command: date
    cron: "* * * * *" # a cron string to specify when to run
    on_success:
      trigger_job: # trigger something on run
        - bar
  bar:
    command: # command to run, use a list if you want to pass args
      - echo
      - $foo
    env: # you can pass env variables
      foo: bar
    encoding: utf-8 # specify output encoding if job produces non-UTF-8 text
  other_workingdir:
    command: pwd
    working_directory: ../testdata # specify the working directory of the job
  coffee:
    command: this fails
    cron: "* * * * *"
    retries: 3
    disable_concurrent_execution: true # prevent concurrent runs of this job (defaults to false)
    on_error:
      notify_webhook: # notify something on error
        - https://webhook.site/4b732eb4-ba10-4a84-8f6b-30167b2f2762
      notify_slack_webhook: # notify slack via a slack compatible webhook
        - https://webhook.site/048ff47f-9ef5-43fb-9375-a795a8c5cbf5
      notify_discord_webhook: # notify discord via a discord compatible webhook
        - https://discord.com/api/webhooks/user/token
  legacy_system:
    command: legacy-app.exe
    encoding: windows-1252 # handle Windows-specific encoding for legacy applications
```

## Configuration Options

All configuration options are available by checking out `cheek --help` or the help of its subcommands (e.g. `cheek run --help`).

Configuration can be passed as flags to the `cheek` CLI directly. All configuration flags are also possible to set via environment variables. The following environment variables are available, they will override the default and/or set value of their similarly named CLI flags (without the prefix): `CHEEK_PORT`, `CHEEK_SUPPRESSLOGS`, `CHEEK_LOGLEVEL`, `CHEEK_PRETTY`, `CHEEK_HOMEDIR`.

## Output Encoding

By default, cheek expects job output to be UTF-8 encoded. If your jobs produce output in a different encoding, unsupported characters may display as fallback Unicode "tofu" characters (□) or become corrupted. This problem most commonly occurs on Windows systems, legacy applications, or when working with applications that output text in regional character encodings such as Chinese, Japanese, or Korean.

To handle non-UTF-8 output correctly, you can specify the `encoding` parameter at the job level to tell cheek how to properly decode the job's output.

### Supported Encodings

cheek supports the following output encodings:

**UTF-8 Compatible** (no transformation needed):
- `utf-8`, `utf8` - UTF-8 Unicode encoding (default)
- `ascii` - ASCII encoding

**Chinese Encodings**:
- `gbk`, `gb2312`, `cp936` - Simplified Chinese (GBK)
- `gb18030` - Extended Chinese encoding standard
- `big5` - Traditional Chinese

**Japanese Encodings**:
- `shift-jis`, `shiftjis`, `sjis` - Shift JIS encoding
- `euc-jp` - Extended Unix Code for Japanese

**Korean Encodings**:
- `euc-kr` - Extended Unix Code for Korean

**Western Encodings**:
- `iso-8859-1`, `latin1` - Western European (Latin-1)
- `windows-1252`, `cp1252` - Windows Western European

## Important Notes

- If your `command` requires arguments, please make sure to pass them as an array like in the `bar` job example above
- You can set `tz_location` if the system time of where you run your service is not to your liking
- The `encoding` parameter is specified per job, allowing different jobs in the same schedule to use different encodings
- If you see garbled characters or □ symbols in job output, consider setting the appropriate `encoding` parameter
- The configuration structure should be self-explanatory, but if it's not, please create an [issue](https://github.com/bart6114/cheek/issues)

## Running cheek

Once you have your schedule configuration ready, you can start the scheduler with:

```bash
cheek run ./path/to/my-schedule.yaml
```

Check out `cheek run --help` for additional configuration options.