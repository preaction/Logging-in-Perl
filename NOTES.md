Logging and Reporting is about finding problems. So, let's start with how to
produce useful logs:

## `print`

The `print` function is perhaps our first and best problem-finding tool: We
write a `print` statement around where we think the problem is. Maybe we use
[the Data::Dumper module](http://metacpan.org/pod/Data::Dumper) to show a data
structure to make sure it looks like how it should. But, once we've found the
problem, we usually delete all these `print` statements before we commit our
bugfix.

(I say `print`, but I'm really going to use `say` here just because I don't
want to type `"\n"` all the time)

    use Data::Dumper qw( Dumper );
    sub write ( $data ) {
        say "Writing data to database";
        say "Data to write: " . Dumper( $data );
        ...;
    }

If our project is small enough, this might be all we need. But as our project
grows, so will our need for reporting. So we'll end up leaving some print
statements in to notify our users as we perform actions on their behalf. But,
now we have a few problems:

First, how do we distinguish the debugging print statements we add with the
notifications we have. We don't want to accidentally remove the wrong print
line and confuse our users.

### Determining what reporting is for what audience

#### Tip: Easily find debugging statements

To easily find which statements are added for debugging, prefix the whole line
with a semi-colon `;`. This makes the line stand out...

    use Data::Dumper qw( Dumper );
    sub write ( $data ) {
        say "Writing data to database";
        ; say "Data to write: " . Dumper( $data );
        ...;
    }

... and it makes it easier to find and remove them later:

    use Data::Dumper qw( Dumper );
    sub write ( $data ) {
        say "Writing data to database";
        #; say "Data to write: " . Dumper( $data );
        ...;
    }

#### Using a "DEBUG" flag

I've seen this technique quite a bit: Set a `$DEBUG` flag (or a `$ENV{DEBUG}`
environment variable) to enable/disable more verbose logging. This can be an
easy way to make the internal operation of your program more transparent, but
it's all-or-nothing.

    use Data::Dumper qw( Dumper );
    sub write ( $data ) {
        say "Writing data to database";
        say "Data to write: " . Dumper( $data ) if $DEBUG;
        ...;
    }

And, at least in Perl, the flag comes after the message when reading the
code. So, I've seen some people encapsulate this in a subroutine call
(at which point, you've started down the road to writing your own
logging system, and you have come to the right place to dissuade you of
that idea).

### Increasing and Decreasing Reporting

How do we deal with increasing or decreasing the amount of reporting we
do? It's far more convenient to leave the debugging print statements in
the code: If we needed them once, we're going to need them again. So how
can we selectively enable and disable those print statements?

I've seen people try to fix the binary nature by turning their `$DEBUG`
flag into a number, say 1-10. But then we have this problem: What DEBUG
value does what? And are we handling those values in a sane way?

    use Data::Dumper qw( Dumper );
    sub write ( $data ) {
        say "Entering write method" if $DEBUG == 2;
        say "Writing data to database" if $DEBUG;
        say "Data to write: " . Dumper( $data ) if $DEBUG >= 3;
        ...;
        say "Leaving write method" if $DEBUG == 2;
    }

Here, `$DEBUG = 2` will trace the entry and exit of our method, but
`$DEBUG = 3`, a "higher" debugging value, loses this information. We
should have said `if $DEBUG >= 2` to ensure that higher numbers include
the lower numbers.

If, like before, we wrap this in a subroutine call, we now have an
actual logging system. Since we should avoid reinventing the wheel, by
the time we get to this point, we should consider finding a logging
system on CPAN.

#### Log Levels

Those of you who've used logging before will recognize that my `$DEBUG`
value is setting the logging level. The log4j/log4perl model, which
I prefer to the syslog model, describes 6 logging levels, in ascending
severity:

1. TRACE - Pedantic execution tracing, like entering/leaving methods
2. DEBUG - Useful debugging information, like data dumps and internal branch
   decisions
3. INFO - General information about the program's behavior
4. WARN - Something unexpected, but recoverable, happened that you should know
   about
5. ERROR - Something unexpected and unrecoverable happened that you should know
   about
6. FATAL - Something catastrophic happened and we are halting execution
   immediately

Each log level includes every higher log level, so if I ask to see all INFO
messages, I will also see all WARN, ERROR, and FATAL messages.

