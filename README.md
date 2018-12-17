# Let's write a swift script in an hour

* Xcode 10.1 / Swift 4.2 (Development Environment)

Nowadays we are moving away from keeping records on papers and we are facing new challenges/opportunities to analyze data in bulk. Most fields like sciences, finance, accounting, engeneering, etc. have the challenge of cobining all thir data to have a higher level analysis. Commonly data is stored in excel files or a csv files so it's easily consumed, but what if this data is on different files? We will end up doing lots of repetitive work. How about we dont do any tedious work? Lets write a script that we can use over and over, or hand in to our fellow coworkers as a tool to use to make sense of their data. 

# Full procedure

## 1, Create A Playground File

* Open Xcode  
* Select <code>Get started with a playground</code> or <code>File</code> -> <code>New</code> -> <code>Playground</code>
* Select <code>Blank</code> and then tap <code>Next</code>  
* Rename "name_of_the_file"
* Select the place for saving your project and then tap <code>Create</code>  

## 2, Import Assets

We need 1 folder. Please download <code>scvdata</code> (https://github.com/iosClassForBeginner/scripting_en) from the link. You can clone the whole project and just use this folder. 

  * <details><summary>You don't know how to import them? Check this out!</summary><div style="text-align:center"><img src ="https://github.com/iosClassForBeginner/quiz_en/blob/master/demos/tutorial/assets.gif" /></div></details>

## 3, Massaging the data

Rename all the files in the "csvdata" folder to be consumed by our script. Rename them "File1" ... "File12"

## 4, Coding the script

Dive straight into coding the script.
#Our work flow would be:
* 4.1. Create URL paths to our folder
* 4.2. Read the files and extract their data one by one.
* 4.3. Do something with extracted information
* 4.4. Write to file the information

```Swift

import Foundation  // Needed for those pasting into Playground

let readWriteFolderName = "csvdata/"

// Optional file name thats consistent between all files.
let prefix = "File"
let fileType = "csv"

// Can fixate the file number argument to second argument too, for consistency
let numberArgument = CommandLine.arguments.last

var foodExpenses: [Int] = []
var entertainmentExpenses: [Int] = []
var homeExpenses: [Int] = []
var transportationExpenses: [Int] = []
var destinationDirectory: URL?

// Look up in the Desktop directory
if var dir = try? FileManager.default.url(for: .desktopDirectory,
                                          in: .userDomainMask, appropriateFor: nil, create: true) {
    
    // If we are looking inside another directory on Desktop then add the path here.
    dir.appendPathComponent(readWriteFolderName)
    
    // Save the directory path to save the results in the same place. Feel free to change it to anywhere else
    destinationDirectory = dir
    
    // The number of files to process. Comes from the last argument passed in the command line tool.
    // Stop extracting if its not provided and is not more than 1.
    if let numberOfFiles = Int(numberArgument ?? ""), numberOfFiles > 1 {
        
        for i in 1...numberOfFiles {
            let fileName = prefix + String(i)
            let fileURL = dir.appendingPathComponent(fileName).appendingPathExtension(fileType)
            
            // Read the contents of the file
            var inString = ""
            
            do {
                inString = try String(contentsOf: fileURL)
                
                if !inString.isEmpty {
                    // Each line of csv file ends with '\n'. A good marker to extract each line as an element in an array.
                    var lines = inString.components(separatedBy: "\n")
                    lines.removeFirst()
                    
                    lines.forEach {
                        // All inputs in a comma seperated valu (csv) file are seperated by ','
                        // A good marker to extract each value as an element in an array.
                        var columns = $0.components(separatedBy: ",")
                        columns.removeFirst()
                        
                        // Values are stored as a string in a csv file so we need to convert them to Int (or Double)
                        // in order to be able to do math with them at some point.
                        let values = columns.map { text -> Int in
                            let value = String(text.filter { !" \n\t\r".contains($0) })
                            let integer = Int(value)
                            return integer ?? 0
                        }
                        
                        // Add values of each column into they own arrays so we can later add them together.
                        for (index, value) in values.enumerated() {
                            switch index {
                            case 0:
                                foodExpenses.append(value)
                            case 1:
                                entertainmentExpenses.append(value)
                            case 2:
                                homeExpenses.append(value)
                            case 3:
                                transportationExpenses.append(value)
                            default:
                                // Not expecting this value
                                break
                            }
                        }
                    }
                }
            } catch {
                print("Failed reading from URL: \(fileURL), Error: " + error.localizedDescription)
            }
        }
    }
}


// Write to the file named Test
let titles = "Food,Entertainment,Home,Transportation,Total"
var content = titles + "\n"
content += String(foodExpenses.reduce(0, { $0 + $1 })) + ","
content += String(entertainmentExpenses.reduce(0, { $0 + $1 })) + ","
content += String(homeExpenses.reduce(0, { $0 + $1 })) + ","
content += String(transportationExpenses.reduce(0, { $0 + $1 })) + ","
content += "=A2+B2+C2+D2"

// If the directory was found, we write a file to it. 
if let dir = destinationDirectory {
    
    do {
        let fileName = "AllExpenses"
        let fileURL = dir.appendingPathComponent(fileName).appendingPathExtension(fileType)
        try content.write(to: fileURL, atomically: true, encoding: .utf8)
    } catch {
        print("Failed writing to URL: \(dir), Error: " + error.localizedDescription)
    }
} else {
    print("Can't find saving directory URL: \(String(describing: destinationDirectory))")
}

```

## 4, Create an exceutable file

To create an executable file
```
$ xcrun swiftc csv_creator.swift
```

To run the executable file
```
$ ./csv_creator 10
```
