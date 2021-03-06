## Changelog

This file is written in reverse chronological order, newer releases will
appear at the top.

## `master` (Unreleased)

  * Add your entries below here, remember to credit yourself however you want
    to be credited!
  * Added support for :interaction_handler option on commands. @robd
  * Removed partially supported 'trace' log level. @robd
  * No longer strip whitespace or newlines in `capture` method on Netssh backend. @robd
    * This is to make the `Local` and `Netssh` backends consistent (they diverged at 7d15a9a)
    * If you need the old behaviour back, call `.strip` (or `.chomp`) on the captured string i.e. `capture(:my_command).strip`
  * Simplified backend hierarchy. @robd
    * Moved duplicate implementations of `make`, `rake`, `test`, `capture`, `background` on to `Abstract` backend.
    * Backend implementations now only need to implement `execute_command`, `upload!` and `download!`
    * Removed `Printer` from backend hierarchy for `Local` and `Netssh` backends (they now just extend `Abstract`)
    * Removed unused `Net::SSH:LogLevelShim`

## 1.7.1

  * Fix a regression in 1.7.0 that caused command completion messages to be removed from log output. @mattbrictson

## 1.7.0

  * Update Vagrantfile to use multi-provider Hashicorp precise64 box - remove URLs. @townsen
  * Merge host ssh_options and Netssh defaults @townsen
    Previously if host-level ssh_options were defined the Netssh defaults
    were ignored.
  * Merge host ssh_options and Netssh defaults
  * Fixed race condition where output of failed command would be empty. @townsen
    Caused random failures of `test_execute_raises_on_non_zero_exit_status_and_captures_stdout_and_stderr`
    Also fixes output handling in failed commands, and generally buggy output.
  * Remove override of backtrace() and backtrace_locations() from ExecuteError. @townsen
    This interferes with rake default behaviour and creates duplicate stacktraces.
  * Allow running local commands using `on(:local)`
  * Implement the upload! and download! methods for the local backend

## 1.6.0

  * Fix colorize to use the correct API (@fazibear)
  * Lock colorize (sorry guys) version at >= 0.7.0

## 1.6.0 (Yanked, because of colorize.)

  * Force dependency on colorize v0.6.0
  * Add your entries here, remember to credit yourself however you want to be
    credited!
  * Remove strip from capture to preserve whitespace. Nick Townsend
  * Add vmware_fusion Vagrant provider. Nick Townsend
  * Add some padding to the pretty log formatter

## 1.5.1

  * Use `sudo -u` rather than `sudo su` to switch users. Mat Trudel

## 1.5.0

  * Deprecate background helper - too many badly behaved pseudo-daemons. Lee Hambley
  * Don't colourize unless $stdout is a tty. Lee Hambley
  * Remove out of date "Known Issues" section from README. Lee Hambley
  * Dealy variable interpolation inside `as()` block. Nick Townsend
  * Fixes for functional tests under modern Vagrant. Lewis Marshal
  * Fixes for connection pooling. Chris Heald
  * Add `localhost` hostname to local backend. Adam Mckaig
  * Wrap execptions to include hostname. Brecht Hoflack
  * Remove `shellwords` stdlib dependency Bruno Sutic
  * Remove unused `cooldown` accessor. Bruno Sutic
  * Replace Term::ANSIColor with a lighter solution. Tom Clements
  * Documentation fixes. Matt Brictson

## 1.4.0

