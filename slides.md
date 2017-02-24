
# Logging in Perl

<http://preaction.github.io/Logging-in-Perl/>

<div style="width: 40%; float: left">

by [Doug Bell](http://preaction.me)  
<small>(he, him, his)</small>  
[<i class="fa fa-twitter"></i> @preaction](http://twitter.com/preaction)  
[<i class="fa fa-github"></i> preaction](http://github.com/preaction)  
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

<small>
For speaker view, press `S`<br/>
For full-screen, press `F`
</small>
</div>

------

# Logging

---

# Finding Problems

------

# `print`

---

# Say what's happening

---

# `Data::Dumper`

---

# Add `print`

---

# Fix the Bug

---

# Delete

---

```
use Data::Dumper qw( Dumper );
sub write ( $data ) {
    say "Writing data to database";
    say "Data to write: " . Dumper( $data );
    ...;
}
```

---

# Small and Simple

---

# Growing

---

# More reporting

---

# More `print`

(well, `say`)

------

# Problems

------

# Debugging vs. Reporting

------

# Tip

---

# Make Debugging Visible

---

# Prefix a `;`

---

```
use Data::Dumper qw( Dumper );
sub write ( $data ) {
    say "Writing data to database";
    ; say "Data to write: " . Dumper( $data );
    ...;
}
```

---

# Comment later

---

```
use Data::Dumper qw( Dumper );
sub write ( $data ) {
    say "Writing data to database";
    #; say "Data to write: " . Dumper( $data );
    ...;
}
```

------

# Debugging vs. Reporting

---

# Use a flag

---

# `$DEBUG`

---

# `$ENV{DEBUG}`

---

```
use Data::Dumper qw( Dumper );
sub write ( $data ) {
    say "Writing data to database";
    say "Data to write: " . Dumper( $data ) if $DEBUG;
    ...;
}
```

---

# Less visible

---

# Make a sub?

------

# More and Less Reporting

---

# Leave Debugging In

---

# Enable Certain Messages

---

# Make `$DEBUG` a Number!

---

```
use Data::Dumper qw( Dumper );
sub write ( $data ) {
    say "Entering write method" if $DEBUG == 2;
    say "Writing data to database" if $DEBUG;
    say "Data to write: " . Dumper( $data ) if $DEBUG >= 3;
    ...;
    say "Leaving write method" if $DEBUG == 2;
}
```

---

# What Number Do I Want?

---

```
    say "Entering write method" if $DEBUG == 2;
    say "Data to write: " . Dumper( $data ) if $DEBUG >= 3;
```

---

# `$DEBUG = 3`

---

# No more tracing

---

```
    say "Entering write method" if $DEBUG >= 2;
    say "Data to write: " . Dumper( $data ) if $DEBUG >= 3;
```

---

# Make a sub?

------

# Logging System

---

# `$DEBUG` Log Level

---

# Log4perl / Log4j Levels

---

# 1: TRACE

---

# 2: DEBUG

---

# 3: INFO

---

# 4: WARN

---

# 5: ERROR

---

# 6: FATAL

---

# Thresholds

------

# Log::Any

---

```
use Log::Any qw( $LOG );
sub write ( $data ) {
    $LOG->trace( "Entering write method" );
    $LOG->info( "Writing data to database" );
    $LOG->debugf( "Data to write: %s", $data );
    ...;
    $LOG->trace( "Leaving write method" );
}
```

---

## `use Log::Any qw( $LOG )`

---

# Levels are Methods

---

# `*f` for Format

(like `sprintf`)

---

#### `$LOG->debugf( "Data to write: %s", $data )`

---

# `*f` for Data::Dumper

------

# Levels are a Contract

---

# Trace is more logging

---

# Contracts are Clarity

------

# `warn` and `die`

---

# Problems

---

# Loud

---

# Deprecations

---

# Coersions

---

Are you sure you meant what you said?

---

# `warn`

---

# Like `print`, but...

---

# `STDERR`

---

# Adds code line number

------

# `STDERR`

---

# Terminal

---

# Even if Piped!

---

```
$ perl myscript.pl | grep alert
Something's wrong at myscript.pl line 9.
```

---

# Diagnostic Destination

------

# Warn even if Logging

---

# Helps user

---

# Helps `cron`

------

# Log::Any

---

# Works with `warn`

---

# Returning the Message

---

```
use Log::Any qw( $LOG );
sub write ( $data ) {
    warn $LOG->warn( "Data is missing 'id' field" )
      unless $data->{id};
    my $dbh = DBI->connect( ... );
    ...;
}
```

---

# `die` too!

---

```
use Log::Any qw( $LOG );
sub write ( $data ) {
    warn $LOG->warn( "Data is missing 'id' field" )
      unless $data->{id};
    my $dbh = DBI->connect( ... )
      or die $LOG->fatal( "Failed to connect: $DBI::errstr\n" );
    ...;
}
```

------

# Carp

---

# Point to Caller

---

# `carp`

---

# `warn` to Caller

---

```
 1: package Module;
 2: use Carp qw( carp );
 3: sub foo ( ) {
 4:     warn "foo called";
 5: }
 6: sub bar ( ) {
 7:     carp "bar called";
 8: }
```

---

```
10: package main;
11: Module::foo();
12: Module::bar();
```

---

```
$ perl test-carp.pl
foo called at test.pl line 4.
bar called at test.pl line 12.
```

---

---

------

# Questions?

------

# Thank You!

Slides on Github:  
<https://github.com/preaction/Logging-in-Perl>

<img src="http://preaction.me/images/avatar-small.jpg" style="display: inline-block; max-width: 100%"/>

[<i class="fa fa-twitter"></i> @preaction](http://twitter.com/preaction)  
[<i class="fa fa-github"></i> preaction](http://github.com/preaction)  

