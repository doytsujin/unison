# tests for built-in IO functions

Tests for IO builtins which wired to foreign haskell calls.

## Setup

You can skip the section which is just needed to make the transcript self-contained.

TempDirs/autoCleaned is an ability/hanlder which allows you to easily
create a scratch directory which will automatically get cleaned up.

## Basic File Functions

### Creating/Deleting/Renaming Directories

Tests: createDirectory,
       isDirectory,
       fileExists,
       renameDirectory,
       deleteDirectory

```unison
testCreateRename : '{io2.IO} [Result]
testCreateRename _ =
  test = 'let
    tempDir = newTempDir "fileio"
    fooDir = tempDir ++ "/foo"
    barDir = tempDir ++ "/bar"
    createDirectory.impl fooDir
    check "create a foo directory" (isDirectory fooDir)
    check "directory should exist" (fileExists fooDir)
    renameDirectory fooDir barDir
    check "foo should no longer exist" (not (fileExists fooDir))
    check "directory should no longer exist" (not (fileExists fooDir))
    check "bar should now exist" (fileExists barDir)

    bazDir = barDir ++ "/baz"
    createDirectory.impl bazDir
    removeDirectory.impl barDir

    check "removeDirectory works recursively" (not (isDirectory barDir))
    check "removeDirectory works recursively" (not (isDirectory bazDir))

  runTest test
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      testCreateRename : '{IO} [Result]

```
```ucm
.> add

  ⍟ I've added these definitions:
  
    testCreateRename : '{IO} [Result]

.> io.test testCreateRename

    New test results:
  
  ◉ testCreateRename   create a foo directory
  ◉ testCreateRename   directory should exist
  ◉ testCreateRename   foo should no longer exist
  ◉ testCreateRename   directory should no longer exist
  ◉ testCreateRename   bar should now exist
  ◉ testCreateRename   removeDirectory works recursively
  ◉ testCreateRename   removeDirectory works recursively
  
  ✅ 7 test(s) passing
  
  Tip: Use view testCreateRename to view the source of a test.

```
### Opening / Closing files

Tests: openFile
       closeFile
       isFileOpen

```unison
testOpenClose : '{io2.IO} [Result]
testOpenClose _ =
  test = 'let
    tempDir = (newTempDir "seek")
    fooFile = tempDir ++ "/foo"
    handle1 = openFile fooFile FileMode.Write
    check "file should be open" (isFileOpen handle1)
    setBuffering handle1 (SizedBlockBuffering 1024)
    check "file handle buffering should match what we just set." (getBuffering handle1 == SizedBlockBuffering 1024)
    setBuffering handle1 (getBuffering handle1)
    putBytes handle1 0xs01
    setBuffering handle1 NoBuffering
    setBuffering handle1 (getBuffering handle1)
    putBytes handle1 0xs23
    setBuffering handle1 BlockBuffering
    setBuffering handle1 (getBuffering handle1)
    putBytes handle1 0xs45
    setBuffering handle1 LineBuffering
    setBuffering handle1 (getBuffering handle1)
    putBytes handle1 0xs67
    closeFile handle1
    check "file should be closed" (not (isFileOpen handle1))

    -- make sure the bytes have been written
    handle2 = openFile fooFile FileMode.Read
    check "bytes have been written" (getBytes handle2 4 == 0xs01234567)
    closeFile handle2

    -- checking that ReadWrite mode works fine
    handle3 = openFile fooFile FileMode.ReadWrite
    check "bytes have been written" (getBytes handle3 4 == 0xs01234567)
    closeFile handle3

    check "file should be closed" (not (isFileOpen handle1))

  runTest test
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      testOpenClose : '{IO} [Result]

```
```ucm
.> add

  ⍟ I've added these definitions:
  
    testOpenClose : '{IO} [Result]

.> io.test testOpenClose

    New test results:
  
  ◉ testOpenClose   file should be open
  ◉ testOpenClose   file handle buffering should match what we just set.
  ◉ testOpenClose   file should be closed
  ◉ testOpenClose   bytes have been written
  ◉ testOpenClose   bytes have been written
  ◉ testOpenClose   file should be closed
  
  ✅ 6 test(s) passing
  
  Tip: Use view testOpenClose to view the source of a test.

```
### Reading files with getSomeBytes