So, if I replace my debug print statement with logging, it looks like this:

    use Log::Any qw( $LOG );
    sub write ( $data ) {
        $LOG->trace( "Entering write method" );
        $LOG->info( "Writing data to database" );
        $LOG->debugf( "Data to write: %s", $data );
        ...;
        $LOG->trace( "Leaving write method" );
    }

All we need to do to start using Log::Any is to get a log object by
doing `use Log::Any qw( $LOG )`. In Log::Any, each log level is a method
we call on our `$LOG` object. There is a further set of `*f` methods
that first runs our log message through `sprintf` (and, if given
a reference as data, Data::Dumper too).

You might have noticed that we now have an agreed-upon convention for
what log level will enable what kind of logging messages that was
opposite to what we had with our $DEBUG value: Tracing enter/exit is
a higher logging level than seeing the data we're writing. Having these
agreed-upon conventions will also help reduce the amount of confusion
for people trying to debug your code.

## `warn` and `die`

Sometimes we have a potential problem that people might have in the
future: For example, a user could give our function a string instead of
a number. Or they could give us `undef`, which we used to allow but we
want people to be more explicit now so could they please not give us
`undef` here?

    sub write ( $data ) {
        warn "Data is missing 'id' field" unless $data->{id};
        ...;
    }

For situations like these, Perl offers the `warn` function: `warn` is
similar to `print` except:

* It writes to STDERR, not STDOUT
* It adds a short blurb telling what line the warning came from

Writing to STDERR means that usually the message will be shown on the
user's terminal. The UNIX shell is a programming environment where
a user can string multiple commands together in a "shell script" to
process data. This is called "piping", and it is one of the true
strengths of UNIX. Pipes between programs generally use STDOUT, and do
not write out to the user's terminal. STDERR is left hooked up to the
terminal so that the user can see warnings and diagnostic information as
it happens.

XXX Example 2-2 - Example of pipe with warning output

It is extremely important that these messages be directed to STDERR,
even if we already have a logging system in-place: We don't know if the
user sitting at their terminal is watching the log or even knows where
the logging is being directed. There are also environments like `cron`
which take any output to STDERR and sends it in an e-mail to the cron
user, which can reveal problems.

For this, Log::Any allows us to log the warning and also call `warn`
quite easily:

    use Log::Any qw( $LOG );
    sub write ( $data ) {
        warn $LOG->warn( "Data is missing 'id' field" ) unless $data->{id};
        my $dbh = DBI->connect( ... );
        ...;
    }

Every Log::Any method returns the fully-formatted log line, so we can
pass it to `warn`, or even throw an exception with `die`:

    use Log::Any qw( $LOG );
    sub write ( $data ) {
        warn $LOG->warn( "Data is missing 'id' field" ) unless $data->{id};
        my $dbh = DBI->connect( ... )
            or die $LOG->fatal( "Could not connect to database: $DBI::errstr\n" );
        ...;
    }

This ensures that our logging is complete for wherever the logs will be
stored for later use, but also that anything connected to STDERR of our
terminal will be notified as well.

### Carp

