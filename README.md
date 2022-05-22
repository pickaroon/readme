README
======

Welcome to Pickaroon! Like its real-world namesake,
it's a tool that aims to make it easier for you
to manually rummage around in a heap of logs.


What is it?
-----------

Pickaroon aims to be a tool that supports you if you're
trying to manually analyze heterogeneous application logs.
Another way to think of its intended usage target is
"after the fact, log-based debugging".

Within that framework, one of the main goals of Pickaroon
is to make your computer work for you. If you're debugging
in a system that is based on message passing (or even event
driven), you'll have to work with many different kinds of
log messages and you'll likely only ever be interested
in a tiny subset of the information present in your logs.
Pickaroon allows you to write scripts that highlight the
information you're interested in and blend out things you
don't need to see right now, which hopefully helps you
focus on the task at hand.


What is it not?
---------------

If you're searching for "log analysis" on the internet, you'll
find a lot of tools that involve automated real-time analysis
of usually very homogenuous logs -- often with pretty
visualizations or some kind of pattern matching.
Pickaroon is not intended to replace any of them. You might
reach for it if you want to hunt down a bug or an error in a
very specific scenario and want to get an overview of what
actually happened.

Pickaroon is also not designed to deal with large numbers of
log messages at a time (its performance might get sluggish if
you load more than a few thousand messages). If you're running
into performance problems, try pre-filtering your logs
accordingly (you might even do so in your
[data fetching scripts](#data-fetching-scripts)).

Also, while it makes use of the browser's scripting and UI
capabilities, it is by no means a lean web application --
only its use of the Typescript compiler alone means that it
weighs in at multiple tens of megabytes. If you'd like to
use Pickaroon in a constrained network environment, it might
be a good idea to load it from disk or via a local HTTP server
instead. Since Pickaroon is a static web application, doing
so should not impose any major problems (but it could limit
your ability to access logs via HTTP due to [CORS] restrictions).

[CORS]: https://developer.mozilla.org/de/docs/Web/HTTP/CORS

Inspirations
------------

In his book [The Design of Everyday Things], Donald Norman
succinctly explains how short feedback cycles can dramatically
improve the way a user interacts with a tool.  Bret Victor
demonstrated many of the same ideas in his impressive talk
[Inventing on Principle].

[The Design of Everyday Things]: https://en.wikipedia.org/wiki/The_Design_of_Everyday_Things
[Inventing on Principle]: https://www.youtube.com/watch?v=PUv66718DII


Since a big part of debugging issues with the abundance of data
one often finds in application logs is focussing on the right
things, Pickaroon draws inspiration from many powerful UNIX
filtering and transformation tools (such as [GREP] and [AWK]
[^1] [^2]) and the way they can be combined to form a dynamic
toolchain one can use to interactively analyze logs.

[GREP]: https://en.wikipedia.org/wiki/Grep
[AWK]: https://en.wikipedia.org/wiki/AWK

[^1]: If you're not familiar with `awk`, have a look at the great [manual for GNU awk](https://www.gnu.org/software/gawk/manual/gawk.html) (`gawk`)

[^2]: The [original book](https://ia803404.us.archive.org/0/items/pdfy-MgN0H1joIoDVoIC7/The_AWK_Programming_Language.pdf) on `awk` is also a great read


Pickaroon's way of dealing with messages is also similar to the
way the GNU Debugger ([GDB]) allows users to write custom pretty
printers for any datatype using its integrated Python scripting
environment [^3] [^4].

[GDB]: https://www.gnu.org/software/gdb/

[^3]: See also: The [GDB manual](https://sourceware.org/gdb/download/onlinedocs/gdb.pdf), page 158

[^4]: Tom Tromey has written some simple tutorial on the topic here: [part 1](http://tromey.com/blog/?p=524), [part 2](http://tromey.com/blog/?p=546)

Finally, the presentation of Pickaroon takes many ideas from
Matt Godbolt's Tool [Compiler Explorer] and the way it allows
users to dynamically set up their own environment for quick
comparisons of generated assembly, adapted to the individual
user's current task.

[Compiler Explorer]: https://godbolt.org/


Usage
-----

The main goal of Pickaroon is to let your computer do the
tedious tasks involved with analyzing complex application
log messages. If you're combing through a log file trying
to figure out what exactly happened in a specific error
case, it's often very helpful if you can focus on the
specific context needed to understand the problem.

In order to achieve this, you can order Pickaroon to filter
out things that aren't of interest to your current problem
by applying transformation scripts to all messages in your
log. Similarly to `grep` in a UNIX environment, you can
tell it to filter out messages you don't currently care
about, and by generating "pretty printed" lines from log
messages, you are able to only look at the specific details
you currently need to see, potentially ignoring the bulk of
information otherwise present in the log you're analyzing.

In order to do so, you should have at least a passing
familiarity with the scripting language [Typescript]
Pickaroon uses for its automation tasks (or, alternatively,
[Javascript], as any Javascript program is valid Typescript).

[Typescript]: https://www.typescriptlang.org/
[Javascript]: https://en.wikipedia.org/wiki/JavaScript


Sharing sessions
----------------

Pickaroon will continuously encode your session state into
your browser's location bar: The resulting Pickaroon URL is
rather long, but it will contain all your current state so
you can share it with your colleagues if you'd like to convey
a specific problem or let them have a look at what you have
uncovered.

Note however, that this state *does not include* the log data
you have loaded into Pickaroon. If your colleague opens the
URL you've sent them, they will simply get to see the components
you've set up and the fetching code and query parameters you
have been using at the time you've copied your Pickaroon
session's URL. The session data also contains the messages
you have marked as well as any message notes you might have
entered, but they won't be visible until the newly opened
session also succeeded at loading the message data you were
looking at.

This means that the [data fetching scripts](#data-fetching-scripts)
you use in Pickaroon should be kept deterministic if you'd like
to make use of this feature. If you write your scripts in a
non-deterministic way (e.g. by somehow using the current
timestamp to decide which messages to load), you cannot assume
that any other session spawned from the same URL will display
the same content you were looking at.


UI components
-------------

Pickaroon is divided into multiple separate UI components
you can combine into a log analysis environment that best
suits your needs. These UI component can be accessed through
the "Windows" button in the top left corner. Upon clicking
it, a simple menu will be shown on the left border of the
application window, displaying all accessible UI components.

In order to open such a component, simply drag the respective
button into the application's working area. As you drag the
button around your screen, you'll see the area the component
will be created in as soon as you let go of the left mouse
button. It is possible to open multiple instances of the same
component this way (i.e. you might open the "Message Details"
component multiple times if you want to simultaneously look
at multiple representations of the currently selected message).


### Messages

The "Messages" component displays a list of all messages that
are currently loaded into Pickaroon. The text line used to
represent each message is the result of the according call to
the `transform` function from the currently active
[transformation script](#transformation-scripts).

The messages component allows for some simple interactions:
You can select a message (which will influence which message
content is displayed in the "Message Details", "Message Log"
and "Message Notes" components). It's also possible to highlight
("mark") messages that are of a particular interest. In order
to do so, simply double-click the according entry in the list
or press the Space bar while the respective message is selected.

You can also move your message selection using your arrow keys.
Holding shift while using your arrow keys will jump to the next
highlighted message in the according direction (instead of simply
jumping to the next visible message in the list).


### Message Details

Aside from the "Messages" component, the "Message Details"
component is the most useful display in the application. Here,
the contents of the currently selected message are displayed in
a read-only editor window for detailed inspection.

As is outlined in [Transformation scripts](#transformation-scripts),
you can also extend the `transform` function to return new
text representations of a message ("details"), which then
become a display option in this UI component. If you click
on the details selection bar above the editor, you are able
to select either the "Original Message" contents or any detail
view the transformation script has produced for said message.

As noted earlier, it can also be a good idea to open multiple
instances of the "Message Details" component in order to see
multiple visual representations of the same message at a time.

If you'd like to compare the contents of one message to another,
another useful feature is to "pin" a message to the details
component.  To do so, drag the message entry in question from
the "Messages" component's list on top of the "Message Details"
component.  After dropping the message, Pickaroon will create a
copy of the targeted "Message Details" view that is pinned to the
specified message -- even if you select other messages, the
displayed text content will remain the same. You can also change
the message a detail view is pinned to by simply dragging it over
the display component again.

If the two messages you'd like to compare are visually similar
enough that a simple text comparison algorithm (i.e. a "diff")
should suffice, you should also try out the [Message Comparison](#message-comparison)
component.


### Message Log

In the "Message Log" component, you can see all log messages
your transformation scripts produced using the [Console API]
during the transformation of the selected message.

[Console API]: https://developer.mozilla.org/en-US/docs/Web/API/console

This can be very useful if you're trying to debug your
Pickaroon scripts, but it usually isn't really part of
the log analysis process itself.

Like the "Message Details" component, you can pin specific
messages by dragging them into the log component.


### Message Notes

The "Message Notes" component consists of a simple plain text
editor view in which you can jot down quick notes and comments
regarding the currently selected message. The text you enter
into the editor will also be passed on to your message
transformation script, which means that you could even include
them in the "Messages" component if you detect any existing notes
for a specific message.

Like the "Message Details" and "Message Log" components, you can
pin specific messages by dragging them into the log component.

### Message Comparison

The "Message Comparison" component is a handy way of comparing
either the original message or the detailed representations of
two messages using an automatic text comparison ("diff")
visualization. Like in the "Message Details" component, you can
choose the message representation you want to look at using the
details selection bar above the editor view.

In order to choose the two messages you'd like to comare against
each other, you'll again have to drag both message entries from
the "Messages" component over the respective side of the editor
view.


### Local Files

The "Local Files" component is a useful tool if you want to load
log data from your local disk instead via some HTTP API. You can
do so by simply opening up the component and dropping your local
files into it. Each dropped file will appear in the list, and
each visible file will be accessible to your
[data fetching scripts](#data-fetching-scripts).

Note that only one file with a given name can be kept in the list;
if you drop multiple files with the same name into it, only the
last one will be kept around.


### Scratchpad

The "Scratchpad" component is just a simple text editor
component where you can freely enter and edit text. The text
you enter into it isn't tied to any message and won't change
based upon your selection or other state changes.


### Query Parameters

The "Query Parameters" component displays the value that has
been passed as the `query` GET parameter in the URL used to
load Pickaroon in your browser. This is a very useful feature
if you'd like to prepare "pre-built" Pickaroon sessions and
parametrize only a few simple parts of the data fetching
process: You can write your own HTML UIs that allow users to
specify their search terms through an in-browser form and
simply redirect them to a prepared Pickaroon session, passing
the specified search terms on through the `query` parameter.

It's also possible to modify the `query` parameter from within
the Pickaroon session, and doing so will result in the URL in
your browser's location bar changing accordingly (as long as
you entered proper JSON data). However, you'll have to
re-trigger your data fetching script manually if you want to
repeat the loading process.


Pickaroon scripts
-----------------

Pickaroon provides you with two main extension points
you can use to implement the way logs are accessed and
displayed: Data fetching and transformation scripts.

A [data fetching script](#data-fetching-scripts) should,
as its name implies, access the log data you want to
analyze and pass the results back to Pickaroon's main
logic. If necessary, it is possible to also run any logic
on the acquired log data before doing so, which can be a
good way to deal with overly complex log files
(i.e. you can strip out things you don't need or interpret
proprietary log formats before passing the data on in a
JSON-like representation).

[Message transformation scripts](#transformation-scripts),
on the other hand, are tasked with preparing messages in
such a way that they're displayed in a format that is useful
for further analysis. For starters, this can be something as
simple as a call to `JSON.stringify(message)`, but you'll
quickly want to expand them into something more useful
after that.

For very complex, reoccurring analysis tasks, you might
also want to share source code between Pickarron sessions.
In order to achieve this, a third script category --
[library fetching scripts](#library-fetching-scripts) --
might be useful. These scripts basically just load small
blocks of Typescript code you can then import into your
data fetching or transformation scripts.


### Data fetching scripts

At their core, data fetching scripts consist of a single
exported asynchronous function called `loadData` that
accepts an instance of `LoadingArguments` and returns
an instance of `LoadedData`:

```ts
export async function loadData(
    args: LoadingArguments)
    : Promise<LoadedData>
{
    // Data fetching scripts are responsible for loading
    // the messages you're interested in into Pickaroon.
    // They should be returned as a Javascript array of
    // messages (Javascript objects):

   const messages = [
     { hello: 'world' },
   ];

   return { messages };
}
```

The task of this function is the loading of all data
you will be inspecting in your debugging attempts later on.
The most important return value, as can be seen in the
example above, is the `messages` property, which must be
an array of the messages you'd like to analyze. These
messages can be of any type (i.e. there's no need to
limit them to strings; Javascript objects are fine too)
and will be passed to your transformation logic later
on (see also: [Transformation scripts](#transformation-scripts)).

If the log data you are interested in is reachable via
an HTTP API, the best way to implement this function is
usually to use the [Fetch API] that is available in all
modern browsers:

```ts
export async function loadData() {
    // Download log data from an HTTP server using `fetch()`:
    const url = 'http://example.com/path/to/log.txt';
    const log = await fetch(url).then(f => f.text());
    const messages = log.split('\n');
    return { messages };
}
```

[Fetch API]: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API

In order to perform its tasks, the `loadData` function
is also passed a couple of input arguments, as can be
seen in the definition of the `LoadingArguments` interface:

```ts
interface LoadingArguments {
    query: any,
    localFiles: LocalFiles,
};

type LocalFiles = { [name: string]: LocalFile };

interface LocalFile {
    name: string,
    type: string,
    size: number,
    sizeText: string,
    text:   () => Promise<string>,
    stream: () => ReadableStream<any>,
};
```

The `query` argument can be used to parametrize the
fetching of data via the Pickaroon URL used to load
the application (see also: [Query parameters](#query-parameters)).

The `localFiles` argument makes it possible to load
files into the application through drag and drop
actions from the local disk. This is very useful
in settings where one needs to analyze local log
files that aren't readily available through an
HTTP interface.

Besides loading the message data you will be
analyzing later on, the `loadData` function also
has the possibility to load additional data that
isn't related to any individual message:

```ts
interface LoadedData {
    messages: any[],
    additionalData: AdditionalData,
};

type AdditionalData = { [key: string]: any };
```

All data returned in the `additionalData` property
will also be passed to the transformation logic (through
a separate function; see
[Transformation scripts](#transformation-scripts))
and can be used for processing purposes. As an example
of when this might be useful, consider a scenario
where you need to analyze a log containing messages
that rely on an out of band schema definition.


### Transformation scripts

After the data fetching script has run, Pickaroon will
pass all the downloaded data to your transformation logic,
which is responsible for transforming the loaded messages
into representations that help you debug your problem.


#### `transform` function

At the core of each transformation script, there is a
single mandatory function called `transform`:

```ts
export function transform(
    message: Message,
    index: number,
    notes?: string)
    : MessageTransformationResult
{
    // Transformation scripts are responsible for the
    // processing and visualizing of each message.
    // Dynamically changing the visualization of messages
    // can make debugging jobs much easier, as you can
    // focus only the information you need to be aware
    // about to solve the specific problem at hand.

    return { log: JSON.stringify(message) };
}
```

This `transform` function is called sequentially for
each message instance that was previously returned from
the `loadData` function (described in
[Data fetching scripts](#data-fetching-scripts)).
As you can see, it must at least transform each message
passed into it into a single-line string that will then
be displayed in the messages overview. The `transform`
function can also be used to weed out messages you don't
need to see at all -- in this case, all it must do is to
return `null` or `undefined`:

```ts
export function transform(message, index) {
    // `transform` can also filter out unwanted messages:
    if (index % 2 == 0) return; // Ignore all even messages

    return { log: JSON.stringify(message) };
}
```

The main `message` argument being passed to the
function implements the `Message` interface, and in most
cases, it's useful to properly annotate the argument with
this type in your script. Once Pickaroon has run your
data fetching scripts and attained the messages in
question, it will automatically generate interface type
declarations for your messages. This means that if you
annotate the argument appropriately, the integrated
script editor of Pickaroon is able to provide some
rudimentary autocomplete functionality for message
objects:

```ts
export function transform(message: Message) {

    message. // <- Moving the cursor here and hitting Ctrl+Space
             //    will display an autocomplete menu.
             //    Any items suffixed with a quetion mark (?)
             //    are *optional*, because they aren't included
             //    in *all* loaded message instances.

    return { log: JSON.stringify(message) };
}
```

Besides the `message` argument, the `transform` function
also can also accept some additional arguments that might
be useful to the processing logic:

The `index` argument represents the zero-based index
of the passed message in the list that was originally
loaded.

The `notes` argument *might* contain a string containing
the message's user notes (see also: [message notes](#message-notes)).
If there are none, the argument will be `undefined` instead.

Besides returning the single-line representation of a
message for the messages overview, the `transform` function
can also be used to extract further "details" from messages.
In Pickaroon's terms, details are multi-line representations
of a message that deviate from its original value.
The necessary structure of these returned details can be
seen in the interface definition for
`MessageTransformationResult`:

```ts
interface MessageTransformationResult {
    log: LogText,
    details?: { [name: string]: MessageDetail },
};

type LogText = LogText[] | FormattedText | string;

interface FormattedText {
    content: LogText,
    href?:            string,
    color?:           string,
    backgroundColor?: string,
    fontWeight?:      string,
    fontStyle?:       string,
    textDecoration?:  string,
    tooltip?:         string,
};

interface MessageDetail {
    language: string,
    content:  string,
};
```

The `language` property of a single detail instance must
thus be set to the type of the respective detail as it is
understood by Pickaroon's integrated [Monaco] editor component.
The most likely values for this property are `'plaintext'`,
`'json'` and `'xml'`, if you're working with web technologies.

[Monaco]: https://microsoft.github.io/monaco-editor/

The `content` property of a detail instance should contain the
actual text representation as a simple Javascript string.

This "details" feature is very useful for log formats that
nest multiple message formats inside one another -- a good
example of that could be any form of JSON or XML log entries
that are stored inside an [Elastic Stack] instance or something
similar (provided their content isn't parsed further by a codec).

[Elastic Stack]: https://www.elastic.co/elastic-stack/

In such a scenario, it's not unlikely that every message in
your log will contain the actually interesting application log
data nested inside some JSON string (possibly with near-unreadable
escape sequences), surrounded by a lot of mostly unintersting data
(such as, for example, telemetry data from k8s or the like).

Another possible scenario where details are useful also regularly
occurs with JSON data stored in an Elastic Stack: Since the JSON
implementation it uses doesn't maintain the order of the JSON keys
they were originally specified in, _every message stored in an
Elastic stack will be returned in a different order_.
While this is no problem for automated processes accessing said
messages, it can be very frustrating for human readers who try
to quickly gauge the difference between subsequent messages.

One simple way you can deal with this issue is by re-formatting
each message with a deterministic key order:

```ts
export function transform(message: Message) {

    message = reorder(message);

    return {
        log: JSON.stringify(message),
        details: {
            'Deterministic': {
                language: 'json',
                content: JSON.stringify(message, null, 2),
            }
        }
    };
}

function reorder(value: any) {
    const isNull   = value === null;
    const isObject = typeof(value) === 'object';
    const isArray  = Array.isArray(value);

    if (!isObject || isNull) return value;
    if (isArray) return value.map(reorder);

    const result = {};
    Object.keys(value).sort().forEach(k => {
        result[k] = reorder(value[k])
    });
    return result;
}
```

A useful, third way of using the "details" functionality is the
comparison ("diffing") of messages. If you have the chance to
arrange the data you're currently interested in in a deterministic
overview text, it can sometimes be useful to directly compare two
message instances (see also: [Message Comparisons](#message-comparisons))

Also note that through the `LogText` type returned in each
`MessageTransformationResult`, the single-line representation
of each message can make limited use of styling and interaction
behaviour provided by HTML: You can style text (i.e. by formatting
it in a certain color or making it bold or italic) and embed
links to websites or tooltips for certain pieces of the generated
text line. As a simple example, the following code uses these
formatting possibilities to mark messages for which some
[message notes](#message-notes) have been entered:

```ts
function notesMarker(notes?: string): LogText {
    if (!notes?.trim()) return '   ';
    return {
        content: '[!]',
        color: 'orange',
        fontWeight: 'bold',
        tooltip: notes.trim(),
    };
}

export function transform(
    message: Message,
    index: number,
    notes?: string)
    : MessageTransformationResult
{
    const log = [
        `${index}`.padStart(3),
        ' ',
        notesMarker(notes),
        ' ',
        JSON.stringify(message),
    ];

    return { log };
}
```

Similarly, one could imagine creating inline links to something
like a service description or configuration for any daemon that
logged a specific entry:

```ts
function serviceLink(message: Message, width: number) {
    const service = message?._source?.agent?.hostname?.trim();
    if (!service) return ''.padEnd(width);
    return {
        content: service,
        href: `https://services.example.local/service-configs/${service}`,
    };
}

export function transform(
    message: Message,
    index: number,
    notes?: string)
    : MessageTransformationResult
{
    const log = [
        `${index}`.padStart(3),
        ' ',
        serviceLink(message),
        ' ',
        JSON.stringify(message),
    ];

    return { log };
}
```

#### Optional functions

Aside from the `transform` function outlined above, transformation
scripts _may_ also export some optional functions that get called
before or after all messages are transformed:

```ts
export function acceptAdditionalData(data: AdditionalData): void {
    // Additional data provided by the data fetching script
    // can be processed here (before the message transformations
    // are run).
}

export function getMessageId(message: Message, index: number): string {
    // Some features depend on unique identifiers for each message.
    // If your message already contains a unique identifier (as is the
    // case in the Elastic Stack, for example), you can implement a
    // customized getter for that value here:
    return message._id;
}

export function begin(): void {
    // This function is executed before any of the message
    // transformation calls.
}

export function end(): void {
    // This function is executed after all message transformation calls.
}
```

The `begin` and `end` functions are simply called before and after
all message transformation calls are executed (they vaguely correspond
to the `BEGIN` and `END` clauses one might write in [AWK]).

The `acceptAdditionalData` function can be exported in order to
process any data the current data fetching script returns via the
`additionalData` property of its return value. If so, this
function is called after `begin` and before the first message
transformation call.

The `getMessageId` function is a possible extension that must return
a unique identifier (in form of a string) for each message it gets
called on. Pickaroon internally relies on unique message ids for a
couple of its features (such as message notes and message markers);
these ids will be derived from the message's contents by default
(i.e. by hashing the message). In some cases, like if your messages
are stored in an Elastic Stack, your messages might already contain a
unique identifier (i.e. the `_id` field in Elastic [^5]), and hashing
each message won't be necessary.

[^5]: See the Elastic documentation [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-id-field.html)


### Library fetching scripts

Library fetching scripts aren't integral to the process
of analyzing logs, but can help you set up Pickaroon
environments as templates for analysis use cases you
encounter often.

By writing simple Typescript modules, you can extract
script code you use in many places or for many occasions
into "libraries" you can load into a Pickaroon environment
and use in data fetching or message transformation scripts.
This is especially useful if you're often dealing with some
proprietary protocol that must be parsed before being
visualized or if you try to set up a "default" view in
Pickaroon that your internal tooling points to (see also:
[Sharing Sessions](#sharing-sessions),
[Query parameters](#query-parameters)).

A library fetching script must essentially only export a
single function that is responsible for loading your library
code into Pickaroon's state:

```ts
export async function loadLibraries(args: LoadingArguments)
    : Promise<LoadedLibraries>
{

    // `loadLibraries` is responsible for loading the typescript
    // source code of all libraries used in the data fetching and
    // message transformation scripts.

    // These libraries should be returned as a simple
    // name => code hash map.

    return {
        "testlibrary": `
            export function test() {
                console.trace('Test function called from here');
            }
        `
    };
}
```

Libraries that are accessed by the `loadedLibraries` function
must be returned in a simple Javascript hash map, that contains
the respective library name as the key and its Typescript source
code as a string value:

```ts
export type LoadedLibraries = { [name: string]: string };
```

Again, the most likely way this function will access the necessary
files will be the [Fetch API].  After the library fetching script
has been run successfully, the loaded libraries can be imported
as external Typescript modules:

```ts
import { test } from 'testlibrary';

export function transform(message: Message) {

    test();

    return { log: JSON.stringify(message) };
}
```

### Debugging Pickaroon scripts

Since Pickaroon scripts are executed within the web application
itself, debugging Pickaroon scripts through built-in development
tools of various browsers (like a debugger) can be a complicated
ordeal. But, since Pickaroon scripts usually tend to fall into a
simple "input -> transform -> output" pattern, debugging scripts
through log statements is often effective enough to weed out
problems.

In order to facilitate this, Pickaroon replaces the browser's
built-in implementation of the [Console API] with its own version
that makes log output browseable on a per-message basis (see also:
[Message Log](#message-log)). Thus, if something goes wrong within a
transformation script, simply inserting logging calls into the
respective functions of the transformation script is often a simple
way to quickly find out where the actual problem lies.

[Console API]: https://developer.mozilla.org/en-US/docs/Web/API/console


### Built-in libraries

Pickaroon comes with a limited set of built-in libraries that
implement various often-used functionalities browsers currently
don't provide themselves, such as: Base64 encoding and decoding
for *real* binary data, XML pretty printing or a simple JSONPath
API.

You can find these built-in libraries in the Pickaroon source
under `src/imports`.