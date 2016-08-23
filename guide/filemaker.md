# FileMaker

The FileMaker Server connector provides access to FileMaker Server databases using the XML Web publishing interface. It allows you to allows you to search for, add, update and delete records in a hosted FileMaker database.

## System Requirements

This module uses libxml2 and libcurl along with the [Perfect-XML](https://github.com/PerfectlySoft/Perfect-XML) and [Perfect-CURL](https://github.com/PerfectlySoft/Perfect-CURL) packages.

### macOS

macOS currently includes suitable versions of the dependent libraries and no manual installation is nessesary.

### Linux

Install the libcurl4-openssl-dev &amp; libxml2-dev packages through apt.

```
sudo apt-get install libcurl4-openssl-dev libxml2-dev
```

## Setup

Add the "Perfect-FileMaker" project as a dependency in your Package.swift file:

```swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-FileMaker.git", versions: Version(0,0,0)..<Version(10,0,0))
```

### Import

In any of the source files where you intend to use with this module, add the following import: 

```swift
import PerfectFileMaker
```

## Overview

The main objects that you will be working with when accessing Filemaker databases are described here.

### struct FileMakerServer

This is the main interface to a FileMaker Server. Before accessing any database you will need to create an instance of this struct and provide a server host name or IP address, a port, and a valid username and password for the server. If the given port is 443 then all requests will be encrypted HTTPS.

Once the server connection is instantiated you can perform four different operations. These are: 

* list available databases
* list a database's layouts
* list a layout's fields
* perform a query

In all cases you provide a callback and the operation occurs asynchronously. When the operation has completed the callback is called and given a closure which, when executed, will either return the operation response object or throw an error. The type of the operation response object will differ depending on the type of operation. The exception error object provides access to the error code and accompanying message string. It will be of the following type:

```swift 
public enum FMPError: Error {
	/// An error code and message.
	case serverError(Int, String)
}
```

The relevent portions of the FileMakerServer struct are defined as follows:

```swift
/// A connection to a FileMaker Server instance.
/// Initialize using a host, port, username and password.
public struct FileMakerServer {
	/// Initialize using a host, port, username and password.
	public init(host: String, port: Int, userName: String, password: String)
	/// Retrieve the list of database hosted by the server.
	public func databaseNames(completion: @escaping (() throws -> [String]) -> ())
	/// Retrieve the list of layouts for a particular database.
	public func layoutNames(database: String, completion: @escaping (() throws -> [String]) -> ())
	/// Get a database's layout information. Includes all field and portal names.
	public func layoutInfo(database: String,
	                       layout: String,
	                       completion: @escaping (() throws -> FMPLayoutInfo) -> ())
	/// Perform a query and provide any resulting data. 
	public func query(_ query: FMPQuery, completion: @escaping (() throws -> FMPResultSet) -> ())
}
```

When listing database or layout names, the response object will simply be an array of strings. When retrieving layout information, the response object will be a FMPLayoutInfo object. When performing a query the response object will be a FMPResultSet.

### struct FMPLayoutInfo

FMPLayoutInfo is defined as follows:

```swift
/// Represents meta-information about a particular layout.
public struct FMPLayoutInfo {
	/// Each field or related set as a list.
	public let fields: [FMPMetaDataItem]
	/// Each field or related set keyed by name.
	public let fieldsByName: [String:FMPFieldType]
}
```

This object contains all of the information pertaining to the fields on a layout. This includes the field names and types and also indicates which portals/relations are on the layout and which fields are contained within them.

Field information is provided in two different ways. The first is represented by an array of FMPMetaDataItems. This enum value indicates if the item is a regular field or a portal/relation. The second is a dictionary containing the full field name and the field type. If the field is in a relation it will be named with the standard FileMaker PortalName::FieldName syntax.

What follows are the definitions for FMPMetaDataItem, FMPFieldDefinition &amp; FMPFieldType.

```swift
/// Represents either an individual field definition or a related (portal) definition.
public enum FMPMetaDataItem {
	/// An individual field.
	case fieldDefinition(FMPFieldDefinition)
	/// A related set. Indicates the portal name and its contained fields.
	case relatedSetDefinition(String, [FMPFieldDefinition])
}
```

```swift
/// A FileMaker field definition. Indicates a field name and type.
public struct FMPFieldDefinition {
	/// The field name.
	public let name: String
	/// The field type.
	public let type: FMPFieldType
}
```

```swift
/// One of the possible FileMaker field types.
public enum FMPFieldType {
	/// A text field.
	case text
	/// A numeric field.
	case number
	/// A container field.
	case container
	/// A date field.
	case date
	/// A time field.
	case time
	/// A timestamp field.
	case timestamp
}
```

### struct FMPResultSet

An FMPResultSet object contains the result of a FileMaker query. This includes database &amp; layout meta information, as well as the number of records which were found in total and a each record in the found set.

```swift
/// The result set produced by a query.
public struct FMPResultSet {
	/// Database meta-info.
	public let databaseInfo: FMPDatabaseInfo
	/// Layout meta-info.
	public let layoutInfo: FMPLayoutInfo
	/// The number of records found by the query.
	public let foundCount: Int
	/// The list of records produced by the query.
	public let records: [FMPRecord]
}
```

In addition to the previously described FMPLayoutInfo struct, a result set also contains the following bits of database related information:

```swift
/// Meta-information for a database.
public struct FMPDatabaseInfo {
	/// The date format indicated by the server.
	public let dateFormat: String
	/// The time format indicated by the server.
	public let timeFormat: String
	/// The timestamp format indicated by the server.
	public let timeStampFormat: String
	/// The total number of records in the database.
	public let recordCount: Int
}
```

The found record data is accessed through the ```FMPResultSet.records``` property. This array of FMPRecords will contain one element for each returned record.

```swift
/// An individual result set record.
public struct FMPRecord {
	/// A type of record item.
	public enum RecordItem {
		/// An individual field.
		case field(String, FMPFieldValue)
		/// A related set containing a list of related records.
		case relatedSet(String, [FMPRecord])
	}
	/// The record id.
	public let recordId: Int
	/// The contained record items keyed by name.
	public let elements: [String:RecordItem]
}
```

A record holds each field or portal item keyed by name in its ```elements``` property. The values in this dictionary are ```FMPRecord.RecordItem``` enum objects. These objects indicate if the item is a regular field, in which case its name and value can be directly accessed, or if the item is a related record set. If the item is a related record set then the portal name and an array of nested records are provided.

In either case a field's value is represented through the FMPFieldValue struct.

```swift
/// A returned FileMaker field value.
public enum FMPFieldValue: CustomStringConvertible {
	/// A text field.
	case text(String)
	/// A numeric field.
	case number(Double)
	/// A container field.
	case container(String)
	/// A date field.
	case date(String)
	/// A time field.
	case time(String)
	/// A timestamp field.
	case timestamp(String)
}
```

## Examples

The following code snippets illustrate the basic activities that one would perform against FileMaker databases.

### List Available Databases

This snippet connects to the server and has it list all of the hosted databases.

```swift
let fms = FileMakerServer(host: testHost, port: testPort, userName: testUserName, password: testPassword)
fms.databaseNames {
	result in
	do {
		// Get the list of names
		let names = try result()
		for name in names {
			print("Got a database name \(name)")
		}
	} catch FMPError.serverError(let code, let msg) {
		print("Got a server error \(code) \(msg)")
	} catch let e {
		print("Got an unexpected error \(e)")
	}
}
```

### List Available Layouts

List all of the layouts in a particular database.

```swift
let fms = FileMakerServer(host: testHost, port: testPort, userName: testUserName, password: testPassword)
fms.layoutNames(database: "FMServer_Sample") {
	result in
	guard let names = try? result() else {
		return // got an error
	}
	for name in names {
		print("Got a layout name \(name)")
	}
}
```

### List Field On Layout

List all of the field names on a particular layout.

```swift
let fms = FileMakerServer(host: testHost, port: testPort, userName: testUserName, password: testPassword)
fms.layoutInfo(database: "FMServer_Sample", layout: "Task Details") {
	result in
	guard let layoutInfo = try? result() else {
		return // error
	}
	let fieldsByName = layoutInfo.fieldsByName
	for (name, value) in fieldsByName {
		print("Field \(name) = \(value)")
	}
}
```

### Find All Records

Perform a findall and print all field names and values.

```swift
let query = FMPQuery(database: "FMServer_Sample", layout: "Task Details", action: .findAll)
let fms = FileMakerServer(host: testHost, port: testPort, userName: testUserName, password: testPassword)
fms.query(query) {
	result in
	guard let resultSet = try? result() else {
		return // error
	}
	let fields = resultSet.layoutInfo.fields
	let records = resultSet.records
	let recordCount = records.count
	for i in 0..<recordCount {
		let rec = records[i]
		for field in fields {
			switch field {
			case .fieldDefinition(let def):
				let fieldName = def.name
				if let fnd = rec.elements[fieldName], case .field(_, let fieldValue) = fnd {
					print("Normal field: \(fieldName) = \(fieldValue)")
				}
			case .relatedSetDefinition(let name, _):
				guard let fnd = rec.elements[name], case .relatedSet(_, let relatedRecs) = fnd else {
					continue
				}
				print("Relation: \(name)")
				for relatedRec in relatedRecs {
					for relatedRow in relatedRec.elements.values {
						if case .field(let fieldName, let fieldValue) = relatedRow {
							print("\tRelated field: \(fieldName) = \(fieldValue)")
						}
					}
				}
			}
		}
	}
}
```

### Find All Records With Skip &amp; Max

To add skip and max, the query above would be amended as follows:

```swift
// Skip two records and return a max of two records.
let query = FMPQuery(database: "FMServer_Sample", layout: "Task Details", action: .findAll)
	.skipRecords(2).maxRecords(2)
...
```

### Find Records Where "Status" Is "In Progress"

Find all records where the field "Status" has the value of "In Progress".

```swift
let qfields = [FMPQueryFieldGroup(fields: [FMPQueryField(name: "Status", value: "In Progress")])]
let query = FMPQuery(database: "FMServer_Sample", layout: "Task Details", action: .find)
	.queryFields(qfields)
let fms = FileMakerServer(host: testHost, port: testPort, userName: testUserName, password: testPassword)
fms.query(query) {
	result in
	guard let resultSet = try? result() else {
		return // error
	}
	let fields = resultSet.layoutInfo.fields
	let records = resultSet.records
	let recordCount = records.count
	for i in 0..<recordCount {
		let rec = records[i]
		for field in fields {
			switch field {
			case .fieldDefinition(let def):
				let fieldName = def.name
				if let fnd = rec.elements[fieldName], case .field(_, let fieldValue) = fnd {
					print("Normal field: \(fieldName) = \(fieldValue)")
					if name == "Status", case .text(let tstStr) = fieldValue {
						print("Status == \(tstStr)")
					}
				}
			case .relatedSetDefinition(let name, _):
				guard let fnd = rec.elements[name], case .relatedSet(_, let relatedRecs) = fnd else {
					continue
				}
				print("Relation: \(name)")
				for relatedRec in relatedRecs {
					for relatedRow in relatedRec.elements.values {
						if case .field(let fieldName, let fieldValue) = relatedRow {
							print("\tRelated field: \(fieldName) = \(fieldValue)")
						}
					}
				}
			}
		}
	}
}
```
