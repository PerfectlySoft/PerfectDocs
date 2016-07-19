# Building with Swift Package Manager
[Swift Package Manager](https://swift.org/package-manager/) (SPM) is a command line tool for building, testing and managing dependencies for Swift projects. All of the components in Perfect are designed to build with the SPM. If you create a project with Perfect it will need to use SPM as well.

The best way to start a new Perfect project is to fork or clone the [PerfectTemplate](https://github.com/PerfectlySoft/PerfectTemplate). This will give you a very simple Hello, World! server which you can then edit and add your own code to.

Before beginning, ensure you have read the [dependencies](https://github.com/PerfectlySoft/Perfect/wiki/Dependencies) document and that you have a functioning Swift 3.0 toolchain for your platform. The next step is to clone the template project. Open a new command line terminal and cd to the directory in which you want the project to be cloned. Enter the following:

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
```

This will download the project and give you a directory called *PerfectTemplate*. cd into this directory. Inside you will find two important items. First is a *Sources* directory. *Sources* contains all the Swift source files for the project. Second is the SPM manifest file named *Package.swift*. This file lists the dependencies the project requires. All SPM projects will have, at least, both a *Sources* directory and a *Package.swift* file. This project starts out with only one dependency; that for the [Perfect-HTTPServer](https://github.com/PerfectlySoft/Perfect-HTTPServer) project.

### Dependencies in Package.swift

The PerfectTemplate *Package.swift* manifest file has the following content:

```swift
import PackageDescription

let package = Package(
	name: "PerfectTemplate",
	targets: [],
	dependencies: [
		.Package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git", 
			versions: Version(0,0,0)..<Version(10,0,0))
    ]
)
```

Note that the **versions** information that you see here may differ. Consult the actual repository for the most up-to-date content.

There are two important elements in this *Package.swift* file which you may wish to edit. First is the **name** element. This indicates the name of the project and thus the name of the executable which will be generated when the project is built. The second element is the **dependencies** list. This element indicates all of the sub-projects which your application depends on. Each item in this array consists of a *.Package* containing a repository URL and a version. The example above indicates a wide range of versions so that the template will always grab the newest revision of the HTTPServer project. You may want to restrict your dependencies to specific stable versions. For example, if you wanted to only build against version 2 of the Perfect HTTPServer project, your *.Package* element might look like the following:

```swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git", majorVersion: 2)
```

As your project grows and you add dependencies you will put them all in the **dependencies** list and SPM will automatically download the appropriate versions and compile them along with your project. All dependencies are downloaded into a *Packages* directory which SPM will automatically create. To illustrate, if you wanted to use Mustache templates in your server, your *Package.swift* file might look like the following:

```swift
import PackageDescription

let package = Package(
	name: "PerfectTemplate",
	targets: [],
	dependencies: [
		.Package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git", 
			majorVersion: 2),
		.Package(url: "https://github.com/PerfectlySoft/Perfect-Mustache.git", 
			majorVersion: 2)
    ]
)
```

As you can see, the [Perfect-Mustache](https://github.com/PerfectlySoft/Perfect-Mustache) project was added as a dependency. This enables you to, within your project code, import PerfectMustache and use the facilities provided by that project.

As your dependency list grows you may want to manage the list a little differently. The following, somewhat contrived, example includes all of the repositories in the Perfect project. The list of URLs is maintained seperately and mapped to the format expected by the **dependencies** parameter.

```swift
import PackageDescription

let versions = Version(0,0,0)..<Version(10,0,0)
let urls = [
	"https://github.com/PerfectlySoft/Perfect-HTTPServer.git",
	"https://github.com/PerfectlySoft/Perfect-FastCGI.git",
	"https://github.com/PerfectlySoft/Perfect-CURL.git",
	"https://github.com/PerfectlySoft/Perfect-PostgreSQL.git",
	"https://github.com/PerfectlySoft/Perfect-SQLite.git",
	"https://github.com/PerfectlySoft/Perfect-Redis.git",
	"https://github.com/PerfectlySoft/Perfect-MySQL.git",
	"https://github.com/PerfectlySoft/Perfect-MongoDB.git",
	"https://github.com/PerfectlySoft/Perfect-WebSockets.git",
	"https://github.com/PerfectlySoft/Perfect-Notifications.git",
	"https://github.com/PerfectlySoft/Perfect-Mustache.git"
]

let package = Package(
	name: "PerfectTemplate",
	targets: [],
	dependencies: urls.map { .Package(url: $0, versions: versions) }
)
```

### Building

Swift Package Manager provides the following commands for building your project and for cleaning up any build artifacts:

```swift build``` 

This command will download any dependencies if they haven't been already acquired and attempt to build the project. If the build is successful then the resulting executable will be placed in the (hidden) ```.build/debug/``` directory. When building the PerfectTemplate project you will see as the last line of SPM output: ```Linking .build/debug/PerfectTemplate```. Entering ```.build/debug/PerfectTemplate``` will run the server. By default, a debug verison of the executable will be generated. To build a production ready release version, you would issue the command ```swift build -c release```. This will place the resulting executable in the ```.build/release/``` directory.

```swift build --clean```

```swift build --clean=dist```

It can often be useful to wipe out all intermediate data and do a fresh build. Providing the ```--clean``` argument will delete the ```.build``` directory and permit a fresh build. Providing the ```--clean=dist``` argument will delete both the ```.build``` directory and the ```Packages``` directory. After doing this building will re-download all project dependencies. This can often be useful to ensure you have the latest version of a dependent project.

### Xcode Project

Swift Package Manager can generate an Xcode project based on your *Package.swift* file. This project will permit you to build and debug your application within Xcode. To generate the Xcode project, issue the following command:

```swift package generate-xcodeproj```

The command will generate the Xcode project file into the same directory. For example, issuing the command within the PerfectTemplate project directory will produce the message ```generated: ./PerfectTemplate.xcodeproj```.

It is not advised to edit or add files directly to this Xcode project. The reason for this is that if you add any further dependencies or require later versions of any dependencies then you will need to regenrate this Xcode project and any modifications you have made will be overwritten.

### Additional Information

For more information on the Swift Package Manager, consult the SPM site itself:

[https://swift.org/package-manager/](https://swift.org/package-manager/)


