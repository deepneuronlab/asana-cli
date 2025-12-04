# Asana CLI

[![Build Status](https://travis-ci.org/thash/asana.svg?branch=master)](https://travis-ci.org/thash/asana)

[Asana](https://asana.com/) command line client implemented in Go.

## About This Fork

This is an enhanced fork of [thash/asana](https://github.com/thash/asana) maintained by Deep Neuron Lab. We've added several features and fixes to improve the CLI experience:

### What's New

- **✅ Asana API v1.0 Compatibility**: Fixed support for the new `gid` (Global ID) string format
- **✅ Custom Fields Display**: View task metadata like Urgency, Dev Severity, Environment, etc.
- **✅ JSON Output Format**: Machine-readable output with `--json` flag for automation and scripting
- **✅ Attachment Support**: View and download file attachments from tasks

See the [Changes](#changes-from-upstream) section below for detailed information.

**Upstream PRs**: We've opened pull requests ([#30](https://github.com/thash/asana/pull/30), [#31](https://github.com/thash/asana/pull/31), [#32](https://github.com/thash/asana/pull/32), [#33](https://github.com/thash/asana/pull/33)) to contribute these improvements back to the original project.

## Installation

### Requirements

- Go 1.16 or higher

### From Source

```bash
go get github.com/deepneuronlab/asana-cli
```

### Mac OS X (Original Repository)

```bash
brew tap thash/asana
brew install asana
```

## Usage

```bash
asana help
```

### Available Commands

```
COMMANDS:
   config, c      Asana configuration. Your settings will be saved in ~/.asana.yml
   workspaces, w  get workspaces
   tasks, ts      get tasks
   task, t        get a task
   comment, cm    Post comment
   done           Complete task
   due            set due date
   browse, b      open a task in the web browser
   download, dl   download attachment from a task
   help, h        Shows a list of commands or help for one command
```

## Configuration

First, configure the CLI with your Asana Personal Access Token:

```bash
asana config
```

Visit: https://app.asana.com/-/account_api
- Go to Settings > Apps > Manage Developer Apps > Personal Access Tokens
- Create New Personal Access Token
- Copy and paste the token when prompted

Your configuration will be saved in `~/.asana.yml`:

```yaml
personal_access_token: 0/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
workspace: 4444444444444
```

## Working with Tasks

### List Tasks

```bash
asana tasks
# or short form
asana ts
```

Output:
```
 0 [ 2014-08-13 ] Write README.
 1 [ 2014-08-18 ] Buy gift for coworkers.
 2 [ 2014-08-29 ] Read "Unweaving the Rainbow".
 3 [            ] haircut
```

### View Task Details

View a specific task by its index:

```bash
asana task 0
# or short form
asana t 0
```

#### With Custom Fields ✨ NEW

Custom fields are automatically displayed:

```bash
asana task 64
```

Output includes:
```
[ 2025-12-04 ] [Customer-Reported] KI Vorschläge

  Custom Fields:
    Urgency: Very High
    Dev Severity: High
    Environment: Production
    Ticket Type: Bug
    Dev Team: Backend, Customer Support

[Task notes and description...]
```

#### Verbose Mode (with Stories/Comments)

```bash
asana task -v 0
```

Shows the task with all comments and activity history.

#### JSON Output ✨ NEW

Get machine-readable output for automation:

```bash
# Basic JSON output
asana task --json 64
asana task -j 64

# JSON with stories/comments
asana task -j -v 64
```

Example JSON structure:
```json
{
  "task": {
    "gid": "1212293030279497",
    "name": "[Customer-Reported] KI Vorschläge",
    "custom_fields": [
      {
        "gid": "1212278811468834",
        "name": "Urgency",
        "display_value": "Very High",
        "type": "enum"
      }
    ],
    "notes": "...",
    "assignee": {...}
  },
  "stories": [...],
  "attachments": [...]
}
```

#### Process with jq

```bash
# Extract urgency field
asana task -j 64 | jq '.task.custom_fields[] | select(.name=="Urgency")'

# Get all attachment names
asana task -j 64 | jq '.attachments[].name'
```

### Working with Attachments ✨ NEW

#### View Attachments

Attachments are listed when viewing a task:

```bash
asana task 64
```

Output includes:
```
  Attachments (1):
    [0] Screenshot.png (GID: 1212292979508802)
```

#### Download Attachments

Download by task index and attachment index:

```bash
# Download first attachment from task 64
asana download 64 0

# Download with custom output path
asana download -o /tmp/screenshot.png 64 0
asana download --output /tmp/screenshot.png 64 0
```

Download directly by attachment GID:

```bash
asana download 1212292979508802
asana download -o myfile.png 1212292979508802
```

### Task Management

#### Complete a Task

```bash
asana done 12
```

#### Set Due Date

```bash
# Specific date
asana due 5 2014-08-21

# Relative dates
asana due 5 today
asana due 5 tomorrow
```

#### Add Comments

```bash
asana comment 2
# or short form
asana cm 2
```

This opens your `$EDITOR` to write a comment. Save and close to post.

#### Open in Browser

```bash
asana browse 1
# or short form
asana b 1
```

## Changes from Upstream

This fork includes the following enhancements over the original [thash/asana](https://github.com/thash/asana):

### 1. Asana API v1.0 Compatibility Fix

**Problem**: Asana's API changed from integer `id` fields to string `gid` (Global ID) fields, causing 404 errors.

**Solution**:
- Added `Gid` string field to structs with backward compatibility for `Id`
- Added `GetTaskId()` method that prefers `Gid` over `Id`
- Updated cache to store string GIDs
- Added proper JSON tags for all API fields

**Status**: [Pull Request #30](https://github.com/thash/asana/pull/30) opened upstream

### 2. Custom Fields Display

**Feature**: Automatically display Asana custom fields when viewing tasks.

**Details**:
- Shows fields like Urgency, Dev Severity, Environment, Ticket Type, etc.
- Only displays fields with values (hides empty fields)
- Works with all custom field types (enum, multi_enum, text, number, date)
- No additional flags or API calls needed

**Status**: [Pull Request #31](https://github.com/thash/asana/pull/31) opened upstream

### 3. JSON Output Format

**Feature**: Added `--json` / `-j` flag for machine-readable output.

**Use Cases**:
- Automation and scripting
- Integration with other tools
- Data processing with jq, Python, etc.

**Details**:
- Outputs complete task data in JSON format
- Includes custom fields, attachments, assignee, dates, etc.
- Works with `--verbose` flag to include stories/comments
- Fully backward compatible (default text output unchanged)

**Status**: [Pull Request #32](https://github.com/thash/asana/pull/32) opened upstream

### 4. Attachment Support

**Feature**: View and download file attachments from tasks.

**Capabilities**:
- List attachments with indices when viewing tasks
- Download by task index + attachment index
- Download directly by attachment GID
- Custom output paths with `--output` / `-o` flag
- Automatic handling of temporary S3 download URLs
- Included in JSON output for automation

**Status**: [Pull Request #33](https://github.com/thash/asana/pull/33) opened upstream

## Development

### Build from Source

```bash
git clone https://github.com/deepneuronlab/asana-cli.git
cd asana-cli
go build
./asana help
```

### Run Tests

```bash
go test ./...
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

We maintain compatibility with the upstream repository and contribute improvements back when appropriate.

## License

This project maintains the same license as the upstream [thash/asana](https://github.com/thash/asana) repository.

## Credits

- Original project by [thash](https://github.com/thash)
- Enhancements and maintenance by [Deep Neuron Lab](https://github.com/deepneuronlab)
- Built with [urfave/cli](https://github.com/urfave/cli)

## Links

- **This Fork**: https://github.com/deepneuronlab/asana-cli
- **Upstream**: https://github.com/thash/asana
- **Upstream PRs**: [#30](https://github.com/thash/asana/pull/30), [#31](https://github.com/thash/asana/pull/31), [#32](https://github.com/thash/asana/pull/32), [#33](https://github.com/thash/asana/pull/33)
- **Asana API Documentation**: https://developers.asana.com/docs