Tests: getSomeBytes
       putBytes
       isFileOpen
       seekHandle

```unison
testGetSomeBytes : '{io2.IO} [Result]
testGetSomeBytes _ =
  test = 'let
    tempDir = (newTempDir "getSomeBytes")
    fooFile = tempDir ++ "/foo"

    testData = "0123456789"
    testSize = size testData

    chunkSize = 7
    check "chunk size splits data into 2 uneven sides" ((chunkSize > (testSize / 2)) && (chunkSize < testSize))


    -- write testData to a temporary file
    fooWrite = openFile fooFile Write
    putBytes fooWrite (toUtf8 testData)
    closeFile fooWrite
    check "file should be closed" (not (isFileOpen fooWrite))

    -- reopen for reading back the data in chunks
    fooRead = openFile fooFile Read

    -- read first part of file
    chunk1 = getSomeBytes fooRead chunkSize |> fromUtf8
    check "first chunk matches first part of testData" (chunk1 == take chunkSize testData)

    -- read rest of file
    chunk2 = getSomeBytes fooRead chunkSize |> fromUtf8
    check "second chunk matches rest of testData" (chunk2 == drop chunkSize testData)

    check "should be at end of file" (isFileEOF fooRead)

    readAtEOF = getSomeBytes fooRead chunkSize
    check "reading at end of file results in Bytes.empty" (readAtEOF == Bytes.empty)

    -- request many bytes from the start of the file
    seekHandle fooRead AbsoluteSeek +0
    bigRead = getSomeBytes fooRead (testSize * 999) |> fromUtf8
    check "requesting many bytes results in what's available" (bigRead == testData)

    closeFile fooRead
    check "file should be closed" (not (isFileOpen fooRead))

  runTest test
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      testGetSomeBytes : '{IO} [Result]

```
```ucm
.> add

  ⍟ I've added these definitions:
  
    testGetSomeBytes : '{IO} [Result]

.> io.test testGetSomeBytes

    New test results:
  
  ◉ testGetSomeBytes   chunk size splits data into 2 uneven sides
  ◉ testGetSomeBytes   file should be closed
  ◉ testGetSomeBytes   first chunk matches first part of testData
  ◉ testGetSomeBytes   second chunk matches rest of testData
  ◉ testGetSomeBytes   should be at end of file
  ◉ testGetSomeBytes   reading at end of file results in Bytes.empty
  ◉ testGetSomeBytes   requesting many bytes results in what's available
  ◉ testGetSomeBytes   file should be closed
  
  ✅ 8 test(s) passing
  
  Tip: Use view testGetSomeBytes to view the source of a test.

```
### Seeking in open files

Tests: openFile
       putBytes
       closeFile
       isSeekable
       isFileEOF
       seekHandle
       getBytes
       getLine

