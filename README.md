# Roseingrave

Massively scalable musical source comparator.

## Dependencies

These scripts interact with Google Sheets through the
[`gspread` package](https://docs.gspread.org/en/latest/).
Currently, the script only supports using a Google service account with which
Spreadsheets may be created, accessed, and edited. See
[here](https://docs.gspread.org/en/latest/oauth2.html#for-bots-using-service-account)
for steps on how to create and use this service account.

The credentials are expected to be in the file defined by `"credentials"` in the
[settings](#settings).

## Settings

The `roseingrave.json` file defines alternative names for the input and output
files for the commands. The default configuration is:

```json
{
  "credentials": "service_account.json",
  "definitionFiles": {
    "template": ["input", "template_definitions.json"],
    "pieces": ["input", "piece_definitions.json"],
    "volunteers": ["input", "volunteer_definitions.json"]
  },
  "outputs": {
    "spreadsheetsIndex": ["output", "spreadsheets.json"],
    "pieceSummary": ["output", "summary.json"],
    "pieceDataPath": ["output", "data", "by-piece", "{piece}.json"],
    "volunteerDataPath": ["output", "data", "by-volunteer", "{email}.json"]
  }
}
```

Each value can either be a string for the filename, or an array defining the
path to the file.

For `"pieceDataPath"` and `"volunteerDataPath"`, you may use `"{piece}"` and
`"{email}"` respectively in the path to format the name of the piece and the
email of the volunteer respectively.

In the following, file names/paths will be referenced by its corresponding key.

## Input files

### `"template"`

The `"template"` file defines the names of rows or columns and other values to
use for created spreadsheets. It has the following format with default values:

```json
{
  "owner": "REQUIRED",
  "metaDataFields": {
    "title": "Title",
    "tempo": "Tempo",
    "key": "Key",
    "keySig": "Key sig.",
    "timeSig": "Time sig.",
    "barCount": "Bars",
    "compass": "Compass",
    "comments": "Comments",
    "notes": "Notes",
    "clefs": "Clefs (if other than G and F)",
    "endOrRepeat": "Endings and Repeat signs",
    "articulation": "Articulation signs",
    "dynamic": "Dynamic signs",
    "hand": "Hand signs",
    "otherIndications": "Other indications"
  },
  "values": {
    "defaultBarCount": 100
  }
}
```

The `"owner"` field is required and should be the email of the person to give
ownership of each created spreadsheet. (Note: As of April 2022, transferring
ownership requires consent. Thus, this email will be made an "editor" until
there is a workaround for this issue.)

Each field under `"metaDataFields"` defines the header name of each row above
the bars section, with the exception of the two following special fields:

- `"columns"`: The right-most column, where comments can be left on any of the
  rows or bars.
- `"notes"`: A single row below the bars section, where source-specific notes
  may be left.

Each field under `"values"` has the following meaning:

- `"defaultBarCount"`: If no bar counts are given in `"pieces"`, use this value.

In the future, there will be additional fields for customizing font, font size,
font weight, etc.

### `"pieces"`

The `"pieces"` file defines each piece and the sources for each piece. Each
piece can have an optional link. Each source requires a name and a link and also
has an optional bar count. The resulting bar section for this piece will be the
max of all the bar counts given, or a default if no bar counts are given.

The file should have the following format:

```json
[
  {
    "title": "pieceName1",
    "sources": [
      {
        "name": "sourceName1",
        "link": "sourceLink1",
        "barCount": 100
      }
    ]
  },
  {
    "title": "pieceName2",
    "link": "pieceLink2",
    "sources": [
      {
        "name": "sourceName2",
        "link": "sourceLink2"
      }
    ]
  }
]
```

Pieces with repeated names will be treated as a single piece with the
combination of all their sources. All sources will be saved, regardless of
repeated names or links.

### `"volunteers"`

The `"volunteers"` file defines each volunteer and the pieces for each
volunteer.

The file should have the following format:

<!-- prettier-ignore -->
```json
[
  {
    "email": "volunteerEmail",
    "pieces": [
      "pieceName1",
      "pieceName2"
    ]
  }
]
```

Volunteers with repeated emails will be treated as a single volunteer with the
combination of all their pieces. Repeated pieces will be ignored after the first
occurrence. Unknown pieces will be ignored. Spreadsheets will be created with
the order of the pieces preserved.

## Output files

### `"spreadsheetsIndex"`

The `"spreadsheetsIndex"` file defines a mapping from volunteer emails to their
corresponding spreadsheet link. It will also have a key of `"MASTER"` for the
master spreadsheet.

### `"pieceSummary"`

TODO

### `"pieceDataPath"`

TODO

### `"volunteerDataPath"`

TODO

## Commands

Run with `python -m roseingrave <command> [options]`.

More commands to come.

### `create_sheet`

Create volunteer spreadsheets.

Requires `"template"`, `"pieces"`, and `"volunteers"`. Outputs created
spreadsheet links to `"spreadsheetsIndex"`.

If any volunteers already exist in `"spreadsheetsIndex"`, they will be skipped.

#### Arguments

- `emails` (optional, variadic): The volunteers to create spreadsheets for.
  If none given, creates spreadsheets for all volunteers found in
  `"volunteers"`.

#### Options

- `-r`/`--replace` (flag): Replace existing volunteer spreadsheets.
  This will not create a new spreadsheet, but will wipe all current content in
  the existing spreadsheet.
- `-n`/`--new` (flag): Create new spreadsheets for all volunteers.
- `-td`: A filepath to replace `"template"`.
- `-pd`: A filepath to replace `"pieces"`.
- `-vd`: A filepath to replace `"volunteers"`.
- `-si`: A filepath to replace `"spreadsheetsIndex"`.
- `--strict` (flag): Fail on warnings instead of only displaying them.

<!-- TODO: below -->

<!--
### `volunteer_summary [EMAIL]`

- creates a volunteer JSON data file for a given volunteer email
  - if no email provided, creates JSONs for all volunteers
- requires `spreadsheets.json` to find the spreadsheet link
  - error if not found
  - maybe use a flag to override the name, like `-s spreadsheets.json`
- outputs `data/by-volunteers/<email>.json`
  - use `data/by-volunteers` as a default output folder and use `-o other_folder` as a way to override
  - see Pathlib to make paths: https://stackoverflow.com/a/50110841/408734

### `piece_summary [PIECE]`

- creates a piece JSON file for a given piece
  - if no piece provided, creates JSONs for all pieces found​
- reads the existing files in the `data/by-pieces/` subdirectory and compiles info from them
  - for accurate summary, run `volunteer_summary` first
  - how to specify if the path name has been changed?
  - does adding options to rename the paths make things too complicated? it's just inelegant to hard-code everything IMHO? thoughts?
- outputs `data/by-pieces/<piece>.json`
  - same remark as for `volunteer_summary` re: output folder flag

### `compile_pieces`

- compiles all piece JSON files into a single file for importing to the master spreadsheet
- reads the existing files in the `data/by-pieces/` subdirectory
  - for accurate summary, run `piece_summary` first
- outputs `summary.json`
  - the format for this file will be a little different from `<piece>.json`, for ease of importing/exporting from the master spreadsheet
  - for example, will include a "summary" field (defaults to `""`) for each source

### `import_master`

- updates the master spreadsheet, or creates it if it doesn't exist in `spreadsheets.json`
- requires `summary.json` and `template_definition.json`
  - for accurate sheet, run `compile_pieces` first
  - this could be issued as a warning with loguru to inform the user
- if created the sheet, updates `spreadsheets.json` with a "MASTER" key and the link

### `export_master`

- exports the master spreadsheet to a JSON file
- requires `spreadsheets.json` (for the spreadsheet link)
  - or `-s other_spreadsheets.json`? is this too bulky?
- outputs/replaces `summary.json` (same as `compile_pieces`)
-->