https://github.com/capistrano/sshkit/compare/v1.3.0...v1.4.0

  * Removed `invoke` alias for [`SSHKit::Backend::Printer.execute`](https://github.com/capistrano/sshkit/blob/master/lib/sshkit/backends/printer.rb#L20). This is to prevent collisions with
  methods in capistrano with similar names, and to provide a cleaner API. See [capistrano issue 912](https://github.com/capistrano/capistrano/issues/912) and [issue 107](https://github.com/capistrano/sshkit/issues/107) for more details.
  * Connection pooling now uses a thread local to store connection pool, giving each thread its own connection pool. Thank you @mbrictson see [#101](https://github.com/capistrano/sshkit/pull/101) for more.
  * Command map indifferent towards strings and symbols thanks to @thomasfedb see [#91](https://github.com/capistrano/sshkit/pull/91)
  * Moved vagrant wrapper to `support` directory, added ability to run tests with vagrant using ssh. @miry see [#64](https://github.com/capistrano/sshkit/pull/64)
  * Removed unnecessary require `require_relative '../sshkit'` in `lib/sshkit/dsl.rb` prevents warnings thanks @brabic.
  * Doc fixes thanks @seanhandley @vojto

## 1.3.0

https://github.com/capistrano/sshkit/compare/v1.2.0...v1.3.0

  * Connection pooling. SSH connections are reused across multiple invocations
    of `on()`, which can result in significant performance gains. See:
    https://github.com/capistrano/sshkit/pull/70. Matt @mbrictson Brictson.
  * Fixes to the Formatter::Dot and to the formatter class name resolver. @hab287:w
  * Added the license to the Gemspec. @anatol.
  * Fix :limit handling for the `in: :groups` run mode. Phil @phs Smith
  * Doc fixes @seanhandley, @sergey-alekseev.

## 1.2.0

https://github.com/capistrano/sshkit/compare/v1.1.0...v1.2.0

  * Support picking up a project local SSH config file, if a SSH config file
    exists at ./.ssh/config it will be merged with the ~/.ssh/config. This is
    ideal for defining project-local proxies/gateways, etc. Thanks to Alex
    @0rca Vzorov.
  * Tests and general improvements to the Printer backends (mostly used
    internally). Thanks to Michael @miry Nikitochkin.
  * Update the net-scp dependency version. Thanks again to Michael @miry
    Nikitochkin.
  * Improved command map. This feature allows mapped variables to be pushed
    and unshifted onto the mapping so that the Capistrano extensions for
    rbenv and bundler, etc can work together. For discussion about the reasoning
    see https://github.com/capistrano/capistrano/issues/639 and
    https://github.com/capistrano/sshkit/pull/45. A big thanks to Kir @kirs
    Shatrov.
  * `test()` and `capture()` now behave as expected inside a `run_locally` block
    meaning that they now run on your local machine, rather than erring out. Thanks
    to Kentaro @kentaroi Imai.
  * The `:wait` option is now successfully passed to the runner now. Previously the
    `:wait` option was ignored. Thanks to Jordan @jhollinger Hollinger for catching
    the mistake in our test coverage.
  * Fixes and general improvements to the `download()` method which until now was
    quite naïve. Thanks to @chqr.

## 1.1.0

https://github.com/capistrano/sshkit/compare/v1.0.0...v1.1.0

  * Please see the Git history. `git rebase` ate our changelog (we should have been
    more careful)

## 1.0.0

  * The gem now supports a run_locally, although it's nothing to do with SSH,
    it makes a nice API. There are examples in the EXAMPLES.md.

## 0.0.34

  * Allow the setting of global SSH options on the `backend.ssh` as a hash,
    these options are the same as Net::SSH configure expects. Thanks to Rafał
    @lisukorin Lisowski

## 0.0.32

  * Lots of small changes since 0.0.27.
  * Particularly working around a possible NaN issue when uploading
    comparatively large files.

## 0.0.27

  * Don't clobber SSH options with empty values. This allows Net::SSH to
    do the right thing most of the time, and look into the SSH configuration
    files.

## 0.0.26

  * Pretty output no longer prints white text. ("Command.....")
  * Fixed a double-output bug, where upon receiving the exit status from a
    remote command, the last data packet that it sent would be re-printed
    by the pretty formatter.
  * Integration tests now use an Ubuntu Precise 64 Vagrant base box.
  * Additional host declaration syntax, `SSHKit::Host` can now take a hash of
    host properties in addition to a number of new (common sense) DSN type
    syntaxes.
  * Changes to the constants used for logging, we no longer re-define a
    `Logger::TRACE` constant on the global `Logger` class, rather everyhing
    now uses `SSHKit::Logger` (Thanks to Rafa Garcia)
  * Various syntax and documentation fixes.

## 0.0.25

  * `upload!` and `download!` now log to different levels depending on
    completion percentage. When the upload is 0 percent complete or a number
    indivisible by 10, the message is logged to `Logger::DEBUG` otherwise the
    message is logged to `Logger::INFO`, this should mean that normal users at
    a sane log level should see upload progress jump to `100%` for small
    files, and otherwise for larger files they'll see output every `10%`.

## 0.0.24

  * Pretty output now streams stdout and stderr. Previous versions would
    append (`+=`) chunks of data written by the remote host to the `Command`
    instance, and the `Pretty` formatter would only print stdout/stderr if the
    command was `#complete?`. Historically this lead to issues where the
    remote process was blocking for input, had written the prompt to stdout,
    but it was not visible on the client side.

    Now each time the command is passed to the output stream, the
    stdout/stderr are replaced with the lines returned from the remote server
    in this chunk. (i.e were yielded to the callback block). Commands now have
    attribute accessors for `#full_stdout` and `#full_stderr` which are appended
    in the way that `#stdout` and `#stderr` were previously.

    This should be considered a private API, and one should beware of relying
    on `#full_stdout` or `#full_stderr`, they will likely be replaced with a
    cleaner soltion eventually.

  * `upload!` and `download!` now print progress reports at the `Logger::INFO`
     verbosity level.

## 0.0.23

  * Explicitly rely on `net-scp` gem.

## 0.0.22

  * Added naïve implementations of `upload!()` and `download!()` (syncoronous) to
    the Net::SSH backend. See `EXAMPLES.md` for more extensive usage examples.

    The `upload!()` method can take a filename, or an `IO`, this reflects the way
    the underlying Net::SCP implementation works. The same is true of
    `download!()`, when called with a single argument it captures the file's
    contents, otherwise it downloads the file to the local disk.

        on hosts do |host|
          upload!(StringIO.new('some-data-here'), '~/.ssh/authorized_keys')
          upload!('~/.ssh/id_rsa.pub', '~/.ssh/authorized_keys')
          puts download!('/etc/monit/monitrc')
          download!('/etc/monit/monitrc', '~/monitrc')
        end

## 0.0.21

  * Fixed an issue with default formatter
  * Modified `SSHKit.config.output_verbosity=` to accept different objects:

        SSHKit.config.output_verbosity = Logger::INFO
        SSHKit.config.output_verbosity = :info
        SSHKit.config.output_verbosity = 1

## 0.0.20

 * Fixed a bug where the log level would be assigned, not compared in the
   pretty formatter, breaking the remainder of the output verbosity.

## 0.0.19

 * Modified the `Pretty` formatter to include the log level in front of
   executed commands.

 * Modified the `Pretty` formatter not to print stdout and stderr by default,
   the log level must be raised to Logger::DEBUG to see the command outputs.

 * Modified the `Pretty` formatter to use `Command#to_s` when printing the
   command, this prints the short form (without modifications/wrappers applied
   to the command for users, groups, directories, umasks, etc).

## 0.0.18

 * Enable `as()` to take either a string/symbol as previously, but also now
   accepts a hash of `{user: ..., group: ...}`. In case that your host system
   supports the command `sg` (`man 1 sg`) to switch your effective group ID
   then one can work on files as a team group user.

        on host do |host|
          as user: :peter, group: griffin do
            execute :touch, 'somefile'
          end
        end

    will result in a file with the following permissions:

        -rw-r--r-- 1 peter griffin 0 Jan 27 08:12 somefile

    This should make it much easier to share deploy scripts between team
    members.

    **Note:** `sg` has some very strict user and group password requirements
    (the user may not have a password (`passwd username -l` to lock an account
    that already has a password), and the group may not have a password.)

    Additionally, and unsurprisingly *the user must also be a member of the
    group.*

    `sg` was chosen over `newgrp` as it's easier to embed in a one-liner
    command, `newgrp` could be used with a heredoc, but my research suggested
    that it might be better to use sg, as it better represents my intention, a
    temporary switch to a different effective group.

 * Fixed a bug with environmental variables and umasking introduced in 0.0.14.
   Since that version the environmental variables were being presented to the
   umask command's subshell, and not to intended command's subshell.

       incorrect: `ENV=var umask 002 && env`
       correct:   `umask 002 && ENV=var env`

 * Changed the exception handler, if a command returns with a non-zero exit
   status then the output will be prefixed with the command name and which
   channel any output was written to, for example:

       Command.new("echo ping; false")
       => echo stdout: ping
          echo stderr: Nothing written

   In this contrived example that's more or less useless, however with badly
   behaved commands that write errors to stdout, and don't include their name
   in the program output, it can help a lot with debugging.

## 0.0.17

 * Fixed a bug introduced in 0.0.16 where the capture() helper returned
   the name of the command that had been run, not it's output.

 * Classify the pre-directory switch, and pre-user switch command guards
   as having a DEBUG log level to exclude them from the logs.

## 0.0.16

 * Fixed a bug introduced in 0.0.15 where the capture() helper returned
   boolean, discarding any output from the server.

## 0.0.15

 * `Command` now takes a `verbosity` option. This defaults to `Logger::INFO`
   and can be set to any of the Ruby logger level constants. You can also set
   it to the symbol `:debug` (and friends) which will be expanded into the correct
   constants.

   The log verbosity level is set to Logger::INFO by default, and can be
   overridden by setting `SSHKit.config.output_verbosity = Logger::{...}`,
   pick a level that works for you.

   By default `test()` and `capture()` calls are surpressed, and not printed
   by the pretty logger as of this version.

## 0.0.14

 * Umasks can now be set on `Command` instances. It can be set globally with
   `SSHKit.config.umask` (default, nil; meaning take the system default). This
   can be used to set, for example a umask of `007` for allowing users with
   the same primary group to share code without stepping on eachother's toes.

## 0.0.13

 * Correctly quote `as(user)` commands, previously it would expand to:
   `sudo su user -c /usr/bin/env echo "Hello World"`, in which the command to
   run was taken as simply `/usr/bin/env`. By quoting all arguments it should
   now work as expected. `sudo su user -c "/usr/bin/env echo \""Hello World\""`

## 0.0.12

 * Also print anything the program wrote to stdout when the exit status is
   non-zero and the command raises an error. (assits debugging badly behaved
   programs that fail, and write their error output to stdout.)

## 0.0.11

 * Implementing confuguration objects on the backends (WIP, undocumented)
 * Implement `SSHKit.config.default_env`, a hash which can be modified and
   will act as a global `with`.
 * Fixed #9 (with(a: 'b', c: 'c') being parsed as `A=bC=d`. Now properly space
   separated.
 * Fixed #10 (overly aggressive shell escaping), one can now do:
   `with(path: 'foo:$PATH') without the $ being escaped too early.

## 0.0.10

* Include more attributes in `Command#to_hash`.

## 0.0.9

* Include more attributes in `Command#to_hash`.

## 0.0.8

* Added DSL method `background()` this sends a task to the background using
  `nohup` and redirects it's output to `/dev/null` so as to avoid littering
  the filesystem with `nohup.out` files.

**Note:** Backgrounding a task won't work as you expect if you give it a
string, that is you must do `background(:sleep, 5)` and not `background("sleep 5")`
according to the rules by which a command is not processed in any way **if it
contains a spaca character in it's first argument**.

Usage Example:

    on hosts do
      background :rake, "assets:precompile" # typically takes 5 minutes!
    end

**Further:** Many programs are badly behaved and no not work well with `nohup`
it has to do with the way nohup works, reopening the processe's file
descriptors and redirecting them. Programs that re-open, or otherwise
manipulate their own file descriptors may lock up when the SSH session is
disconnected, often they block writing to, or reading from stdin/out.

## 0.0.7

* DSL method `execute()` will now raise `SSHKit::Command::Failed` when the
  exit status is non-zero. The message of the exception will be whatever the
  process had written to stdout.
* New DSL method `test()` behaves as `execute()` used to until this version.
* `Command` now raises an error in `#exit_status=()` if the exit status given
  is not zero. (see below)
* All errors raised by error conditions of SSHKit are defined as subclasses of
  `SSHKit::StandardError` which is itself a subclass of `StandardError`.

The `Command` objects can be set to not raise, by passing `raise_on_non_zero_exit: false`
when instantiating them, this is exactly what `test()` does internally.

Example:

    on hosts do |host
      if test "[ -d /opt/sites ]" do
        within "/opt/sites" do
          execute :git, :pull
        end
      else
        execute :git, :clone, 'some-repository', '/opt/sites'
      end
    end

## 0.0.6

* Support arbitrary properties on Host objects. (see below)

Starting with this version, the `Host` class supports arbitrary properties,
here's a proposed use-case:

    servers = %w{one.example.com two.example.com
                 three.example.com four.example.com}.collect do |s|
      h = SSHKit::Host.new(s)
      if s.match /(one|two)/
        h.properties.roles = [:web]
      else
        h.properties.roles = [:app]
      end
    end

    on servers do |host|
      if host.properties.roles.include?(:web)
        # Do something pertinent to web servers
      elsif host.properties.roles.include?(:app)
        # Do something pertinent to application servers
      end
    end

Naturally, this is a contrived example, the `#properties` attribute on the
Host instance is implemented as an [`OpenStruct`](http://ruby-doc.org/stdlib-1.9.3/libdoc/ostruct/rdoc/OpenStruct.html) and
will behave exactly as such.

## 0.0.5

* Removed configuration option `SSHKit.config.format` (see below)
* Removed configuration option `SSHKit.config.runner` (see below)

The format should now be set by doing:

    SSHKit.config.output = File.open('/dev/null')
    SSHKit.config.output = MyFormatterClass.new($stdout)

The library ships with three formatters, `BlackHole`, `Dot` and `Pretty`.

The default is `Pretty`, but can easily be changed:

    SSHKit.config.output = SSHKit::Formatter::Pretty.new($stdout)
    SSHKit.config.output = SSHKit::Formatter::Dot.new($stdout)
    SSHKit.config.output = SSHKit::Formatter::BlackHole.new($stdout)

The one and only argument to the formatter is the *String/StringIO*ish object
to which the output should be sent. (It should be possible to stack
formatters, or build a multi-formatter to log, and stream to the screen, for
example)

The *runner* is now set by `default_options` on the Coordinator class. The
default is still *:parallel*, and can be overridden on the `on()` (or
`Coordinator#each`) calls directly.

There is no global way to change the runner style for all `on()` calls as of
version `0.0.5`.

## 0.0.4

* Rename the ConnectionManager class to Coordinator, connections are handled
  in the backend, if it needs to create some connections.

## 0.0.3

* Refactor the runner classes into an abstract heirarchy.

## 0.0.2

* Include a *Pretty* formatter
* Modify example to use Pretty formatter.
* Move common behaviour to an abstract formatter.
* Formatters no longer inherit StringIO

## 0.0.1

First release.