```unison
testSeek : '{io2.IO} [Result]
testSeek _ =
  test = 'let
    tempDir = newTempDir "seek"
    emit (Ok "seeked")
    fooFile = tempDir ++ "/foo"
    handle1 = openFile fooFile FileMode.Append
    putBytes handle1 (toUtf8 "12345678")
    closeFile handle1

    handle3 = openFile fooFile FileMode.Read
    check "readable file should be seekable" (isSeekable handle3)
    check "shouldn't be the EOF" (not (isFileEOF handle3))
    expectU "we should be at position 0" 0 (handlePosition handle3)

    seekHandle handle3 AbsoluteSeek +1
    expectU "we should be at position 1" 1 (handlePosition handle3)
    bytes3a = getBytes handle3 1000
    text3a = Text.fromUtf8 bytes3a
    expectU "should be able to read our temporary file after seeking" "2345678" text3a
    closeFile handle3

    barFile = tempDir ++ "/bar"
    handle4 = openFile barFile FileMode.Append
    putBytes handle4 (toUtf8 "foobar\n")
    closeFile handle4

    handle5 = openFile barFile FileMode.Read
    expectU "getLine should get a line" "foobar" (getLine handle5)
    closeFile handle5

  runTest test

testAppend : '{io2.IO} [Result]
testAppend _ =
  test = 'let
    tempDir = newTempDir "openFile"
    fooFile = tempDir ++ "/foo"
    handle1 = openFile fooFile FileMode.Write
    putBytes handle1 (toUtf8 "test1")
    closeFile handle1

    handle2 = openFile fooFile FileMode.Append
    putBytes handle2 (toUtf8 "test2")
    closeFile handle2

    handle3 = openFile fooFile FileMode.Read
    bytes3 = getBytes handle3 1000
    text3 = Text.fromUtf8 bytes3

    expectU "should be able to read our temporary file" "test1test2" text3

    closeFile handle3

  runTest test
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      testAppend : '{IO} [Result]
      testSeek   : '{IO} [Result]

```
```ucm
.> add

  ⍟ I've added these definitions:
  
    testAppend : '{IO} [Result]
    testSeek   : '{IO} [Result]

.> io.test testSeek

    New test results:
  
  ◉ testSeek   seeked
  ◉ testSeek   readable file should be seekable
  ◉ testSeek   shouldn't be the EOF
  ◉ testSeek   we should be at position 0
  ◉ testSeek   we should be at position 1
  ◉ testSeek   should be able to read our temporary file after seeking
  ◉ testSeek   getLine should get a line
  
  ✅ 7 test(s) passing
  
  Tip: Use view testSeek to view the source of a test.

.> io.test testAppend

    New test results:
  
  ◉ testAppend   should be able to read our temporary file
  
  ✅ 1 test(s) passing
  
  Tip: Use view testAppend to view the source of a test.

```
### SystemTime
```unison
testSystemTime : '{io2.IO} [Result]
testSystemTime _ =
  test = 'let
    t = !systemTime
    check "systemTime should be sane" ((t > 1600000000) && (t < 2000000000))

  runTest test
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      testSystemTime : '{IO} [Result]

```
```ucm
.> add

  ⍟ I've added these definitions:
  
    testSystemTime : '{IO} [Result]

.> io.test testSystemTime

    New test results:
  
  ◉ testSystemTime   systemTime should be sane
  
  ✅ 1 test(s) passing
  
  Tip: Use view testSystemTime to view the source of a test.

```
### Get temp directory

```unison
testGetTempDirectory : '{io2.IO} [Result]
testGetTempDirectory _ =
  test = 'let
    tempDir = reraise !getTempDirectory.impl
    check "Temp directory is directory" (isDirectory tempDir)
    check "Temp directory should exist" (fileExists tempDir)
  runTest test
```

```ucm
.> add

  ⍟ I've added these definitions:
  
    testGetTempDirectory : '{IO} [Result]

.> io.test testGetTempDirectory

    New test results:
  
  ◉ testGetTempDirectory   Temp directory is directory
  ◉ testGetTempDirectory   Temp directory should exist
  
  ✅ 2 test(s) passing
  
  Tip: Use view testGetTempDirectory to view the source of a
       test.

```
### Get current directory

```unison
testGetCurrentDirectory : '{io2.IO} [Result]
testGetCurrentDirectory _ =
  test = 'let
    currentDir = reraise !getCurrentDirectory.impl
    check "Current directory is directory" (isDirectory currentDir)
    check "Current directory should exist" (fileExists currentDir)
  runTest test
```

```ucm
.> add

  ⍟ I've added these definitions:
  
    testGetCurrentDirectory : '{IO} [Result]

.> io.test testGetCurrentDirectory

    New test results:
  
  ◉ testGetCurrentDirectory   Current directory is directory
  ◉ testGetCurrentDirectory   Current directory should exist
  
  ✅ 2 test(s) passing
  
  Tip: Use view testGetCurrentDirectory to view the source of a
       test.

```
### Get directory contents

