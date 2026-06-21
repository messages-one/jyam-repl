
# JYam Complete Usage Guide

## Table of Contents
1. [Introduction](#introduction)
   2. [Installation & Setup](#installation--setup)
   3. [REPL Mode](#repl-mode)
   4. [Query Syntax](#query-syntax)
   5. [File Operations](#file-operations)
   6. [Data Display](#data-display)
   7. [Data Modification](#data-modification)
   8. [Validation](#validation)
   9. [Binary Operations](#binary-operations)
   10. [Infrastructure Mode](#infrastructure-mode)
   11. [Scripting & Automation](#scripting--automation)
   12. [Cloud Schema Management](#cloud-schema-management)
   13. [Advanced Features](#advanced-features)
   14. [Troubleshooting](#troubleshooting)

---

## Introduction

JYam is a zero-dependency, cloud-native toolkit for working with JSON, YAML, MessagePack, and CBOR data. It provides an interactive REPL with powerful querying, validation, and infrastructure management capabilities.

### Key Features
- **Multi-format support**: JSON, YAML, MessagePack, CBOR
  - **Interactive REPL**: Query and modify data in real-time
  - **JSONPath support**: Standard and extended path queries
  - **Schema validation**: JSON Schema and OpenAPI 3.1 validation
  - **Infrastructure mode**: Build and deploy cloud resources
  - **Binary operations**: Compress, decompress, and convert between formats
  - **Scripting**: Variables, functions, loops, and conditionals
  - **Streaming**: Handle large files with memory-mapped I/O
  - **Cloud schemas**: Download and use AWS, Azure, GCP schemas

---

## Installation & Setup

# Run the REPL
on Windows you can download jyam.exe
 if your organization firewall blocks executables then
      choose the version which best matches your jdk distribution
      the jar file follows the same pattern:   jyam-java<jdk-version>-1.0.0.jar

    java -jar <path-to-jar>  
    ex:  java -jar jyam-java26-1.0.0.jar

### Main Menu Options

```
1. REPL Mode - Query/Modify JSON/YAML files
2. Infrastructure Mode - Build cloud resources
3. Binary Mode - Convert to/from MsgPack/CBOR
4. Validate Mode - JSON Schema / OpenAPI validation
5. Help
6. Exit
```

### REPL Prompt

```
jyam (filename | MODE) [CLOUD] > 
```

- **filename**: Currently loaded file
  - **MODE**: JSON or YAML (display mode)
  - **CLOUD**: Active cloud provider (AWS, Azure, GCP)

---

## Query Syntax

### Overview

JYam supports **two query syntaxes**:

1.  Dot Notation  (JYam native)    - Simple and fast
2.  JSONPath      (Standard)          - Full JSONPath support

### Dot Notation

#### Basic Queries

```bash
jyam>  .contexts

# Nested properties
jyam>  .contexts.source.region

# Array access
jyam>  .contexts.source.zones[0]

# Multiple array elements
jyam> .contexts.source.zones[0], .contexts.source.zones[1]
```

#### Keys with Special Characters

**Rule**: Use quotes for keys containing `.`, `/`, `@`, ` ` (space), or other special characters.

```bash
# Key with :: (no quotes needed if no . in key)
.PropertyTypes.AWS::ACMPCA::Certificate

# Key with :: and . (quotes needed)
.PropertyTypes."AWS::ACMPCA::Certificate.ApiPassthrough"

# Key with / (quotes needed)
.contexts.source.name."blogging-app/okd311-cluster"

# Key with spaces
.user."First Name"

# Key with @
.data."@timestamp"

# Mixed special characters
.data."my.key@with/special:chars"
```

#### Nested Access with Special Keys

```bash
# Full path with special keys
.PropertyTypes."AWS::ACMPCA::Certificate.ApiPassthrough".Properties.Subject.Type

# Mixed notation
.contexts.source.name."blogging-app/okd311-cluster".region
```

### JSONPath Syntax

Use the `path` command for JSONPath expressions.

#### Basic JSONPath

```bash
# Root
path $

# Property access
path $.contexts

# Nested properties
path $.contexts.source.region

# Array access
path $.contexts.source.zones[0]

# Wildcard
path $.contexts.*
```

#### Bracket Notation

```bash
# Keys with special characters
path "$.PropertyTypes['AWS::ACMPCA::Certificate.ApiPassthrough']"

# With double quotes
path "$.PropertyTypes[\"AWS::ACMPCA::Certificate.ApiPassthrough\"]"

# Array with bracket notation
path "$.contexts.source.zones[0]"

# Multiple levels
path "$.PropertyTypes['AWS::ACMPCA::Certificate.ApiPassthrough'].Properties.Subject"
```

#### JSONPath Operators

```bash
# Wildcard - all properties
path $.contexts.*.region

# Recursive descent
path $..name

# Array slicing (if supported)
path $.contexts.source.zones[0:2]

# Filter expressions
path "$.contexts[?(@.region == 'us-west1')]"
path "$.contexts[?(@.region)]"
```

### Comparing Syntaxes

| Feature | Dot Notation | JSONPath |
|---------|--------------|----------|
| Simple properties | `.contexts.source` | `path $.contexts.source` |
| Array access | `.zones[0]` | `path $.zones[0]` |
| Special chars | `."key.with.dots"` | `path $['key.with.dots']` |
| Wildcards | Not supported | `path $.contexts.*` |
| Recursive | Not supported | `path $..name` |
| Filters | Not supported | `path "$[?(@.region)]"` |

---

## File Operations

### Loading Files

```bash
# Load JSON file
jyam> load data.json

# Load YAML file
jyam> load config.yaml

# Large file (>50MB) - automatically uses streaming mode
jyam> load large-file.json
```

**Supported Formats:**
- JSON (RFC 8259)
  - YAML (1.2)
  - JSON with comments
  - YAML with anchors and aliases

### Saving Files

```bash
# Save to current file
jyam> save

# convert from json to yaml
jyam>  load config.yaml
jyam>  save config.json
```

### Auto-detection

JYam automatically detects file format:
- `.json` → JSON
  - `.yaml` / `.yml` → YAML
  - `.msgpack` → MessagePack
  - `.cbor` → CBOR

---

## Data Display

### Display Modes

```bash
# Pretty JSON (default)
pretty

# Compact JSON
json
pretty off

# YAML format
yaml

# Switch back to JSON
json
```

### Viewing Data

```bash
# Display current data
# (Just press Enter or type any command that triggers display)

# Show tree structure
tree

# Show specific properties
.contexts.source

# Show formatted output
# Data is automatically formatted based on mode
```

### Output Formatting

```bash
# Pretty print (default)
pretty

# Compact (single line)
pretty off

# YAML format
yaml

# Interactive formatting
# Strings are quoted, objects are pretty-printed
```

---

## Data Modification

### Setting Values

```bash
# Set simple value
.contexts.source.region = "us-west2"

# Set nested value
.contexts.source.cluster.name = "new-cluster"

# Set array value
.contexts.source.zones[0] = "us-west1-a"

# Set with variable
let region = "us-east1"
.contexts.source.region = $region

# Set with computed value
.contexts.source.port = 8080
```

### Deleting Values

```bash
# Delete property
del .contexts.source.region

# Delete array element
del .contexts.source.zones[0]

# Delete nested property
del .contexts.source.cluster.name
```

### Variable Assignment

```bash
# Simple variable
let name = "production"

# Variable with path
let region = .contexts.source.region

# Variable with special characters
let cert = "AWS::ACMPCA::Certificate.ApiPassthrough"

# Use variable in path
.PropertyTypes[$cert].Properties

# Display variable
$name

# List all variables
vars
```

---

## Validation

### JSON Schema Validation

```bash
# Validate current data against schema
validate schema.json

# In validate mode
validate-mode
  Option 1: Validate data against JSON Schema
  Option 2: Validate OpenAPI 3.1 document
  Option 3: Validate both
```

### OpenAPI Validation

```bash
# Load OpenAPI file
load openapi.yaml

# Validate
openapi

# Or in validate mode
validate-mode -> Option 2
```

### Combined Validation

```bash
# Validate both data and OpenAPI
validate-mode -> Option 3

# Interactive prompts:
# Enter data file (JSON/YAML): data.json
# Enter schema file (JSON): schema.json
# Enter OpenAPI file (JSON/YAML): openapi.yaml
```

---

## Binary Operations

### Converting to Binary Formats

```bash
# Convert current data to MessagePack
bin

# Show size comparison
compress

# Convert JSON to MessagePack (binary mode)
binary-mode -> Option 1

# Convert JSON to CBOR
binary-mode -> Option 3

# Compress with GZIP
binary-mode -> Option 6
```

### Unpacking Binary Formats

```bash
# Unpack MessagePack
unpack msgpack data.msgpack

# Unpack CBOR
unpack cbor data.cbor

# Decompress GZIP+MsgPack
binary-mode -> Option 2

# Decompress GZIP+CBOR
binary-mode -> Option 4
```

### Binary Size Comparison

```bash
# Show comparison of formats
binary-mode -> Option 5

# Output:
# Format                Size       Ratio
# JSON                  1024 bytes  100.0%
# MessagePack           512  bytes   50.0%
# CBOR                  480  bytes   46.9%
# GZIP + MessagePack    320  bytes   31.3%
```

---

## Infrastructure Mode

### Starting Infrastructure Mode

From main menu:
```
2. Infrastructure Mode - Build cloud resources
```

### Selecting Cloud Provider

```
Select Cloud Provider:
  1. AWS
  2. Azure
  3. GCP
  b. Back
```

### Browsing Resources

```
AWS Services
--------------------------------------------------
   1. EC2 (15 resources)
   2. S3 (8 resources)
   3. Lambda (6 resources)
   4. RDS (4 resources)
   ...

Select service number to browse resources: 1

EC2 Resources
--------------------------------------------------
    1. VPC
    2. Subnet
    3. Instance
    4. SecurityGroup
    5. InternetGateway
    ...

Select resource numbers (comma-separated): 1,3
Added: AWS::EC2::VPC
Added: AWS::EC2::Instance
```

### Configuring Resources

```
Infrastructure Tree
--------------------------------------------------
  [X] my-infra (Project)
    [X] vpc (AWS::EC2::VPC)
    [ ] instance (AWS::EC2::Instance)

  [A]dd  [C]onfigure  [D]elete  [R]eview  [S]ave  [M]enu
> C

Enter resource name to configure: instance

Configure AWS::EC2::Instance
--------------------------------------------------
InstanceType (string) [t2.micro]: t2.large
ImageId (string) [ami-12345]: ami-67890
SubnetId (string): subnet-abc123
SecurityGroupIds (array) []: sg-12345
KeyName (string) [my-key]: prod-key

instance configured
```

### Deploying Resources

```bash
# Review configuration
R

Infrastructure Summary
--------------------------------------------------
  [X] my-infra (Project)
    [X] vpc (AWS::EC2::VPC)
      - CidrBlock: 10.0.0.0/16
    [X] instance (AWS::EC2::Instance)
      - InstanceType: t2.large
      - ImageId: ami-67890
      - SubnetId: subnet-abc123

Deploy to AWS? (y/n): y

Generating CloudFormation template...
AWSTemplateFormatVersion: 2010-09-09
Description: Infrastructure built with JYam
Resources:
  vpc123456789:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
  instance123456790:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.large
      ImageId: ami-67890
      SubnetId: subnet-abc123

Save to file? (y/n): y
Saved to cfn-template.yaml
```

### Saving Projects

```bash
# Save infrastructure project
S

Save as: my-infra-project.jyam
Saved to my-infra-project.jyam
```

---

## Scripting & Automation

### Variables

```bash
# Define variable
let name = "production"
let region = .contexts.source.region
let zones = .contexts.source.zones

# Use variable
echo "Region is ${region}"
path $.PropertyTypes[$cert]

# List variables
vars

# Delete variable
del $name
```

### Functions

```bash
# Define function
function getRegion()
Enter function body (type 'endfunction' on a new line to finish):
  .contexts.source.region
endfunction

# Define function with parameters
function sum(a, b)
Enter function body:
  echo ${a} + ${b}
  let result = ${a} + ${b}
  echo "Result: ${result}"
endfunction

# Call function
call getRegion()
call sum(5, 3)
```

### For Loops

```bash
# Process all JSON files
for json
Enter loop body (type 'end' to finish):
  load ${file}
  echo "Processing: ${filename}"
  path .contexts.source.region
end

# Process all YAML files recursively
recurse yaml
Enter loop body:
  load ${file}
  echo ${filename}
end

# Loop with conditions
for json
Enter loop body:
  load ${file}
  if .contexts.source.region == "us-west1"
    echo "Found West Coast deployment"
  endif
end
```

### If Statements

```bash
# Simple if
if .contexts.source.region == "us-west1"
Enter if body (type 'endif' on a new line to finish):
  echo "West Coast deployment"
endif

# If-else
if .contexts.source.region == "us-west1"
Enter if body:
  echo "West Coast"
else
  echo "Other region"
endif

# Complex conditions
if .contexts.source.region == "us-west1" && .contexts.source.zones[0] == "us-west1-b"
  echo "Specific zone"
endif

if .contexts.source.region == "us-west1" || .contexts.source.region == "us-east1"
  echo "US region"
endif

# Contains
if .contexts.source.repo.url.https contains "boutique"
  echo "Boutique repository"
endif
```

### Combining Scripting

```bash
# Complex script example
let env = "production"
let region = .contexts.source.region

function deploy(service)
Enter function body:
  echo "Deploying ${service} to ${env} in ${region}"
  .contexts.target.name = ${service}
  .contexts.target.region = ${region}
endfunction

call deploy("api-service")

# Loop with function
for json
  load ${file}
  if .contexts.source.region == "us-west1"
    call deploy(.contexts.source.name)
  endif
end
```

---

## Cloud Schema Management

### Downloading Schemas

```bash
# In infrastructure mode, schemas are automatically downloaded
# from cloud provider APIs

# AWS
# Download from: https://d1uauaxba7bl26.cloudfront.net/latest/gzip/CloudFormationResourceSpecification.json
# Saved to: ~/.jyam/schemas/aws/latest.json

# Azure
# Download from: https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json
# Saved to: ~/.jyam/schemas/azure/latest.json

# GCP
# Download from: https://deploymentmanager.googleapis.com/$discovery/rest?version=v2
# Saved to: ~/.jyam/schemas/gcp/latest.json
```

### Manual Schema Management

```bash
# Schema location
~/.jyam/schemas/{cloud}/latest.json

# Custom schema
~/.jyam/schemas/{cloud}/custom.json
```

---

## Advanced Features

### Streaming Large Files

```bash
# Files > 50MB automatically use streaming mode
load large-file.json

# Streaming indicator in prompt
jyam (large-file.json [STREAM] | JSON) >

# Queries work the same way
path $.PropertyTypes."AWS::ACMPCA::Certificate"

# Cannot modify in streaming mode
# Save will show error
save
# Output: Cannot save in streaming mode
```

### YAML Streaming Chunking

```bash
# Stream YAML blocks by key
chunk resources config.yaml

# Outputs each block separately
[BLOCK: resources]
  - name: my-vpc
    type: AWS::EC2::VPC
[BLOCK: resources]
  - name: my-instance
    type: AWS::EC2::Instance
```

### External Command Execution

```bash
# Execute shell command
exec ls -la
exec echo "Hello from shell"
exec kubectl get pods

# On Windows
exec dir
exec type file.txt
```

### History

```bash
# Show command history
history

# Output:
  1 load config.yaml
  2 path .contexts.source
  3 let region = .contexts.source.region
  4 echo ${region}
```

### Trace Mode

```bash
# Enable tracing
trace

# All commands are printed before execution
[TRACE] Executing: load config.yaml
[TRACE] Executing: path .contexts.source

# Disable tracing
trace
```

---

## Troubleshooting

### Common Issues

#### 1. "null (path not found)"

**Problem**: Path doesn't exist in data

**Solutions**:
```bash
# Check if path exists
tree

# Check parent object
path .contexts

# Use wildcard
path .contexts.*

# Use JSONPath to search
path $..region
```

#### 2. Keys with Special Characters

**Problem**: Keys with `.`, `/`, `::` not found

**Solutions**:
```bash
# Use quotes for keys with .
."key.with.dots"

# Use quotes for keys with /
."key/with/slashes"

# Use JSONPath bracket notation
path "$['key.with.dots']"
path "$['key/with/slashes']"

# For AWS :: keys with .
.PropertyTypes."AWS::ACMPCA::Certificate.ApiPassthrough"
```

#### 3. "Cannot save in streaming mode"

**Problem**: File > 50MB loaded in streaming mode

**Solutions**:
```bash
# Load without streaming (for small files)
# Streaming is automatic for large files

# Work with the data as-is
# Or use queries to extract needed parts
```

#### 4. Schema Download Fails

**Problem**: Can't download cloud schema

**Solutions**:
```bash
# Check internet connection
exec ping google.com

# Use fallback resources
# Infrastructure mode will use fallback if download fails

# Manual download
# Download schema and place in ~/.jyam/schemas/{cloud}/latest.json
```

### Debugging Tips

```bash
# Enable trace mode
trace

# Show current data type
# Check if data is loaded
path $

# Display as tree
tree

# Show variables
vars

# Check command history
history
```

### Performance Tips

```bash
# Use streaming for large files
# >50MB automatically streamed

# Use compact JSON for large outputs
json
pretty off

# Use path command for complex queries
path $.PropertyTypes.*.Properties

# Use variables to avoid repeated queries
let cert = .PropertyTypes."AWS::ACMPCA::Certificate"
path $cert.Properties.Subject
```

---

## Quick Reference

### Essential Commands

| Command | Description |
|---------|-------------|
| `load <file>` | Load JSON/YAML file |
| `save [file]` | Save current data |
| `path <query>` | Execute JSONPath query |
| `.path` | Dot notation query |
| `pretty` | Toggle pretty printing |
| `yaml` / `json` | Switch output format |
| `tree` | Display tree structure |
| `let name = value` | Define variable |
| `vars` | Show variables |
| `del <path>` | Delete property |
| `validate <schema>` | Validate against schema |
| `openapi` | Validate OpenAPI |
| `help` | Show help |

### Query Syntax Examples

| Query | Description |
|-------|-------------|
| `.contexts.source` | Dot notation |
| `path $.contexts.source` | JSONPath |
| `.PropertyTypes."AWS::ACMPCA"` | Key with :: |
| `path "$['AWS::ACMPCA']"` | JSONPath bracket |
| `.contexts.source.zones[0]` | Array access |
| `path $.contexts.source.zones[0]` | JSONPath array |
| `.contexts.*` | Not supported in dot notation |
| `path $.contexts.*` | JSONPath wildcard |

### Special Character Handling

| Character | Dot Notation | JSONPath |
|-----------|--------------|----------|
| `.` (in key) | `."key.with.dots"` | `$['key.with.dots']` |
| `/` (in key) | `."key/with/slash"` | `$['key/with/slash']` |
| `::` (in key) | `.key::with::colons` | `$['key::with::colons']` |
| `@` (in key) | `."@timestamp"` | `$['@timestamp']` |
| Space (in key) | `."key with spaces"` | `$['key with spaces']` |

---

## Examples

### Example 1: Query AWS CloudFormation Property

```bash
# Load AWS schema
load ~/.jyam/schemas/aws/latest.json

# Query EC2 Instance properties
.PropertyTypes."AWS::EC2::Instance".Properties

# Get InstanceType property details
.PropertyTypes."AWS::EC2::Instance".Properties.InstanceType

# Get documentation
.PropertyTypes."AWS::EC2::Instance".Properties.InstanceType.Documentation
```

### Example 2: Infrastructure Deployment

```bash
# Start infrastructure mode
# Select AWS
# Add VPC and Instance
# Configure resources
# Review and deploy
# Save CloudFormation template
```

### Example 3: Bulk Processing

```bash
# Process all JSON files
for json
Enter loop body:
  load ${file}
  echo "Processing ${filename}"
  path .metadata.name
  let env = .metadata.labels.environment
  echo "Environment: ${env}"
end
```

### Example 4: Validation Pipeline

```bash
# Validate mode
validate-mode
Option 3

# Enter files:
# Data: data.json
# Schema: schema.json
# OpenAPI: openapi.yaml

# Get combined validation results
```

---

## Conclusion

JYam provides a powerful, unified interface for working with structured data across multiple formats and cloud platforms. The combination of intuitive dot notation with full JSONPath support, validation, infrastructure management, and scripting capabilities makes it an essential tool for cloud-native development.

### Key Takeaways

1. **Dot notation** for simple, fast queries
   2. **JSONPath** for complex queries with wildcards and filters
   3. **Quotes** for keys with special characters (`.`, `/`, etc.)
   4. **`::`** is preserved as part of key names (AWS resource types)
   5. **Streaming** for large files > 50MB
   6. **Infrastructure mode** for building and deploying cloud resources
   7. **Scripting** with variables, functions, loops, and conditionals
   8. **Validation** for JSON Schema and OpenAPI 3.1

### Getting Help

```bash
# REPL help
help

# Command-specific help
# Most commands show usage with invalid syntax

# Check documentation
# See examples above for common use cases
```
