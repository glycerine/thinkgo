thinkgo
=======

Here is my outline for those learning Go for the first time. This will get you up to speed quickly.

a. Upfront hints: When googling or searching stack overflow, search for "golang" instead of "go". The standard alias "golang" has been adopted, even in the official website [golang.org](http://golang.org).

The Go Tour introduces lots of features:

http://tour.golang.org

b. My favorite tutorial/book:
Learning Go by Miek Gieben

http://miek.nl/downloads/Go/Learning-Go-latest.pdf

112 pages. Short and sweet. With exercises.

c. Signup for updates on the latest things happening in golang land:

http://www.newspaper.io/golang

d. the main user group, worth reading for announcements of new releases. The place to ask and get answers as you are learning.

https://groups.google.com/forum/#!forum/Golang-nuts


e. How to learn the unique concurrency mechanism of Go: channels. Channels are based on CSP (Communicating Sequential Processes, introduced by Quicksort-inventor and Turing award winner Tony Hoare; http://en.wikipedia.org/wiki/Communicating_sequential_processes ):

  - http://www.slideshare.net/cloudflare/a-channel-compendium

In go, concurrency is treated just using three language elements: go routines, channels, and select statements.

  - Course by Rob Pike that predates Go 1.0 and is considered out of date, but still has
some of the best intro to concurrency (goroutines + channels) idioms:

http://go.googlecode.com/hg-history/release-branch.r60/doc/GoCourseDay1.pdf
http://go.googlecode.com/hg-history/release-branch.r60/doc/GoCourseDay2.pdf
http://go.googlecode.com/hg-history/release-branch.r60/doc/GoCourseDay3.pdf

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

Usally one sends pointers to structs on channels. If sending is considered a transfer of ownership, and there is only ever one owner, then no other synchronization is needed. The owner go-routine is the only one that can read or write from that structure.

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

Tips
----
To run all your tests in the *_test.go files: go test -v

To check for data races: go test -v -race

To kill and get a stack dump of your running/hung program: kill -QUIT <pid>

To generate debugging information (for use in gdb): first use go1.2.1 (not 1.3; they've worsened the debug info), then compile with these flags to turn off inlining and registerization: go build -gcflags "-N -l"


