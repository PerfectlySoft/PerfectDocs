# FileMaker

The FileMaker Server connector provides access to FileMaker Server databases using the XML Web publishing interface. It allows you to search for, add, update and delete records in a hosted FileMaker database.

## System Requirements

This module uses libxml2 and libcurl along with the [Perfect-XML](https://github.com/PerfectlySoft/Perfect-XML) and [Perfect-CURL](https://github.com/PerfectlySoft/Perfect-CURL) packages.

### macOS

macOS currently includes suitable versions of the dependent libraries and no manual installation is necessary.

### Linux

Install the libcurl4-openssl-dev &amp; libxml2-dev packages through apt.

```
sudo apt-get install libcurl4-openssl-dev libxml2-dev
```

## Setup

Add the "Perfect-FileMaker" project as a dependency in your Package.swift file:

```swift
.Package(
	url:"https://github.com/PerfectlySoft/Perfect-FileMaker.git",
	majorVersion: 3
	)
```

### Import

In any of the source files where you intend to use with this module, add the following import: 

```swift
import PerfectFileMaker
```

## Overview

The main objects that you will be working with when accessing FileMaker databases are described here.

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

The relevant portions of the FileMakerServer struct are defined as follows:

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

## Queries

The ```FileMakerServer.query``` function lets you search for sets of records or manipulate individual records. You specify a query using a FMPQuery struct. This struct is then formatted as a FileMaker XML query string and sent to the FileMaker Server. The server returns its response as XML. This XML is parsed and converted to a FMPResultSet object.

The query strings generated by the FMPQuery object correspond to what's defined in the "FileMaker® Server 12 Custom Web Publishing with XML" document. [PDF Doc](https://www.filemaker.com/support/product/docs/12/fms/fms12_cwp_xml_en.pdf).

A query is created by instantiating a FMPQuery struct and then adding options to it. Each option add will return a new modified object. In this manner options can be "chained" to construct the desired final query object. The query object is then given to the ```FileMakerServer.query``` function and the results are generated.

FMPQuery is as follows:

```swift
/// An individual query & database action.
public struct FMPQuery: CustomStringConvertible {
	/// Initialize with a database name, layout name & database action.
	public init(database: String, layout: String, action: FMPAction)
	/// Sets the record id and returns the adjusted query.
	public func recordId(_ recordId: Int) -> FMPQuery
	/// Adds the query fields and returns the adjusted query.
	public func queryFields(_ queryFields: [FMPQueryFieldGroup]) -> FMPQuery
	/// Adds the query fields and returns the adjusted query.
	public func queryFields(_ queryFields: [FMPQueryField]) -> FMPQuery
	/// Adds the sort fields and returns the adjusted query.
	public func sortFields(_ sortFields: [FMPSortField]) -> FMPQuery
	/// Adds the indicated pre-sort scripts and returns the adjusted query.
	public func preSortScripts(_ preSortScripts: [String]) -> FMPQuery
	/// Adds the indicated pre-find scripts and returns the adjusted query.
	public func preFindScripts(_ preFindScripts: [String]) -> FMPQuery
	/// Adds the indicated post-find scripts and returns the adjusted query.
	public func postFindScripts(_ postFindScripts: [String]) -> FMPQuery
	/// Sets the response layout and returns the adjusted query.
	public func responseLayout(_ responseLayout: String) -> FMPQuery
	/// Adds response fields and returns the adjusted query.
	public func responseFields(_ responseFields: [String]) -> FMPQuery
	/// Sets the maximum records to fetch and returns the adjusted query.
	public func maxRecords(_ maxRecords: Int) -> FMPQuery
	/// Sets the number of records to skip in the found set and returns the adjusted query.
	public func skipRecords(_ skipRecords: Int) -> FMPQuery
	/// Returns the formulated query string.
	/// Useful for debugging purposes.
	public var queryString: String
}
```

A FMPQuery is first instantiated with a database name, layout name and action. The possible actions are:

```swift
/// A database action.
public enum FMPAction: CustomStringConvertible {
	/// Perform a search given the current query.
	case find
	/// Find all records in the database.
	case findAll
	/// Find and retrieve a random record.
	case findAny
	/// Create a new record given the current query data.
	case new
	/// Edit (update) the record indicated by the record id with the current query fields/values.
	case edit
	/// Delete the record indicated by the current record id.
	case delete
	/// Duplicate the record indicated by the current record id.
	case duplicate
}
```

Many of the FMPQuery functions accepts Strings or Ints and are self-explanatory. Any function which accepts an array can be called multiple times and the new values will be appended to the existing values.

When performing an ```.edit```, ```.delete``` or ```.duplicate``` action a record id must be set by calling ```FMPQuery.recordId(_ recordId: Int)```. Setting a record id when performing a ```.find``` will retrieve only the indicated record.

It is possible to search on one layout but return fields from another. Call ```FMPQuery.responseLayout(_ responseLayout: String)``` to set the response layout. By default the layout which is searched on will be the response layout.

It is possible and advisable, for performance reasons, to return only the minimum of required fields in a result. The names of the fields you want returned in the result can be set through the ```FMPQuery.responseFields(_ responseFields: [String])``` function.

Sorting and the usage of query fields are detailed below.

### Sorting

Records in a result set can be returned sorted by indicating one or more sort fields and sort orders. This is accomplished by calling the ```FMPQuery.sortFields``` function and passing the desired fields and orders as an array of FMPSortField objects. FMPSortField uses the FMPSortOrder enum to indicate the sort order. Both are defined as follows:

```swift
/// A record sort order.
public enum FMPSortOrder: CustomStringConvertible {
	/// Sort the records by the indicated field in ascending order.
	case ascending
	/// Sort the records by the indicated field in descending order.
	case descending
	/// Sort the records by the indicated field in a custom order.
	case custom
}

/// A sort field indicator.
public struct FMPSortField {
	/// The name of the field on which to sort.
	public let name: String
	/// A field sort order.
	public let order: FMPSortOrder
	/// Initialize with a field name and sort order.
	public init(name: String, order: FMPSortOrder)
	/// Initialize with a field name using the default FMPSortOrder.ascending sort order.
	public init(name: String)
}
```

### Query Fields

Query fields are added to a FMPQuery to indicate either fields which should be modified, in the case of an ```.edit``` action or fields which should be searched on, in the case of a ```.find``` action. Query fields hold a field name along with a corresponding value. In the case of the ```.find``` action, query fields also hold a field level operator. These operators represent a relation of a field to the indicated value. For example, a field operator could indicate that a field's contents should be greater-than-or-equal to the value when performing a search. The default field level operator is begins-with (as is standard for FileMaker database searches).

Individual query fields are represented by FMPQueryField objects. Field level operators are represented by FMPFieldOp. These are defined as follows:

```swift
/// An individual field search operator.
public enum FMPFieldOp {
	case equal
	case contains
	case beginsWith
	case endsWith
	case greaterThan
	case greaterThanEqual
	case lessThan
	case lessThanEqual
}

/// An individual query field.
public struct FMPQueryField {
	/// The name of the field.
	public let name: String
	/// The value for the field.
	public let value: Any
	/// The search operator.
	public let op: FMPFieldOp
	/// Initialize with a name, value and operator.
	public init(name: String, value: Any, op: FMPFieldOp = .beginsWith)
}
```

When performing an ```.edit``` action, FMPQueryFields can be added to a FMPQuery as an array through the ```FMPQuery.queryFields(_ queryFields: [FMPQueryField])``` function. Any field level operators are ignored in an ```.edit```.

When performing a ```.find``` query fields are grouped by logical operator. Logical operators indicate how the fields should be treated together in a query. The possible logical operators are: and, or, not. Their meanings, when performing a search and selecting records for the result set, are:

* and: All query fields in the group must match for the record to be selected.
* or: Any field in the group can match and the record will be selected.
* not: If all query fields in the group match then the record will be omitted from the result set.

Multiple query field groups can be added to a FMPQuery. Each group will be considered in-order when FileMaker performs the search.

Logical operators and query field groups are represented by FMPLogicalOp &amp; FMPQueryFieldGroup, respectively.

```swift
/// A logical operator used with query field groups.
public enum FMPLogicalOp {
	case and, or, not
}

/// A group of query fields.
public struct FMPQueryFieldGroup {
	/// The logical operator for the field group.
	public let op: FMPLogicalOp
	/// The list of fiedls in the group.
	public let fields: [FMPQueryField]
	/// Initialize with an operator and field list.
	/// The default logical operator is FMPLogicalOp.and.
	public init(fields: [FMPQueryField], op: FMPLogicalOp = .and)
}
```

Query field groups are added through the ```FMPQuery.queryFields(_ queryFields: [FMPQueryFieldGroup])``` function.

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

Perform a findAll and print all field names and values.

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
