# elevate.metabase.tools
Tools for managing [Metabase](https://www.metabase.com/) .

[Docker images on docker hub](https://hub.docker.com/r/elevate/elevate.metabase.tools/).

# Metabase-exporter

Used to export/import the current state of Metabase (questions/dashboards/collections) to/from a JSON file. 
This file can then be stored on git for versioning.

## Limitations

This tool currently exports/imports collections, dashboards and SQL-defined questions only. 
Other kinds of questions are not supported because they are hard to make portable across Metabase instances.
Other features (e.g. pulses) are not supported simply because I don't need them :)

## Usage

Parameters can be given via command-line, appsettings.json or environment variables.
There are two modes of operation, given by the `Command` parameter: `import` or `export`.

Regardless of the operation, the Metabase API settings must be configured. E.g. in appsettings.json:

```json
{
  "MetabaseApi": {
    "Url": "https://metabase-local.elevatedirect.com:32443",
    "Username": "mauricio@elevatedirect.com",
    "Password":  "123456789"
  }
}
```

### Export

This exports the state to a file. Sample usage:

`metabase-exporter.exe Command=export OutputFilename=metabase-state.json`

You can optionally exclude personal collections by setting `ExcludePersonalCollections=1`
Also you can optionally select the exact collection to be exported with setting `CollectionIDToExport`. It exports a collection and all SQL cards and dashboards inside.

### Import

This imports a state file into a Metabase instance. Sample usage:

`metabase-exporter.exe Command=import InputFilename=metabase-state.json DatabaseMapping:1=2 DatabaseMapping:2=3`

The `DatabaseMapping` settings map Metabase database IDs in the state file to database IDs in the target Metabase instance.
In the example above, it maps:
* the database ID 1 in the file to the database ID 2 in the target Metabase instance
* the database ID 2 in the file to the database ID 3 in the target Metabase instance

### Usage with Docker

Pull the image or build it with `docker build`.

#### Run export

Run export command with defined volumes mappings for:
* passing `appsettings.json` into a container
* fetching the output file to your local machine

Usage example given `appsettings_export.json` with Metabase API settings is defined in your current directory:
```
docker run -v ./appsettings_export.json:/app/appsettings.json -v ./export_output:/app/export_output metabase-export-tool:latest Command=export OutputFilename='export_output/metabase-state.json'
```

#### Run import

Usage example given `appsettings_import.json` with import target Metabase API settings is defined in your current directory:
```
docker run -v ./export_output/metabase-state.json:/app/metabase-state.json -v ./appsettings_import.json:/app/appsettings.json metabase-export-tool:latest Command=import InputFilename=metabase-state.json DatabaseMapping:2=3
```