```unison
testDirContents : '{io2.IO} [Result]
testDirContents _ =
  test = 'let
    tempDir = newTempDir "dircontents"
    c = reraise (directoryContents.impl tempDir)
    check "directory size should be"  (size c == 2)
    check "directory contents should have current directory and parent" let
      (c == [".", ".."]) || (c == ["..", "."])
  runTest test
```

```ucm
.> add

  ⍟ I've added these definitions:
  
    testDirContents : '{IO} [Result]

.> io.test testDirContents

    New test results:
  
  ◉ testDirContents   directory size should be
  ◉ testDirContents   directory contents should have current directory and parent
  
  ✅ 2 test(s) passing
  
  Tip: Use view testDirContents to view the source of a test.

```
### Read environment variables

```unison
testGetEnv : '{io2.IO} [Result]
testGetEnv _ =
  test = 'let
    path = reraise (getEnv.impl "PATH") -- PATH exists on windows, mac and linux.
    check "PATH environent variable should be set"  (size path > 0)
    match getEnv.impl "DOESNTEXIST" with 
      Right _ -> emit (Fail "env var shouldn't exist")
      Left _ -> emit (Ok "DOESNTEXIST didn't exist")
  runTest test
```

```ucm
.> add

  ⍟ I've added these definitions:
  
    testGetEnv : '{IO} [Result]

.> io.test testGetEnv

    New test results:
  
  ◉ testGetEnv   PATH environent variable should be set
  ◉ testGetEnv   DOESNTEXIST didn't exist
  
  ✅ 2 test(s) passing
  
  Tip: Use view testGetEnv to view the source of a test.

```
### Read command line args

`runMeWithNoArgs`, `runMeWithOneArg`, and `runMeWithTwoArgs` raise exceptions 
unless they called with the right number of arguments.

```unison
testGetArgs.fail : Text -> Failure
testGetArgs.fail descr = Failure (typeLink IOFailure) descr !Any

testGetArgs.runMeWithNoArgs : '{io2.IO, Exception} ()
testGetArgs.runMeWithNoArgs = 'let
  args = reraise !getArgs.impl
  match args with
    [] -> printLine "called with no args"
    _ -> raise (fail "called with args")

testGetArgs.runMeWithOneArg : '{io2.IO, Exception} ()
testGetArgs.runMeWithOneArg = 'let
  args = reraise !getArgs.impl
  match args with
    [] -> raise (fail "called with no args")
    [_] -> printLine "called with one arg"
    _ -> raise (fail "called with too many args")

testGetArgs.runMeWithTwoArgs : '{io2.IO, Exception} ()
testGetArgs.runMeWithTwoArgs = 'let
  args = reraise !getArgs.impl
  match args with
    [] -> raise (fail "called with no args")
    [_] -> raise (fail "called with one arg")
    [_, _] -> printLine "called with two args"
    _ -> raise (fail "called with too many args")
```

Test that they can be run with the right number of args.
```ucm
.> add

  ⍟ I've added these definitions:
  
    testGetArgs.fail             : Text -> Failure
    testGetArgs.runMeWithNoArgs  : '{IO, Exception} ()
    testGetArgs.runMeWithOneArg  : '{IO, Exception} ()
    testGetArgs.runMeWithTwoArgs : '{IO, Exception} ()

.> cd testGetArgs

.> run runMeWithNoArgs

.> run runMeWithOneArg foo

.> run runMeWithTwoArgs foo bar

```
Calling our examples with the wrong number of args will error.

```ucm
.> run runMeWithNoArgs foo

  💔💥
  
  The program halted with an unhandled exception:
  
    Failure (typeLink IOFailure) "called with args" (Any ())

```
```ucm
.> run runMeWithOneArg

  💔💥
  
  The program halted with an unhandled exception:
  
    Failure (typeLink IOFailure) "called with no args" (Any ())

```
```ucm
.> run runMeWithOneArg foo bar

  💔💥
  
  The program halted with an unhandled exception:
  
    Failure
      (typeLink IOFailure) "called with too many args" (Any ())

```
```ucm
.> run runMeWithTwoArgs

  💔💥
  
  The program halted with an unhandled exception:
  
    Failure (typeLink IOFailure) "called with no args" (Any ())

```
