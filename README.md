# Simple Node Logger
- - -

[![NPM version](https://badge.fury.io/js/simple-node-logger.svg)](http://badge.fury.io/js/simple-node-logger) [![Build Status](https://travis-ci.org/darrylwest/simple-node-logger.svg?branch=develop)](https://travis-ci.org/darrylwest/simple-node-logger) [![Dependency Status](https://david-dm.org/darrylwest/simple-node-logger.svg)](https://david-dm.org/darrylwest/simple-node-logger)

A simple multi-level logger for console, file, and rolling file appenders.  Features include:

- levels: trace, debug, info, warn, error and fatal levels (plus all and off)
- flexible appender/formatters with default to HH:MM:ss.SSS LEVEL message
- add appenders to send output to console, file, rolling file, etc
- change log levels on the fly
- domain and category columns
- overridable format methods in base appender

## Installation

    npm install simple-node-logger --save


## How to use

	// create a stdout console logger
	var log = require('simple-node-logger').createSimpleLogger();

or

	// create a stdout and file logger
	var log = require('simple-node-logger').createSimpleLogger('project.log');
	
or

	// create a custom timestamp format for log statements
	var SimpleNodeLogger = require('simple-node-logger'),
		opts = {
			logFilePath:'mylogfile.log',
			timestampFormat:'YYYY-MM-DD HH:mm:ss.SSS'
		},
		log = SimpleNodeLogger.createSimpleLogger( opts );

or

	// create a file only file logger
	var log = require('simple-node-logger').createSimpleFileLogger('project.log');
	
or

    // create a rolling file logger based on date/time
    var opts = {
        logDirectory:'/mylogfiles',
        fileNamePattern:'roll-<DATE>.log',
        dateFormat:'YYYY.MM.DD'
    };

    var log = require('simple-node-logger').createRollingFileLogger( opts );
    
or

	// create a log manager
    var manager = require('simple-node-logger').createLogManager();
    
    manager.createConsoleAppender();
    
    var log = manager.createLogger('MyClass');
    // create other logs and appenders...
    

The first use simply logs to the console.  The second logs to the console and to the project.log file.  The third create a console logger with a custom timestamp format. The fourth logs to the file only. The fifth creates a rolling file log system in the target log folder.  The fifth creates a log manager to enable you to add various appenders with multiple levels and create logs for each module or class.

*See the examples folder for in depth samples...*

## Log Levels

The log levels include the standard set: trace, debug, info, warn, error and fatal.  The default level is info.  The log level can be set at run-time by doing this:

	log.setLevel('warn');
	
This sets the log level to warn and suppresses debug and info messages.

## Log Statement Formats

### Simple Logger

The default format is HH:mm:ss.SSS LEVEL message. For example, the log message:

	log.info('subscription to ', channel, ' accepted at ', new Date().toJSON());

Yields:

	14:14:21.363 INFO  subscription to /devchannel accepted at 2014-04-10T14:20:52.938Z
	
### Category Logger

If you create a logger with a category name, all log statements will include this category.  Typically a category is a class or module name.  If you create a logger with the category name 'MyCategory', the log statement would format like this:

	 14:14:21.363 INFO  MyCategory subscription to /devchannel accepted at 2014-04-10T14:20:52.938Z
	 
## Appenders

### Console

Writes to the console.  This is the simplest appender typically used for command line applications or for development.

### File

Writes to the specified file.  This appender is typically used for services that periodically start and stop or that have a limited number of log statements.  An example would be to log just error & fatal messages separate from other logs.

### Rolling File Appender

The rolling file appender offers a full production logger where files roll based on date and time.  The minimum roll time is a single hour.  A typical application would be a production environment where log files are rolled throughout the day then archived to a separate location.

The rolling file appender requires a valid date format and file name pattern.  The filename must contain the key word <DATE> that will be replaced with the formatted date.  The configuration must also include a target log directory where the files will be written.

#### Valid Filename Patterns

	mylog-<DATE>.log
	ApplicationName.log.<DATE>
	<DATE>.log
	<DATE>

#### Valid Date Formats

Date formats must map to acceptable file names so have more restrictions than typical dates.  If you use delimiters, you are restricted to a dash or dot delimiter to separate year, month, day and hour.  Valid examples include:

	MMDD  // simple month day that rolls at midnight (no delimiters)
	YYYY.MM.DD-HH // year month day and hour that can roll up to once per hour
	YYYY-MM-DD.a // year month day and am/pm that rolls twice per day
	YYYY-MMM-DD // year month day where month is the short name (Mar, Apr, etc)
	
The default format YYYY.MM.DD is used if the format is not supplied.

## Dynamic Configuration

Create a javascript configuration that implements 'readConfig' to return configuration details.  

## Examples

The examples folder includes a handful of simple to not so simple cases for console, file, multi-appender, category, etc.

## Customizations

### Appenders

Adding a new appender is as easy as implementing write( logEntry ).  The easiest way to implement is by extending the base class AbstractAppender.  You may also easily override the formatting, order, etc by overriding or providing your own abstract or concrete appender.

For example, you can extend the AbstractAppender to create a JSON appender by doing this:

    var AbstractAppender = require('simple-node-logger').AbstractAppender;

    var JSONAppender = function() {
    	'use strict';
    	var appender = this;
    	
        var opts = {
            typeName:'JSONAppender'
        };
        
        AbstractAppender.extend( this, opts );
        
        // format and write all entry/statements
        this.write = function(entry) {
        	var fields = appender.formatEntry( entry );
        	
        	process.stdout.write( JSON.stringify( entry ) + '\n' );
        };
    };

### Overrides

#### Appenders

The appenders have formatting messages that can be overridden at the abstract or concrete level.  The format methods include:

- formatEntry(entry) - to override all formatting
- formatMessage(msgList) - to override a list of messages
- formatDate(value) - custom date, defaults to ISO8601
- formatObject(value) - custom object, defaults to json for regular objects

#### Logger

It's easy to extend any one of the log methods at the instance level.  Here is an example of overrideing the error log to send a socket message:

```
var log = new require('simple-node-logger').createSimpleLogger(),
    socket = openWebSocket();

// override the standard error method to send a socket message
log.error = function() {
    var args = Array.prototype.slice.call( arguments ),
        entry = log.log('error', args);

    // now do something special with the log entry...
    process.nextTick(function() {
    	socket.send( JSON.stringify( entry ));
    });
};
```

## Tests

All unit tests are written in mocha/chai/should and can be run from the command line by doing this:

	make test
	
There is also a file watcher that can be invoked with this:

	make watch
	
	
## Mocks

Mocks used for testing include MockLogger and MockAppender.  Typically you would use MockLogger for unit tests like this:

    var MockLogger = require('simple-node-logger').mocks.MockLogger;

    var log = MockLogger.createLogger('MyCategory');

    log.info('this is a log statement');
    log.getLogEntries().length.should.equal( 1 );
    
MockLogger extends Logger and uses MockAppender to capture log entries.

## License

Apache 2.0

- - -
<p><small><em>Copyright © 2014-2015, rain city software | Version 0.92.21</em></small></p>
