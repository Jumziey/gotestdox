[![Go Reference](https://pkg.go.dev/badge/github.com/bitfield/gotestdox.svg)](https://pkg.go.dev/github.com/bitfield/gotestdox)[![Go Report Card](https://goreportcard.com/badge/github.com/bitfield/gotestdox)](https://goreportcard.com/report/github.com/bitfield/gotestdox)[![CircleCI](https://circleci.com/gh/bitfield/gotestdox.svg?style=svg)](https://circleci.com/gh/bitfield/gotestdox)

```
go install github.com/bitfield/gotestdox/cmd/gotestdox@latest
```

![Writing gopher logo](img/gotestdox.png)

# `gotestdox`

`gotestdox` is a command-line tool for turning Go test names into readable sentences. For example, suppose we have some tests named like this:

```
TestRelevantIsTrueForTestPassOrFailEvents
TestRelevantIsFalseForOtherEvents
```

We can transform them into straightforward sentences that express the desired behaviour, by running `gotestdox`:

**`gotestdox`**

This will run the tests, and print:

```
 ✔ Relevant is true for test pass or fail events (0.00s)
 ✔ Relevant is false for other events (0.00s)
```

# Why

I got the idea from a blog post by Dan North, which says:

> My first “Aha!” moment occurred as I was being shown a deceptively simple utility called `agiledox`, written by my colleague, Chris Stevenson. It takes a JUnit test class and prints out the method names as plain sentences.
>
> The word “test” is stripped from both the class name and the method names, and the camel-case method name is converted into regular text. That’s all it does, but its effect is amazing.
>
> Developers discovered it could do at least some of their documentation for them, so they started to write test methods that were real sentences.\
—Dan North, [Introducing BDD](https://dannorth.net/introducing-bdd/)

As he says, a very simple idea, but it has a powerful effect. As soon as you see what your test names look like as sentences, you immediately know what's wrong with them and how to fix it! And this in turn helps generate missing tests, for behaviours that you hadn't articulated until now.

For example, suppose we have some function `Match` that tells you whether or not a given input matches the string you're looking for:

```go
func Match(input, substring string) bool {
```

What would we name a test for this function? Maybe something like this:

```
TestMatch
```

Pretty standard, and no doubt it does test `Match` in some way, but *what* way? How is `Match` supposed to behave? Under what circumstances? Given what input? We don't know. We need to switch from thinking about the test name as a piece of useless paperwork, and start thinking about it as documentation.

We can write the test name as a sentence that expresses the *behaviour* of `Match` that we want. We might start with:

```
TestMatchIsTrueForMatchingInput
```

Great! But it immediately prompts us to think, "well, what about *non*-matching input?" Okay. That's another test, then:

```
TestMatchIsFalseForNonMatchingInput
```

And that's it! Now we have two sentences that completely describe the important behaviour of `Match`, and `gotestdox` will format them nicely for us:

```
 ✔ Match is true for matching input (0.00s)
 ✔ Match is false for non matching input (0.00s)
 ```

 As you accumulate more tests over time, your `gotestdox` output will be a more and more valuable user manual for your package. In fact, it's better than a user manual, because the moment one of those sentences stops being true, your tests will alert you to the fact!

# How

The original [`testdox`](https://github.com/astubbs/testdox) tool (part of `agiledox`) was very simple, as Dan describes: it just turned a camel-case JUnit test name like `testFailsForDuplicateCustomers` into a space-separated sentence like `fails for duplicate customers`.

And that's what I find neat about it: it's so simple that it hardly seems like it could be of any value, but it is. I've already used the idea to improve a lot of my test names.

There are implementations of `testdox` for various languages other than Java: for example, [PHP](https://phpunit.readthedocs.io/en/9.5/textui.html#testdox), [Python](https://pypi.org/project/pytest-testdox/), and [.NET](https://testdox.wordpress.com/). I haven't found one for Go, so here it is.

`gotestdox` reads the JSON output generated by the `go test -json` command. This is easier than trying to parse Go source code, for example, and also gives us pass/fail information for the tests. It ignores all events except pass/fail events for individual tests (including subtests).

When it finds a relevant event, it extracts the test name and transforms it into a sentence according to the following rules:

* Words are split on camel case and lowercased, except for the first word: `FooDoesBar` -> `Foo does bar`
* Words are also split on underscores: `Foo_does_Bar` -> `Foo does bar`
* All-caps words are preserved as is: `FooReadsPDF` -> `Foo reads PDF`
* The slashes separating test names from subtest names are removed: `Foo/does_bar` -> `Foo does bar`

Because Go function names are camel case (`HandleFoo`) and test names are also conventionally written with camel case (`HandleFooDoesBar`), we have a slight problem with the rendering of multi-word function names:

```
Handle foo does bar
```

To deal with this, there's one extra rule specific to `gotestdox`:

* If the test name contains at least one underscore, the first underscore is interpreted as marking the end of a multi-word function name: `HandleFoo_DoesBar` -> `HandleFoo does bar`.

This isn't great, but I can't think of any better way to handle this at the moment. At least it means you can write test names for multi-word functions without them looking _too_ weird.

The intent is not to *perfectly* render all sensible test names as sentences, in any case, but to do *something* useful with them, primarily to encourage developers to write test names that are informative descriptions of the unit's behaviour, and thus (as a side effect) read well when formatted by `gotestdox`.

In other words, `gotestdox` is not the thing. It's the thing that gets us to the thing, the end goal being meaningful test names (I like the term _literate_ test names).

# Getting fancy

`gotestdox`, with no arguments, will run the command `go test -json` and process its output.

Any arguments you supply will be passed on to `go test`. For example:

**`gotestdox -run ParseJSON`**

will run the command:

`go test -json -run ParseJSON`

You can supply a list of packages to test, or any other arguments or flags understood by `go test`. However, `gotestdox` only prints events about *tests* (ignoring benchmarks and examples).

If you want to run `go test -json` yourself, for example as part of a shell pipeline, and pipe its output into `gotestdox`, you can do that too:

**`go test -json | gotestdox`**

If `gotestdox` detects that its input is not attached to a terminal, it will wait for you to pipe JSON data into it. Otherwise, it will run the tests for you and process the output.

# Hints

`gotestdox` encourages you to create tests and subtests with descriptive names, because the results read nicely. For example, here's a snippet of one of its own tests:

```go
func TestSentence(t *testing.T) {
	t.Parallel()
	tcs := []struct {
		name, input, want string
	}{
		{
			name:  "correctly renders a well-formed test name",
			input: "TestSumCorrectlySumsInputNumbers",
			want:  "Sum correctly sums input numbers",
		},
		{
			name:  "preserves initialisms such as PDF",
			input: "TestFooGeneratesValidPDFFile",
			want:  "Foo generates valid PDF file",
		},
        ...
```

These subtests are rendered as:

```
 ✔ Sentence correctly renders a well-formed test name (0.00s)
 ✔ Sentence preserves initialisms such as PDF (0.00s)
 ...
```

In other words, it's a good idea to name each subtest so that it completes a sentence beginning with the name of the unit under test, describing the specific behaviour checked by that subtest.

When you use `gotestdox`, you write test names as descriptive sentences quite naturally, without you having to think about it too much—which is the point, of course.

# More examples

Here is the complete `gotestdox` rendering of its own tests (sorted for readability), in case it gives you any useful ideas:

```
 ✔ EventString formats fail events with a cross (0.00s)
 ✔ EventString formats pass events with a tick (0.00s)
 ✔ ExtractFuncName (0.00s)
 ✔ ExtractFuncName correctly extracts func name from a subtest (0.00s)
 ✔ ExtractFuncName doesn't break if the test is named just test (0.00s)
 ✔ ExtractFuncName doesn't break if the test is named just test followed by an underscore (0.00s)
 ✔ ExtractFuncName matches the first camel-case word if there are no slashes or underscores (0.00s)
 ✔ ExtractFuncName treats a single underscore as marking the end of a multi-word function name (0.00s)
 ✔ ExtractFuncName treats a single underscore before the first slash as marking the end of a multi-word function name (0.00s)
 ✔ ExtractFuncName treats multiple underscores as word breaks (0.00s)
 ✔ ExtractFuncName without an underscore before a slash treats camel case as word breaks (0.00s)
 ✔ ParseJSON correctly parses a single go test JSON output line (0.00s)
 ✔ Relevant is false for other events (0.00s)
 ✔ Relevant is true for test pass or fail events (0.00s)
 ✔ Sentence (0.00s)
 ✔ Sentence correctly renders a well-formed test name (0.00s)
 ✔ Sentence doesn't incorrectly title-case single-letter words (0.00s)
 ✔ Sentence eliminates any words containing underscores after splitting (0.00s)
 ✔ Sentence handles multiple underscores with the first marking the end of a multi-word function name (0.00s)
 ✔ Sentence inserts a word break before subtest names beginning with a lowercase letter (0.00s)
 ✔ Sentence is okay with test names not in the form of a sentence (0.00s)
 ✔ Sentence knows that just test is a valid test name (0.00s)
 ✔ Sentence preserves initialisms such as PDF (0.00s)
 ✔ Sentence preserves more initialisms (0.00s)
 ✔ Sentence renders subtest names without the slash and with underscores replaced by spaces (0.00s)
 ✔ Sentence retains apostrophised words in their original form (0.00s)
 ✔ Sentence retains hyphenated words in their original form (0.00s)
 ✔ Sentence treats a single underscore as marking the end of a multi-word function name (0.00s)
 ✔ Sentence treats a single underscore before the first slash as marking the end of a multi-word function name (0.00s)
 ✔ Sentence treats numbers as word separators (0.00s)
 ✔ Sentence treats underscores as word breaks (0.00s)
 ```

# Bugs

## Quoted words in subtests

Right now it doesn't cope well with subtest names containing quoted words. For example, this subtest of `Sentence`:

```go
name:  "preserves initialisms such as 'PDF'",
```

is rendered as:

```
 ✔ Sentence preserves initialisms such as PDF' (0.00s)
```

Note that the opening quote has been lost. I haven't found any way to fix this that doesn't also break everything else. Contributions welcome.

## Lonely test names

When a test (such as `TestSentence`) has subtests, each of those subtests will be correctly printed as its own sentence. However, we'll also get a sentence just containing the test name on its own:

```
 ✔ Sentence (0.00s)
```

I'm not sure how to fix this without keeping track of some kind of state information, as Go's JSON test data doesn't tell you whether or not a given test has subtests. We could store all the results and then filter out such bogus lines, but I prefer to see each line printed as the test runs, without having to wait for them all to finish.

## Your bug here

I'd also like to hear about any odd renderings of (reasonable) test names and how you think they could be improved. Please open an issue with the details.

# Links

- [Bitfield Consulting](https://bitfieldconsulting.com/)

<small>Gopher image by [MariaLetta](https://github.com/MariaLetta/free-gophers-pack)</small>