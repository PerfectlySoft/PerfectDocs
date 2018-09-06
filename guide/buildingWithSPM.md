# Building with Swift Package Manager

[Swift Package Manager](https://swift.org/package-manager/) (SPM) is a command-line tool for building, testing, and managing dependencies for Swift projects. All of the components in Perfect are designed to build with SPM. If you create a project with Perfect, it will need to use SPM as well.

The best way to start a new Perfect project is to fork or clone the [PerfectTemplate](https://github.com/PerfectlySoft/PerfectTemplate). It will give you a very simple "Hello, World!" server message which you can edit and modify however you wish.

Before beginning, ensure you have read the [dependencies](https://github.com/PerfectlySoft/Perfect/wiki/Dependencies) document, and that you have a functioning Swift 4.0 toolchain for your platform.

The next step is to clone the template project. Open a new command-line terminal and change directory (cd) where you want the project to be cloned. Type the following into your terminal to download a directory called “PerfectTemplate”:

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
```

Within the Perfect Template, you will find two important file items:

- A *Sources* directory containing all of the Swift source files for Perfect
- An SPM manifest named “*Package.swift*” listing the dependencies this project requires


All SPM projects will have, at least, both a *Sources* directory and a *Package.swift* file. This project starts out with only one dependency: the [Perfect-HTTPServer](https://github.com/PerfectlySoft/Perfect-HTTPServer) project.

### Dependencies in Package.swift

The PerfectTemplate *Package.swift* manifest file contains the following content:

```swift
import PackageDescription

let package = Package(
	name: "PerfectTemplate",
	targets: [],
	dependencies: [
		.Package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git",
			majorVersion: 3)
    ]
)
```

**Note:** The version presented above may differ from what you have on your terminal. We recommend you consult the actual repository for the most up-to-date content.

There are two important elements in the *Package.swift* file that you may wish to edit.

The first one is the **name** element. It indicates the name of the project, and thus, the name of the executable file which will be generated when the project is built.

The second element is the **dependencies** list. This element indicates all of the subprojects that your application is dependent upon. Each item in this array consists of a “*.Package*” with a repository URL and a version.

The example above indicates a wide range of versions so that the template will always grab the newest revision of the HTTPServer project. You may want to restrict your dependencies to specific stable versions. For example, if you want to only build against version 2 of the Perfect HTTPServer project, your “*.Package*” element may look like the following:

```swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git", majorVersion: 3)
```

As your project grows and you add dependencies, you will put all of them in the **dependencies** list. SPM will automatically download the appropriate versions and compile them along with your project. All dependencies are downloaded into a *Packages* directory which SPM will automatically create. For example, if you wanted to use Mustache templates in your server, your *Package.swift* file might look like the following:

```swift
import PackageDescription

let package = Package(
	name: "PerfectTemplate",
	targets: [],
	dependencies: [
		.Package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git",
			majorVersion: 3),
		.Package(url: "https://github.com/PerfectlySoft/Perfect-Mustache.git",
			majorVersion: 3)
    ]
)
```

As you can see, the [Perfect-Mustache](https://github.com/PerfectlySoft/Perfect-Mustache) project was added as a dependency. It provides Mustache template support for your Perfect server. Within your project code, you can now import “PerfectMustache”, and use the facilities it offers.

As your dependency list grows, you may want to manage the list differently. The following example includes all the Perfect repositories. The list of URLs is maintained separately, and they are mapped to the format required by the **dependencies** parameter.

```swift
import PackageDescription

let versions = majorVersion: 3
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
	dependencies: urls.map { .Package(url: $0, versions) }
)
```

### Building

SPM provides the following commands for building your project, and for cleaning up any build artifacts:

```
swift build
```

This command will download any dependencies if they haven't been already acquired and attempt to build the project. If the build is successful, then the resulting executable will be placed in the (hidden) ```.build/debug/``` directory. When building the PerfectTemplate project, you will see as the last line of SPM output: ```Linking .build/debug/PerfectTemplate```. Entering ```.build/debug/PerfectTemplate``` will run the server. By default, a debug version of the executable will be generated. To build a production ready release version, you would issue the command ```swift build -c release```. This will place the resulting executable in the ```.build/release/``` directory.

```
swift package clean
```

It can be useful to wipe out all intermediate data and do a fresh build. Providing the ```--clean``` argument will delete the ```.build``` directory, and permit a fresh build. Providing the ```--clean=dist``` argument will delete both the ```.build``` directory and the ```Packages``` directory. Doing so will re-download all project dependencies during the next build to ensure you have the latest version of a dependent project.

### Xcode Projects

SPM can generate an Xcode project based on your *Package.swift* file. This project will permit you to build and debug your application within Xcode. To generate the Xcode project, issue the following command:

```
swift package generate-xcodeproj
```

The command will generate the Xcode project file into the same directory. For example, issuing the command within the PerfectTemplate project directory will produce the message `generated: ./PerfectTemplate.xcodeproj`.

**Note: It is not advised to edit or add files directly to this Xcode project.** If you add any further dependencies, or require later versions of any dependencies, you will need to regenerate this Xcode project. As a result, any modifications you have made will be overwritten.

### Additional Information

For more information on the Swift Package Manager, visit:

[https://swift.org/package-manager/](https://swift.org/package-manager/)
