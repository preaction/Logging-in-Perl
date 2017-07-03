
# Logging in Perl

<http://preaction.github.io/Logging-in-Perl/>

<div style="width: 40%; float: left">

by [Doug Bell](http://preaction.me)  
<small>(he, him, his)</small>  
[<i class="fa fa-twitter"></i> @preaction](http://twitter.com/preaction)  
[<i class="fa fa-github"></i> preaction](http://github.com/preaction)  
<img src="http://chicago.pm.org/theme/images/chicagopm-small.png" style="border: none; vertical-align: middle" />
[Chicago.PM](http://chicago.pm.org)  

</div>
<div style="width: 20%; float: left; text-align: center">
<img src="http://preaction.me/images/avatar-small.jpg" style="display: inline-block; max-width: 100%"/>
</div>
<div style="width: 40%; float: left">

[<i class="fa fa-file-text-o"></i> Notes](https://github.com/preaction/Logging-in-Perl/blob/master/NOTES.md)  
<small> </small>  
[Source on <i class="fa fa-github"></i>](https://github.com/preaction/Logging-in-Perl/)  

[CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/legalcode)  

<small data-markdown>
For navigation help, press `?`  
For speaker view, press `S`  
For full-screen, press `F`
</small>
</div>

------

# Logging

Note:
Logging in a computer program ...

---

# Finding Problems

Note:
... is about finding problems.

------

# `print`

Note:
So the first logging method we learn about is the print function.
Indeed, this is often the first programming thing we learn about. We
teach developers how to find problems before anything else in
programming.

------

# Debug loop

Note:
This leads to a very commonly used debugging process:

---

# Add `print`

Note:
Add a print statement, or two, or ten.

---

# Run the new code

Note:
Run the new code with the print statements in it

---

# Fix the Bug

Note:
The problem is revealed, so we fix the bug.

---

# Delete

Note:
And then, and this is the unfortunate bit, we delete the print
statements...

---

<pre><code data-noescape><span class="fragment fade-in" data-fragment-index=2>use Data::Dumper qw( Dumper );</span>
sub write ( $data ) {
    <span class="fragment fade-in" data-fragment-index=1>say "Writing data to database";</span>
    <span class="fragment fade-in" data-fragment-index=3>say "Data to write: " . Dumper( $data );</span>
    ...;
}</code></pre>

Note:
So this bit of code uses Data::Dumper, and we're going to write some
data to a database, so we say that, and then we dump out the data that
we're about to write. If we got the wrong data, we know we have to look
somewhere else for our problem.

---

# Small and Simple

Note:
If our project is small enough, this might be all we need.

---

# Growing

Note:
But as our project grows...

---

# More reporting

Note:
... so will our need for reporting.

---

# More `print`

(well, `say`)

Note:
Which means more print statements...

------

# Problems

Note:
Which can lead to problems...

---

# Debugging vs. Reporting

Note:
Like which print statements are for debugging, and which statements are
for reporting.

------
<!-- .slide: data-background="black" -->

# Tip

Note:
Which leads to the best tip I've ever been given ever. If you take
nothing from this talk, take this:

---
<!-- .slide: data-background="black" -->

# Make Debugging Visible

Note:
You can make your debugging print statements more visible...

---
<!-- .slide: data-background="black" -->

# Prefix `;`

Note:
... by prefixing them with a semi-colon.

---
<!-- .slide: data-background="black" -->

```
use Data::Dumper qw( Dumper );
sub write ( $data ) {
    say "Writing data to database";
    ; say "Data to write: " . Dumper( $data );
    ...;
}
```

Note:
In this code, it's very, very easy to see which line is the line I added
for temporary debugging.

---
<!-- .slide: data-background="black" -->

# Comment later

Note:
And it's then very very easy to comment this line out for debugging
later.

---
<!-- .slide: data-background="black" -->

```
use Data::Dumper qw( Dumper );
sub write ( $data ) {
    say "Writing data to database";
    #; say "Data to write: " . Dumper( $data );
    ...;
}
```

Note:
Many of you can probably imagine the editor macro you'd have to write to
do this, even.

------

# Debugging vs. Reporting

Note:
But there are other ways of demarking debugging versus reporting.

---

# Use a flag

Note:
We can set a flag to enable/disable our debugging

---

# `$DEBUG`

Note:
I often see people call this $DEBUG

---

# `$ENV{DEBUG}`

Note:
And sometimes I see people use an environment variable for this

---

<pre><code data-noescape>use Data::Dumper qw( Dumper );
sub write ( $data ) {
    say "Writing data to database";
    say "Data to write: " . Dumper( $data )<span class="fragment fade-in"> if $DEBUG</span>;
    ...;
}
</code></pre>

Note:
Then we can use postfix if to check for our flag before writing our
debugging ifnormation

---

# Less visible

Note:
Postfix if is less visible, which could make for confusion as to why
some text isn't printing.

---

# Make a sub?

Note:
To fix this, we could make it into a subroutine, but I'll leave that as
an exercise for you, if you want to after the rest of this talk (and
that means I've failed).

------

# More and Less Reporting

Note:
The next thing I see people adding to their program is a way to
increase/decrease the reporting of the module.

---

# `ssh -v`
<h1 class="fragment fade-in"><code>ssh -vv</code></h1>
<h1 class="fragment fade-in"><code>ssh -vvv</code></h1>

Note:
For example, ssh if we give it one v, is in verbose mode, but we can
give it more v's to make it more verbose

---

# Leave Debugging In

Note:
And this way we can leave our debugging in our code...

---

# Enable Certain Messages

Note:
... and enable certain messages when we need that kind of debugging.

---

# Make `$DEBUG` a Number!

Note:
And one way of doing this I've seen is to make our DEBUG flag into
a number.

---

<pre><code data-noescape>
use Data::Dumper qw( Dumper );
sub write ( $data ) {
    <span class="fragment fade-in">say "Entering write method"</span><span class="fragment fade-in"> if $DEBUG == 2</span>;
    say "Writing data to database"<span class="fragment fade-in"> if $DEBUG</span>;
    say "Data to write: " . Dumper( $data ) if $DEBUG<span class="fragment fade-in"> &gt;= 3</span>;
    ...;
    <span class="fragment fade-in">say "Leaving write method"</span><span class="fragment fade-in"> if $DEBUG == 2</span>;
}
</code></pre>

Note:
So let's add some more reporting to our sub, but conditional on DEBUG
being a certain number. We'll have some pedantic logging if $DEBUG is 2.
If DEBUG is set at all, we mention that we're writing the data to the
database. If DEBUG is greater-than or equal to 3, we dump out the data
we're writing. And finally we add some more pedantic logging.

---

# What Number Do I Want?

Note:
So now I have to pick what number I want when I want debugging.

---

<pre><code data-noescape>
say "Entering write method" if $DEBUG <span class="fragment highlight-red">==</span> 2;
say "Data to write: " . Dumper( $data ) if $DEBUG <span class="fragment highlight-red">&gt;=</span> 3;
</code></pre>

Note:
And you might notice something strange here. One of these compares
equality, one passes a threshold.

---

# `$DEBUG = 3`

Note:
So what happens when I set DEBUG to 3, expecting _more_ debug
information?

---

# No more tracing

Note:
Instead, I lose my tracing messages for entering/leaving the subroutine.

---

<pre><code data-noescape>
say "Entering write method" if $DEBUG <span class="fragment highlight-red">&gt;= 2</span>;
say "Data to write: " . Dumper( $data ) if $DEBUG &gt;= 3;
</code></pre>

---

# Make a sub?

Note:
To prevent problems we could turn this into a subroutine, but again,
before you do that, consider listening to the rest of this and I'll
solve it for you!

------

# Logging System

Note:
If you've done any of these things I've mentioned, you might be
interested in switching to a logging system.

---

# Everything

Note:
A logging system will do literally everything we've talked about so far.

------

# Translate to Log::Any

Note:
So let's translate the code I've been talking about to using Log::Any,
a fast logging system built for integrating with other logging systems.

---

# But first...

Note: But before we can do that...

---

# Log Levels

Note:
... we should talk about log levels.

---

# `$DEBUG`

Note:
Anyone who has used logging libraries before might have recognized
that the $DEBUG number I mentioned functions as a "logging level".

---

# Log Severity

Note:
A log level is the severity of the log message. Some messages are
important for users, others aren't. Some are only important when looking
for problems and are expensive and should be turned off if possible.

---

# Log4perl / Log4j Levels

Note:
There are two standard-ish sets of log levels, of which I prefer the
Log4j / Log4perl levels (the other being syslog levels). These levels
are:

---

# 1: TRACE

Note:
Trace is pedantic execution tracing, like entering/leaving methods.

---

# 2: DEBUG

Note:
Debug is for useful debugging information, like data dumps and internal
branching decisions.

---

# 3: INFO

Note:
Info is for general information about the program's behavior, like
"Fetching the page" or "Committing changes".

---

# 4: WARN

Note:
Warn is for something unexpected, but recoverable. The user might have
to do something to deal with the warning.

---

# 5: ERROR

Note:
Error is for something unexpected and unrecoverable. The thing the user
wanted to do has not been done, and the user needs to do something to
fix it.

---

# 6: FATAL

Note:
Fatal is for something catestrophic (usually an uncaught exception), and
we are halting execution immediately.

---

# Thresholds

Note:
Each log level includes every higher log level, so if I ask to see info
messages I will also see all warn, error, and fatal messages.

------

# Log::Any

Note:
So now we can translate the logging messages we have into the
appropriate log level using Log::Any.

---

<pre><code data-noescape>
use Log::Any qw( $LOG );
sub write ( $data ) {
    $LOG->trace( "Entering write method" );
    $LOG->info( "Writing data to database" );
    $LOG->debugf( "Data to write: %s", $data );
    ...;
    $LOG->trace( "Leaving write method" );
}
</code></pre>

Note:
Here's what that looks like, but let's go through that more slowly.

---

## `use Log::Any qw( $LOG )`

Note:
First, we get a logger object $LOG.

---

# Levels are Methods

Note:
On this object, log levels are methods.

---

## `$LOG->trace( "Entering write method" );`

Note:
So here's our trace method saying that we're entering the method

---

## `$LOG->info( "Writing data to database" );`

Note:
And here's an info-level message telling the user we're writing the
data.

---

# `*f` for Format

(like `sprintf`)

Note: The method name can end with an `f` for `format`, like `sprintf`

---

#### `$LOG->debugf( <span class="fragment highlight-current-red">"Data to write: %s"</span>, <span class="fragment highlight-current-red">$data</span> )`

Note:
So this will use the sprintf format string, and fill in the contents of
the $data variable.

---

# `*f` for Data::Dumper

Note:
Not only that, but if a data structure is given, it will be formatted
with Data::Dumper, showing the whole data structure.

------

# Levels are a Contract

Note:
One thing to note with the log levels is that we now have a convention
for what log level will enable what kind of logging message.

---

# Contracts are Clarity

Note:
Having these agreed-upon conventions will reduce confusion for people
trying to use your logging for debugging

------

# `warn` and `die`

Note:
We have some other Perl functions that are useful for reporting
problems, or necessary for throwing exceptions.

---

# Loud

Note:
These two functions are necessarily loud, and when we start using a log
library, we often stop using warn.

---

# Deprecations

Note:
Which is bad, because warn is still useful for making sure the user sees
a message even if the logging isn't configured, like for deprecations.

---

# Coersions

Note:
Or like Perl uses them for coersions.

---

Are you sure you meant what you said?

Note:
In short, Perl's `warn` function is a polite nudge: Are you sure this is
what you meant?

---

# `warn`

Note:
So the warn function is...

---

# Like `print`, but...

Note:
... similar to print, except...

---

# `STDERR`

Note:
It writes to STDERR and...

---

# Adds code line number

Note:
... adds the code line number to help track down where in the code the
warning comes from.

------
<!-- .slide data-background-color="black" -->

# `STDERR`

Note:
A short note about STDERR.

---
<!-- .slide data-background-color="black" -->

# Terminal

Note:
By default, STDERR is written to the terminal

---
<!-- .slide data-background-color="black" -->

# Even if Piped!

Note:
Even if part of a shell script pipe

---
<!-- .slide data-background-color="black" -->

```
$ perl myscript.pl | grep alert
Something's wrong at myscript.pl line 9.
```

Note:
So, for example, we pipe the output of myscript.pl to grep, but we still
see the warning.

---
<!-- .slide data-background-color="black" -->

# Diagnostic Destination

Note:
So it's important to know that STDERR is the place where problems should
go, where problems are expected to go, and where problems won't be
hidden by pipes.

------

# Warn even if Logging

Note:
So because warn is important, we should warn even if we're writing to
our log.

---

# Helps user

Note:
It helps the user running the script, because the log might be writing
to a file or syslog.

---

# Helps `cron`

Note:
It helps cron, since by default everything written to STDERR is sent via
e-mail to the crontab's owner. I can't even begin to say how many
problems I've found this way.

------

# Log::Any

Note:
So, back to Log::Any.

---

# Works with `warn`

Note:
We should make sure that we write a warning to STDERR while we write
a warning/error to our own log

---

<pre><code data-noescape>
use Log::Any qw( $LOG );
sub write ( $data ) {
    <span class="fragment highlight-current-red" data-fragment-index=2>warn</span> <span class="fragment highlight-current-red" data-fragment-index=1>$LOG->warn( "Data is missing 'id' field" )</span>
      unless $data->{id};
    my $dbh = DBI->connect( ... );
    ...;
}
</code></pre>

Note:
So we can take the return value from the logging method and pass it into
warn. The logging method returns the formatted log message

---

# `die` too!

Note:
More importantly this works for `die` too!

---

```
use Log::Any qw( $LOG );
sub write ( $data ) {
    warn $LOG->warn( "Data is missing 'id' field" )
      unless $data->{id};
    my $dbh = DBI->connect( ... )
      or <span class="fragment highlight-red">die $LOG->fatal( "Failed to connect: $DBI::errstr\n" );</span>
    ...;
}
```

Note:
It is extremely important that any exceptions, caught or not, get
logged. Again, you would not believe how often a blind eval traps a very
important error which takes days to unravel. If you force yourself to
log every die, you can track this down faster.

------

# Producing

Note:
This is all I have for you on producing logs.

------

# Consuming

Note:
So no we have to talk about consuming logs...

------

# Where

Note:
Where do the logs go?

---

# Nowhere
<h1 class="fragment fade-in">Quickly</h1>

Note:
In Log::Any, unless you explicitly set up a place for logs to go, they
go nowhere. Very quickly. There's almost no performance penalty to
leaving Log::Any around, waiting, hiding...

---

# Adapters

Note:
But for our logging to be maximally-useful, we need to write our log to
somewhere. Log::Any calls this "consuming logs", and has a series of
adapters for doing it.

------

# Stderr

Note:
The easiest thing to do is simply to write them to STDERR.

---

```
use Log::Any::Adapter Stderr =>;
```

Note:
Log::Any makes this very easy: We just load Log::Any::Adapter and tell
it to use the Stderr adapter. This loads the Log::Any::Adapter::Stderr
module, which configures all the loggers to write to STDERR.

---

# Other Adapters

Note:
There are lots of other adapters, including ones for writing to files,
STDOUT (if you must), and even other Perl logging systems like
Log::Dispatch and Log4perl.

---

# Other Log Libraries

Note:
It's this bit that makes Log::Any ideal for CPAN modules: Any user of
your module can plug your Log::Any logging into their existing logging
system.

------

# Managing Logs

Note:
Writing to STDERR is really the simplest useful thing we could possibly
do, but it's really only useful for very small applications.

---

# Ephemeral

Note:
STDERR logs are ephemeral. They vanish when the window is closed. This
is of limited use in the long-term.

---

# Long-term

Note:
As an application grows, it becomes more important to have good logs we
can refer to. Problems will happen, and we often need to be able to go
back weeks to see what happened.

---

# Centralized

Note:
More and more solutions are being distributed across multiple physical
(or virtual) machines. So it's best for these systems to write all their
logs to a central location for analysis.

---

# Auditing

Note:
High-value systems could be required to store logs for extended periods
of time for auditing purposes

---

# Monitoring

Note:
And operations teams could want a dashboard powered by logging
information.

------

# Storing

Note:
All of these involve storing logs for some period of time (possibly
forever). So, here's some tips on what solutions to use.

------

# Files

Note:
File logging is a good solution...

---

# Short

Note:
... for very short periods of times, days or weeks.

---

# Small

Note:
... or for very slow-moving / small logs.

---

# Local

Note:
Files are stored locally, so if you've got a networked platform, these
aren't the best solution

---

# Rotated

Note:
File logs must also be rotated: Yesterday's file is renamed, and a new
file is started for today.

---

# Log::Log4perl
# Log::Dispatch

Note:
These Perl logging systems have file rotation built-in

---

# `logrotate`

Note:
But there's also a common unix utility called logrotate which will not
only rotate logs...

---

# Date or Size

Note:
... based on date or size...

---

# Compress

Note:
But it will also compress them

---

# E-mail

Note:
And can even e-mail them if you want

---

# `logrotate`
<h1 class="fragment fade-in">Recommended!</h1>

Note:
So, I highly recommend using lograte.

------

# Syslog

Note:
Syslog is another way of processing and storing logs

---

# 4.2 BSD

Note:
And has been part of all Unixes since 4.2 BSD (1983). It's the standard
logging system, and no, systemd can not take that away from me.

---

# `syslog(...)`

The syslog idea is that all programs log via the standard `syslog`
function

---

<pre class="larger">
/var/log/authlog
/var/log/daemon
/var/log/maillog
/var/log/messages
/var/log/secure
</pre>

Note:
... and then the system configuration can decide where the logs
should go, most likely a file in `/var/log`...

---

# Network

Note:
... but also syslog can ship logs over the network to a central location.

---

# Rsyslog

Note:
Most modern deployments of a syslog solution involve an implementation
called "rsyslog".

---

# CPAN Testers

Note:
I'm planning on using rsyslog for CPAN Testers

---

# Write Locally

Note:
It will write a local file, so I can diagnose problems on the machine.

---

# Act Globally

Note:
But it will also ship to the monitoring server which stores the logs for
longer and parses the logs for metrics and error messages.

------

# Databases

Note:
Recently it's becoming popular to store logs in a database for
reporting.

---

# Sphinx

Note:
A long time ago one of the Madison Perl Mongers talked about using the
Sphinx full-text search engine to store logs that can be easily-searched.

---

# Elastic

Note:
More common nowadays is to use ElasticSearch, a document database that
does full-text searching.

---

# Logstash

Note:
To write the logs into Elastic, they must be parsed into a data
structure by something like logstash. Logstash is like an ETL for logs.

---

# Kibana

Note:
Once they're parsed and stored, tools like Kibana let you build pretty
visual dashboards, manipulating your logs into time series to make
graphs.

---

# ELK
## Elastic
## Logstash
## Kibana

Note:
These are often used together, as the ELK stack.

---

# TICK
## Telegraf
## InfluxDB
## Chronograf
## Kapacitor

Note:
There are other solutions in this space as well

---

## Graphite
## Grafana

Note:
Not all of them have cute initialisms in a well-integrated stack

---

<h1 class="fragment fade-in">CPAN Testers</h1>
## Grafana
## Rsyslog
## InfluxDB
## Telegraf

Note:
And you can mix and match the solutions to customize what you need,
like, for example, this is what I'm using for CPAN Testers

------

# Read Logs

Note:
So managing our logs in a database allows us to read them

---

# Graphs

Note:
But we can also see graphs of the data in the logs, like how many
requests we're serving at a time.

---

# Insight

Note:
So now your logs can provide business insight

---

# Campaign Report

Note:
You can report on a marketing campaign

---

# Usage Projection

Note:
You can do usage projection for capacity planning

---

# Track Down Problems

Note:
Track down when problems started to figure out how to correct them

------

# Questions?

------

# Thank You!

<http://preaction.github.io/Logging-in-Perl/>

<div style="width: 40%; float: left">

by [Doug Bell](http://preaction.me)  
<small>(he, him, his)</small>  
[<i class="fa fa-twitter"></i> @preaction](http://twitter.com/preaction)  
[<i class="fa fa-github"></i> preaction](http://github.com/preaction)  
<img src="http://chicago.pm.org/theme/images/chicagopm-small.png" style="border: none; vertical-align: middle" />
[Chicago.PM](http://chicago.pm.org)  

</div>
<div style="width: 20%; float: left; text-align: center">
<img src="http://preaction.me/images/avatar-small.jpg" style="display: inline-block; max-width: 100%"/>
</div>
<div style="width: 40%; float: left">

[<i class="fa fa-file-text-o"></i> Notes](https://github.com/preaction/Logging-in-Perl/blob/master/NOTES.md)  
<small> </small>  
[Source on <i class="fa fa-github"></i>](https://github.com/preaction/Logging-in-Perl/)  

[CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/legalcode)  

