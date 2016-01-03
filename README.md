The **Random Generator Language** (RGL, pronounced "regal") is a format to define arbitrary random generators, designed to create and collaborate on things like fantasy city generators or "find your rock star name" tools. Though, at its most basic level it is just a text substitution engine, so you could also use it for more practical purposes like generating passwords or ad-hoc test inputs as well.

RGL's primary goals are, in loosely descending order of importance:

1. RGL should be easy to read and understand.
2. RGL should be easy to write with basic text editors.
3. RGL's syntax should be "sugary".
4. RGL should find complexity through layered simplicity.

It also has a few anti-goals:

1. RGL is not made to generate images, structured data, or binary files. It is designed for plain text.
2. RGL is not capable of complex logic. It is designed for non-programmers.

RGL takes significant cues from YAML, Markdown, Liquid, Mustache, Ruby, and Coffeescript. Without them, RGL would be harder to read, harder to write, and objectively worse. Thanks to their authors and maintainers.

## Terms

_Title cased words and phrases in this document refer to technical terms defined here. For example, a "foo" is some nonsensical word, but a "Foo" is an object with a specific definition._

Generator
: A Generator is the object resulting from parsing an RGL-formatted file. Generally, it can also refer to the file itself, as in "run this Generator to create your own countries".

Output
: The Output is the text that is created when you run a Generator.

Space Character
: Space Characters are any whitespace character that is not a line break. Spaces and tabs, for example, are valid Space Characters.

String
: A String is a series of text characters surrounded by single or double quotation marks. There is no difference between the two. Strings can contain a few, simple **escape sequences**:
:
: - `\\'` and `\\"` insert the quotation mark without allowing it to start or end the surrounding of a String. For example, `""foo""` would result in an error, but `"\\"foo\\""` is a String containing the text `"foo"`.
: - `\\@` inserts an at-sign without interpolating the Variable. For example, `"foo = @foo"` is a String containing `foo = ` and the value of `@foo`, but `"foo = \\@foo"` is a String containing the text `foo = @foo`.

Set
: Sets are a collection of Literals. In other computer languages, Sets would either be known as arrays, sets, or unordered lists. Sets in RGL can be created by surrounding the whole collection of Literals with square brackets ("[" and "]") and separating each Literal with a comma (",").

Multiplier
: Items in a Set can be preceeded by a Multiplier, an object unique to RGL. Before each Literal in a Set you can add a number, zero or more spaces, an asterisk ("\*"), and zero or more spaces. This tells the parser to "weight" the Literal a given number of times when choosing a Literal from this Set. For example, in the Set `["foo", "bar"]` "foo" is equally as likely to be chosen as "bar", but in the Set `[3\* "foo", 1\* "bar"]` foo is three times more likely to be chosen.
:
: All Literals in a Set will have a Multiplier of 1 unless they are specifically defined, so you only have to write `[3\* "foo", "bar"]`.

Literal
: A Literal is a generic, umbrella term for a String, Set, or Variable.

Variable
: A Variable is a symbolic name which refers to another value. Variables in RGL consist of an at-sign ("@") and one or more alphanumeric characters. Variables may not start with a number.
:
: Their values can be _assigned_ using the equals sign ("="). For example, `@foo = "str"` would set the `@foo` Variable's value to the String `str`.
:
: Their values can be _recalled_ by entering the name of the Variable. For example, `@bar = @foo` would set the `@bar` Variable's value to the value of the `@foo` Variable.
:
: The **Output Variable** is a special Variable that _must be present_ in every Generator. It _must be called "@output"_, and is the Variable the RGL parser will use to determine the Output of the Generator.

## Interpolation in Strings

Variable names in Strings are interpolated when the String is parsed. When the String `"foo @bar foo"` is parsed, the "@bar" portion of the String is replaced with the value of the `@bar` Variable. If the Variable's value is a Set, the parser _chooses a random Literal from the Set_.

This random selection feature provides the core functionality of the language: When you give the Output Variable a Set of data, or include a Variable whose value is a Set, the parser starts choosing and filling in values until it comes up with a plain text String.

## Writing Generators

Each top-level line of a Generator is a Variable, and all Generators must contain an Output Variable. The parser will process the entire Generator before trying to resolve the Output Variable, so the order of lines is not important (unless you overwrite a Variable).

Here is a simple Generator:

```
@output = "@firstName @lastName"

@firstName = ["Ansel", "Bilbo", "Captain", "Donald"]
@lastName = ["Adams", "Baggins", "Crunch", "Duck"]
```

Some example Outputs from that Generator could be "Ansel Adams", "Ansel Baggins", "Donald Adams", "Captain Duck", and so on.

The real power of Generators comes in layering them together for the parser to construct varied Outputs. Here is a slightly more complicated Generator:

```
@output = "@prefix@nameStyle"

@prefix = [
	4* "",
	"Dr. ",
	"President "
]

@nameStyle = [
	"@firstName",
	"@firstName @lastName",
	"@firstName @lastName-@lastname",
	"@firstName
]

@firstName = ["Ansel", "Bilbo", "Captain", "Donald"]
@lastName = ["Adams", "Baggins", "Crunch", "Duck"]
```

Some example Outputs from this Generator could be "Donald", "Bilbo Crunch", or "Dr. Ansel Duck".