[![Go Reference](https://pkg.go.dev/badge/github.com/bitfield/gotestdox.svg)](https://pkg.go.dev/github.com/sreyardship/testclarity)
[![Go Report Card](https://goreportcard.com/badge/github.com/sreyardship/testclarity)](https://goreportcard.com/report/github.com/sreyardship/testclarity)

![This is the way](img/testclarity.png)

`testclarity` is a command-line tool for giving clarity to your go tests. Here's how to install it:

```
go install github.com/sreyardship/testclarity/cmd/testclarity@latest
```
All credits should go to https://github.com/bitfield/gotestdox from which this project is forked.

# Why fork?
`gotestdox` is very focused on the "documentation" part of the equation. I.e. taking your test name and generate documenation out of those. We believe this should be taken a step further and into your daily development work. By constantly showing you the resulting documenation of your test names you are able to faster realize when you've designed your test bad, normally that stems from such a simple thing as the name you've given to the test.

## What extra features are there then?
Today there's only one flag `-v` which makes `testclarity` verbose and prints the output of tests with filename and numbers. But features in line with the vision for this tooling will be integrated and developed going forward, in lieu of time.


# Links
- [Test names should be sentences](https://bitfieldconsulting.com/golang/test-names)
- [The Power of Go: Tests](https://bitfieldconsulting.com/books/tests)

<small>Gopher image by [MariaLetta](https://github.com/MariaLetta/free-gophers-pack)</small>
