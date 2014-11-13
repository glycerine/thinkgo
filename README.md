thinkgo
=======

Here is my outline for those learning Go. This will get you up to speed quickly.

The [Go standard library](http://golang.org/pkg/) is a gem, and much, much more comprehensive than other standard libraries. In particular [net/http library](http://golang.org/pkg/net/http) is a full-on web serving framework, batteries included.

a. Upfront hints: When googling or searching stack overflow, search for "golang" instead of "go". The standard alias "golang" has been adopted, even in the official website [golang.org](http://golang.org).

The Go Tour introduces lots of features:

http://tour.golang.org

b. My favorite tutorial/book:
Learning Go by Miek Gieben

http://miek.nl/downloads/Go/Learning-Go-latest.pdf <sub><sup>[backup](vendor/Learning-Go-latest.pdf)</sup></sub>

112 pages. Short and sweet. With exercises.

c. Signup for updates on the latest things happening in golang land:

http://www.newspaper.io/golang

d. the main user group, worth reading for announcements of new releases. The place to ask and get answers as you are learning.

https://groups.google.com/forum/#!forum/Golang-nuts


e. How to learn the unique concurrency mechanism of Go: channels. Channels are based on CSP (Communicating Sequential Processes, introduced by Quicksort-inventor and Turing award winner Tony Hoare; http://en.wikipedia.org/wiki/Communicating_sequential_processes ):

  - http://www.slideshare.net/cloudflare/a-channel-compendium <sup><sub>[backup](vendor/John_Graham-Cumming_A_Channel_Compendium.pdf)</sup></sub>

In go, concurrency is treated just using three language elements: go routines, channels, and select statements.

  - Course by Rob Pike that predates Go 1.0 and is considered out of date, but still has
some of the best intro to concurrency (goroutines + channels) idioms:

http://go.googlecode.com/hg-history/release-branch.r60/doc/GoCourseDay1.pdf <sup><sub>[backup](vendor/GoCourseDay1.pdf)</sup></sub>
http://go.googlecode.com/hg-history/release-branch.r60/doc/GoCourseDay2.pdf <sup><sub>[backup](vendor/GoCourseDay2.pdf)</sup></sub>
http://go.googlecode.com/hg-history/release-branch.r60/doc/GoCourseDay3.pdf <sup><sub>[backup](vendor/GoCourseDay3.pdf)</sup></sub>

  - Andrew Gerrand's keynote at GopherCon2014 in Denver back in April:

http://talks.golang.org/2014/go4gophers.slide
videos: http://blog.golang.org/gophercon

  - Read actual code in a real project: https://github.com/glycerine/goq

Here I demonstrate channel techniques, like conditional send, that aren't readily found or discussed elsewhere. The main receive loop/state machine is in goq.go, in the JobServ::Start() method.

f. My favorite BDD (behavior-driven development)/test framework is called GoConvey. I've also used Ginkgo, but it is a bit over-engineered. GoConvey is smaller, simpler, and elegant. Just like Go itself. You can see tons of examples of GoConvey in the [Goq project mentioned above.](https://github.com/glycerine/goq)

https://github.com/smartystreets/goconvey


Channel General advice
-----------------

One channel is best used for one direction of communication. Channels are therefore typically deployed in pairs, one channel for sending a request, and another for replying.

Usually one sends pointers to structs on channels. If sending is considered a transfer of ownership, and there is only ever one owner, then no other synchronization is needed. The owning go-routine should be the only one that ever reads or write from that structure. 

The ownership passing motif described above is merely a pattern that Go enables, it isn't enforced by the language. Ownership passing through channels makes it easy to reason about the code, and remains performant. Classical mutex are available, but are likely to create performance bottlenecks and are prone to deadlock; they should be avoided until after the channel idioms are mastered, and even then used rarely.

The main use of goroutines and channels is to set up communicating state machines. The typical pattern for one such state machine has the go/for/select idiom that looks like this:

~~~
func Machine() {
  go func() {
    for {
      select { 
         case <-Channel0:
         ...
         case <-Channel1:
         ...
      }
    }
  }()
}
~~~

Channel Lifecycle
--------------

To be effective in Go, you need to memorize the channel lifecycle. My suggestion is to write short little five line go programs to demonstrate to yourself each cell in the grid below.

This is important for understanding how
to use nil channels in a select statement, and why closing a channel is broadcasting a message
that can be received at any point in time later. 

In the channel lifecycle table below, we assume an unbuffered channel. As an important exercise, you should construct the analagous chart for a buffered channel.

| unbuffered channel | | | |
|--------|---------|---------|------------|
| *state\action:*  | *send on* | *receive on* | *close* |
| nil    | blocks forever | blocks forever     | panic |  |
| made (open)   | blocks until receive  | blocks until send | ok |
| closed | panic  | immediately returns the zero value | panic  |


Tips
----
To run all your tests in the *_test.go files: `go test -v`

To check for data races: `go test -v -race`

To kill and get a stack dump of your running/hung program: `kill -QUIT <pid>`

To generate debugging information (for use in gdb): first use go1.2.1 (not 1.3; they've worsened the debug info), then compile with these flags to turn off inlining and registerization: `go build -gcflags "-N -l"`

You should use [goimports](https://github.com/bradfitz/goimports) to automatically adjust the imports at the top of your source file. My `.emacs` setup:

~~~
;;;; goimport to fix imports automagically
(setq gofmt-command "goimports")
(add-to-list 'load-path "/home/jaten/go1.3.1/go/misc/emacs")
(require 'go-mode-load)
(add-hook 'before-save-hook 'gofmt-before-save)
~~~

To install goimports, at the shell do: `go get github.com/bradfitz/goimports`
