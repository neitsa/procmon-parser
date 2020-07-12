# Parsing Procmon files with Python

[![Build Status](https://travis-ci.com/eronnen/procmon-parser.svg?branch=master)](https://travis-ci.com/eronnen/procmon-parser)

Procmon (https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) is a very powerful monitoring tool for Windows,
capable of capturing file system, registry, process/thread and network activity. 

Procmon uses internal file formats for configuration (**PMC**) and logs (**PML**). Prior to ``procmon-parser``, **PMC** files could only be parsed and generated by the Procmon GUI, 
and **PML** files could be read only using the Procmon GUI, or converting them to CSV or XML using Procmon command line.

The goals of `procmon-parser` are:
* Parsing & Building **PMC** files - making it possible to dynamically add/remove filter rules, which can significantly
reduce the size of the log file over time as Procmon captures millions of events.
* Parsing **PML** files (limited support) - making it possible to directly load the raw **PML** file into convenient python objects
instead of having to convert the file to CSV/XML formats prior to loading.
 

### Supported Procmon versions
3.4.\*, 3.5.\*


## PMC (Process Monitor Configuration) Parser

### Usage

Loading configuration of a pre-exported Procmon configuration:
```python
>>> from procmon_parser import load_configuration, dump_configuration, Rule
>>> with open("ProcmonConfiguration.pmc", "rb") as f:
...     config = load_configuration(f.read())
>>> config["DestructiveFilter"]
0
>>> config["FilterRules"]
[Rule(Column.PROCESS_NAME, RuleRelation.IS, "System", RuleAction.EXCLUDE), Rule(Column.PROCESS_NAME, RuleRelation.IS, "Procmon64.exe", RuleAction.EXCLUDE), Rule(Column.PROCESS_NAME, RuleRelation.IS, "Procmon.exe", RuleAction.EXCLUDE), Rule(Column.PROCESS_NAME, RuleRelation.IS, "Procexp64.exe", RuleAction.EXCLUDE), Rule(Column.PROCESS_NAME, RuleRelation.IS, "Procexp.exe", RuleAction.EXCLUDE), Rule(Column.PROCESS_NAME, RuleRelation.IS, "Autoruns.exe", RuleAction.EXCLUDE), Rule(Column.OPERATION, RuleRelation.BEGINS_WITH, "IRP_MJ_", RuleAction.EXCLUDE), Rule(Column.OPERATION, RuleRelation.BEGINS_WITH, "FASTIO_", RuleAction.EXCLUDE), Rule(Column.RESULT, RuleRelation.BEGINS_WITH, "FAST IO", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "pagefile.sys", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$Volume", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$UpCase", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$Secure", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$Root", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$MftMirr", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$Mft", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$LogFile", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.CONTAINS, "$Extend", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$Boot", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$Bitmap", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$BadClus", RuleAction.EXCLUDE), Rule(Column.PATH, RuleRelation.ENDS_WITH, "$AttrDef", RuleAction.EXCLUDE), Rule(Column.EVENT_CLASS, RuleRelation.IS, "Profiling", RuleAction.EXCLUDE)]
```

Adding some new rules
```python
>>> new_rules = [Rule('Path', 'contains', '1337', 'include'), Rule('Process_Name', 'is', 'python.exe')]
>>> config["FilterRules"] = new_rules + config["FilterRules"]
```

Dropping filtered events
```python
>>> config["DestructiveFilter"] = 1
```

Dumping the new configuration to a file
```python
>>> with open("ProcmonConfiguration1337.pmc", "wb") as f:
...     dump_configuration(config, f)
```

### File Format

A PMC file is the configuration file for Procmon, which can be exported using the graphical interface, and later
imported in another Procmon instance. This file contains a sequence of records, where every record has a name and a
and a value. The known record options are:

* `Columns` - a list of the width of the GUI columns in pixels.
* `ColumnCount` - the number of columns to show.
* `ColumnMap` - an ordered list of the column types to show.
* `DbgHelpPath`
* `Logfile` - an optional path to a PML file to store the captured events.
* `HighlightFG`   
* `HighlightBG`
* `LogFont`
* `BoookmarkFont`  (they have a typo...)
* `AdvancedMode`
* `Autoscroll`
* `HistoryDepth`
* `Profiling`
* `DestructiveFilter` - whether to drop events that the current filters exclude.   
* `AlwaysOnTop`
* `ResolveAddresses`
* `SourcePath`
* `SymbolPath`
* `FilterRules` - a list of filter rules that can be used to show only interesting events.
* `HighlightRules`

For the raw binary format of PMC files you can refer to [configuration_format.py](procmon_parser/configuration_format.py).

## PML (Process Monitor Log) Parser

### Usage

`procmon-parser` exports a `ProcmonLogsReader` class for reading logs directly from a PML file:
```python
>>> from procmon_parser import ProcmonLogsReader
>>> f = open("LogFile.PML", "rb")
>>> pml_reader = ProcmonLogsReader(f)
>>> first_log = next(pml_reader)  # reading the next log entry
>>> first_log
Event(dwm.exe, 932, 1568, "Registry", "RegQueryValue", 7400, 2020-07-12T01:18:10.775242900, 0, "", "HKCU\Software\Microsoft\Windows\DWM\ColorPrevalence", {})
>>> len(pml_reader)  # number of logs
53214
>>>
```

### File Format

For the raw binary format of PML files you can refer to [logs_format.py](procmon_parser/logs_format.py) and [logs_details_format.py](procmon_parser/logs_details_format.py).

### Unsupported features

The PML format is very complex so there are some features that are not supported yet:
- [ ] Reading logs from multiple related PML files.
- [ ] Detail column is not fully supported for all operation types.

Pull requests to support these missing features are very welcome :)

## Contributing

**`procmon-parser`** is developed on GitHub at [eronnen/procmon-parser](https://github.com/eronnen/procmon-parser).
Feel free to report an issue or send a pull request, use the
[issue tracker](https://github.com/eronnen/procmon-parser/issues).