Don't forget that Perl has a built-in set of additional warning
/ exception functions in the [Carp](http://metacpan.org/pod/Carp)
module. These methods make finding the problem easier by pointing the
user at different parts of the code.

`carp` is like `warn`, but it claims that the problem is not inside the
current sub, but instead is where the current sub is being called.

    package Module;
    use Carp qw( carp );
    sub foo ( ) {
        warn "foo called";
    }
    sub bar ( ) {
        carp "bar called";
    }

    package main;
    Module::foo();
    Module::bar();

    $ perl test-carp.pl
    foo called at test.pl line 4.
    bar called at test.pl line 12.

`croak` is the same thing, but with `die` instead, and `cluck` and
`confess` are warn and die with full stack traces.

## Controlling Log Output

### Where

Now that we're generating logs, we need to do something with them. The
easiest thing to do would be to simply write them to STDERR (remember
before where STDERR was for messages for the human, and STDOUT was for
messages for further parts of the program). Log::Any makes this very
easy as well:

    use Log::Any::Adapter Stderr =>;

Once this is done, all Log::Any messages from any part of the program
will be written to STDERR.

#### Log::Any::Adapter

There are Log::Any::Adapters for many things: STDOUT (if you must),
Files (of course), Syslog, and even other Perl logging systems like
Log::Dispatch and Log::Log4perl. It's these last ones that make Log::Any
ideal for adding logs to your CPAN modules: Any user of your module can
plug your Log::Any logging into their existing logging system.

### What

Once we've decided where we want our logs to go, we need to decide at
what level we want to log. Often we want to make this user-configurable
(like our `$ENV{DEBUG}` example).

For the most part, I decide between defaulting to INFO or WARN by how
long the process will take and how much information the user running the
program will expect. As a UNIX-enthusiast, beard and all, I generally
think that my user expects there to be no output unless something was
wrong, so I tend to choose WARN, like so:

    use Log::Any::Adapter Stderr => ( log_level => 'warn' );

Here's where we can use environment variables to control our logging
level. So, if we set `DEBUG=1`, we switch to DEBUG level logging:

    use Log::Any::Adapter Stderr => ( log_level => $ENV{DEBUG} ? 'warn' : 'debug' );

But we can also make some command-line flags to enable more verbose
logging (like INFO level) by switching to the `set()` method of
Log::Any::Adapter:

    use Log::Any::Adapter;
    use Getopt::Long qw( GetOptions );
    GetOptions( \my %opt, 'verbose|v' );
    my $log_level = $opt{verbose} ? 'info' : 'warn';
    Log::Any::Adapter->set( Stderr => ( log_level => $log_level ) );

Now users can do `myprogram -v` to get INFO level logging instead of
WARN level logging.

### How to manage logs

Writing to STDERR is really the simplest useful thing we could possibly
do, but a lot of systems have other requirements:

* An automated system might not have anyone watching the terminal and
  must write its logs to a rotated set of files
* A distributed system could be required to log to a centralized
  location over the network for storage and monitoring
* A high-value system could be required to store logs for extended
  periods of time for future auditing purposes
* An operations team could want an operations dashboard powered by
  logging information

All of these will involve shipping and storing logs for some period of
time (or even forever). In this respect, you must look at your project
and decide how you want to store your logs:

#### Files

File logging is good for shorter periods of time, days or weeks. Files
are generally stored on the same machine they are generated on, since
there are better solutions when you need to get the network involved.
Files usually end up being "rotated": Yesterday's file is renamed, and
today's file is started with the normal name. 

There are lots of logging systems that will rotate files for you
(Log::Dispatch and Log::Log4perl both have one), but there's also
a standard system utility called "logrotate" which will not only rotate
logs, but will do so based on time or size, and also compress them, and
even mail them to you if you want. I highly recommend using "logrotate".

#### Syslog

In all Unixes since 4.2 BSD, Syslog is now the standard system logging
service (no, systemd, you can't take that away from me). The idea is
that all programs log via `syslog(3)`, and then the system can decide
where the logs should go, including shipping logs over the network to
other machines.

Modern deployments of a syslog solution often involve an implementation
called "rsyslog". Rsyslog has facilities for directing logs to different
files based on what log level or even what program they come from.
Rsyslog also can copy logs to multiple places: Log to a local file for
late-night debugging, and log to a remote machine for storage.

In my project at ServerCentral, we use rsyslog on every machine to ship
logs to a centralized location.

#### Databases

Recently it's been becoming more and more popular to store logs in
a database for ad-hoc reporting. A few years back one of the Madison
Perl Mongers talked quite a bit about using MySQL and then Sphinx (a
full-text search engine with a MySQL-like API) to log network packets to
discover malware operating on the network and other network security
issues.

More recently, Jason Crome gave a talk a few months ago about using
Postgres to store logs (with fluentd to parse the log lines into
structured JSON for reporting). At my last couple jobs, we used Logstash
to parse logs and ElasticSearch to store them.

Storing log lines in a database often involves filtering and parsing
those logs to turn them into structured data which can be stored
efficiently and reports built.

Once they're parsed, they can be stored and retrieved later, often with
the help of some very sophisticated GUI tools like Kibana or Grafana.
These tools let you see the log results, but also graph time series
based on the data inside the logs (for example, to see the number of
requests to a web service over time).

In this way, your logging can be used to provide business insight:

* Report on the success of a marketing campaign or A/B testing
* Determine usage projections for the future to plan for hardware capacity
* Track down when problems started to figure out how to correct them

