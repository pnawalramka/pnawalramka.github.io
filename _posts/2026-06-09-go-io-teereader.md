---
layout: post
title:  "Go: Tee it Together - io.TeeReader()"
date:   2026-06-09
---
# Go: Tee it Together - io.TeeReader()

In my [previous article](https://runningnotes.dev/2026/06/04/go-io-pipe.html), I talked about connecting `io.Reader` and `io.Writer` instances using `io.Pipe()`. Another handy construct in Go's io package is `io.TeeReader()`. Yes, it is conceptually similar to the unix `tee` command. This standard library function accepts an `io.Reader` (original source) and an `io.Writer` (intercepting destination), and returns a wrapper struct, that implements `io.Reader` interface. When a call to `Read(p []byte)` occurs, the bytes are copied to the given destiation buffer, and additionally cloned in real-time to the underlying intercepting writer. This is best explained with an example.


```go
package main

import (
    "crypto/sha256"
    "fmt"
    "io"
    "log"
    "strings"
)

func main() {
    const chunkSize = 8

    sourceVal := `Hello from this demo string.`
    srcRdr := strings.NewReader(sourceVal)

    hash := sha256.New()
    rdr := io.TeeReader(srcRdr, hash) // compute the digest of the bytes while reading it
    var totalBytesRead int
    buffer := make([]byte, chunkSize)
    for {
        bytesRead, err := rdr.Read(buffer)
        if err != nil && err != io.EOF {
            log.Fatalf("error reading from soure: %s\n", err)
        }
        if bytesRead == 0 {
            break
        }

        processChunk(buffer[:bytesRead])
        totalBytesRead += bytesRead
    }

    fmt.Printf("total bytes read: %d\n", totalBytesRead)
    fmt.Printf("Hashsum: %x\n", hash.Sum(nil))
}

func processChunk(b []byte) {
    // simulate processing data in chunks
    fmt.Printf("processing chunk: %s\n", string(b))
}
```

## Points to note
- The underlying read implementation does not explicitly buffer the bytes read.
- The read call blocks until all bytes are written to the intercepting destination. This may have a throttling effect on the original read logic if the writes are slow.


## Under the hood
The `Read()` implementation of the internal `teeReader` wrapper struct calls `Read()` on the original source to populate the given buffer. Additionally, the successfully read bytes will be written to the intermediate destination before the call returns. Any write errors are propagated up to the read caller. If you're asking: _Priyanka, could I do this myself?_ My answer: _Yes, absolutely._ `teeReader` is just a language provided syntactical sugar. In fact, the source code is under 20 lines of code.
