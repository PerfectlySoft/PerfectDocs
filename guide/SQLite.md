# SQLite

The SQLite connector provides a wrapper around SQLite3, allowing interaction between your Perfect Applications &amp; SQLite databases. 

## System Requirements

### macOS

No additional configuration should be required, as SQlite3 is already built in. 

### Linux 
Make sure that you are running SQLite3:
`sudo apt-get install sqlite3`

## Setup

Add the "Perfect-SQLite" project as a dependency in your Package.swift file:

`.Package(url: "https://github.com/PerfectlySoft/Perfect-SQLite.git", versions: Version(0,0,0)..<Version(10,0,0))`

## Usage

### Import

Fist and foremost, in any of the source files you intend to use with SQLite, import the module with: 

`import SQLite`

### Access the Database

The database is accessed via it’s local file path, so the fist step is to store the file path to your sqlite data. 

`let dbPath = "./db/database"`

Once you’ve got that, you can open a connection to the database: 

```let dbPath = "./db/database"

do {
	let sqlite = try SQLite(dbPath)
	defer {
		sqlite.close() // This makes sure we close our connection regardless of the success or failure of this block. 
	}
} catch {
	//Handle Errors
}```