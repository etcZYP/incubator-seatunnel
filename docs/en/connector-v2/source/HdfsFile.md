# HdfsFile

> Hdfs file source connector

## Description

Read data from hdfs file system.

:::tip

If you use spark/flink, In order to use this connector, You must ensure your spark/flink cluster already integrated hadoop. The tested hadoop version is 2.x.

If you use SeaTunnel Engine, It automatically integrated the hadoop jar when you download and install SeaTunnel Engine. You can check the jar package under ${SEATUNNEL_HOME}/lib to confirm this.

:::

## Key features

- [x] [batch](../../concept/connector-v2-features.md)
- [ ] [stream](../../concept/connector-v2-features.md)
- [x] [exactly-once](../../concept/connector-v2-features.md)

Read all the data in a split in a pollNext call. What splits are read will be saved in snapshot.

- [x] [column projection](../../concept/connector-v2-features.md)
- [x] [parallelism](../../concept/connector-v2-features.md)
- [ ] [support user-defined split](../../concept/connector-v2-features.md)
- [x] file format
  - [x] text
  - [x] csv
  - [x] parquet
  - [x] orc
  - [x] json

## Options

|           name            |  type   | required |    default value    |
|---------------------------|---------|----------|---------------------|
| path                      | string  | yes      | -                   |
| type                      | string  | yes      | -                   |
| fs.defaultFS              | string  | yes      | -                   |
| read_columns              | list    | yes      | -                   |
| hdfs_site_path            | string  | no       | -                   |
| delimiter                 | string  | no       | \001                |
| parse_partition_from_path | boolean | no       | true                |
| date_format               | string  | no       | yyyy-MM-dd          |
| datetime_format           | string  | no       | yyyy-MM-dd HH:mm:ss |
| time_format               | string  | no       | HH:mm:ss            |
| kerberos_principal        | string  | no       | -                   |
| kerberos_keytab_path      | string  | no       | -                   |
| skip_header_row_number    | long    | no       | 0                   |
| schema                    | config  | no       | -                   |
| common-options            |         | no       | -                   |

### path [string]

The source file path.

### delimiter [string]

Field delimiter, used to tell connector how to slice and dice fields when reading text files

default `\001`, the same as hive's default delimiter

### parse_partition_from_path [boolean]

Control whether parse the partition keys and values from file path

For example if you read a file from path `hdfs://hadoop-cluster/tmp/seatunnel/parquet/name=tyrantlucifer/age=26`

Every record data from file will be added these two fields:

|     name      | age |
|---------------|-----|
| tyrantlucifer | 26  |

Tips: **Do not define partition fields in schema option**

### date_format [string]

Date type format, used to tell connector how to convert string to date, supported as the following formats:

`yyyy-MM-dd` `yyyy.MM.dd` `yyyy/MM/dd`

default `yyyy-MM-dd`

### datetime_format [string]

Datetime type format, used to tell connector how to convert string to datetime, supported as the following formats:

`yyyy-MM-dd HH:mm:ss` `yyyy.MM.dd HH:mm:ss` `yyyy/MM/dd HH:mm:ss` `yyyyMMddHHmmss`

default `yyyy-MM-dd HH:mm:ss`

### time_format [string]

Time type format, used to tell connector how to convert string to time, supported as the following formats:

`HH:mm:ss` `HH:mm:ss.SSS`

default `HH:mm:ss`

### skip_header_row_number [long]

Skip the first few lines, but only for the txt and csv.

For example, set like following:

`skip_header_row_number = 2`

then Seatunnel will skip the first 2 lines from source files

### type [string]

File type, supported as the following file types:

`text` `csv` `parquet` `orc` `json`

If you assign file type to `json`, you should also assign schema option to tell connector how to parse data to the row you want.

For example:

upstream data is the following:

```json

{"code":  200, "data":  "get success", "success":  true}

```

You can also save multiple pieces of data in one file and split them by newline:

```json lines

{"code":  200, "data":  "get success", "success":  true}
{"code":  300, "data":  "get failed", "success":  false}

```

you should assign schema as the following:

```hocon

schema {
    fields {
        code = int
        data = string
        success = boolean
    }
}

```

connector will generate data as the following:

| code |    data     | success |
|------|-------------|---------|
| 200  | get success | true    |

If you assign file type to `parquet` `orc`, schema option not required, connector can find the schema of upstream data automatically.

If you assign file type to `text` `csv`, you can choose to specify the schema information or not.

For example, upstream data is the following:

```text

tyrantlucifer#26#male

```

If you do not assign data schema connector will treat the upstream data as the following:

|        content        |
|-----------------------|
| tyrantlucifer#26#male |

If you assign data schema, you should also assign the option `delimiter` too except CSV file type

you should assign schema and delimiter as the following:

```hocon

delimiter = "#"
schema {
    fields {
        name = string
        age = int
        gender = string 
    }
}

```

connector will generate data as the following:

|     name      | age | gender |
|---------------|-----|--------|
| tyrantlucifer | 26  | male   |

### fs.defaultFS [string]

Hdfs cluster address.

### hdfs_site_path [string]

The path of `hdfs-site.xml`, used to load ha configuration of namenodes

### kerberos_principal [string]

The principal of kerberos

### kerberos_keytab_path [string]

The keytab path of kerberos

### schema [Config]

#### fields [Config]

the schema fields of upstream data

### read_columns [list]

The read column list of the data source, user can use it to implement field projection.

The file type supported column projection as the following shown:

- text
- json
- csv
- orc
- parquet

**Tips: If the user wants to use this feature when reading `text` `json` `csv` files, the schema option must be configured**

### common options

Source plugin common parameters, please refer to [Source Common Options](common-options.md) for details.

## Example

```hocon

HdfsFile {
  path = "/apps/hive/demo/student"
  type = "parquet"
  fs.defaultFS = "hdfs://namenode001"
}

```

```hocon

HdfsFile {
  schema {
    fields {
      name = string
      age = int
    }
  }
  path = "/apps/hive/demo/student"
  type = "json"
  fs.defaultFS = "hdfs://namenode001"
}

```

## Changelog

### 2.2.0-beta 2022-09-26

- Add HDFS File Source Connector

### 2.3.0-beta 2022-10-20

- [BugFix] Fix the bug of incorrect path in windows environment ([2980](https://github.com/apache/incubator-seatunnel/pull/2980))
- [Improve] Support extract partition from SeaTunnelRow fields ([3085](https://github.com/apache/incubator-seatunnel/pull/3085))
- [Improve] Support parse field from file path ([2985](https://github.com/apache/incubator-seatunnel/pull/2985))

### next version

- [Improve] Support skip header for csv and txt files ([3900](https://github.com/apache/incubator-seatunnel/pull/3840))
- [Improve] Support kerberos authentication ([3840](https://github.com/apache/incubator-seatunnel/pull/3840))

