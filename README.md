![SSHKit Logo](https://raw.github.com/leehambley/sshkit/master/examples/images/logo.png)

**SSHKit** is a toolkit for running commands in a structured way on one or
more servers.

[![Build Status](https://travis-ci.org/capistrano/sshkit.png?branch=master)](https://travis-ci.org/capistrano/sshkit)
[![Dependency Status](https://gemnasium.com/leehambley/sshkit.png)](https://gemnasium.com/leehambley/sshkit)

## How might it work?

The typical use-case looks something like this:

```ruby
require 'sshkit/dsl'

on %w{1.example.com 2.example.com}, in: :sequence, wait: 5 do
  within "/opt/sites/example.com" do
    as :deploy  do
      with rails_env: :production do
        rake   "assets:precompile"
        runner "S3::Sync.notify"
        execute "node", "socket_server.js"
      end
    end
  end
end
```

One will notice that it's quite low level, but exposes a convenient API, the
`as()`/`within()`/`with()` are nestable in any order, repeatable, and stackable.

When used inside a block in this way, `as()` and `within()` will guard
the block they are given with a check.

In the case of `within()`, an error-raising check will be made that the directory
exists; for `as()` a simple call to `sudo su -<user> whoami` wrapped in a check for
success, raising an error if unsuccessful.

The directory check is implemented like this:

    if test ! -d <directory>; then echo "Directory doesn't exist" 2>&1; false; fi

And the user switching test implemented like this:

    if ! sudo su <user> -c whoami > /dev/null; then echo "Can't switch user" 2>&1; false; fi

According to the defaults, any command that exits with a status other than 0
raises an error (this can be changed). The body of the message is whatever was
written to *stdout* by the process. The `1>&2` redirects the standard output
of echo to the standard error channel, so that it's available as the body of
the raised error.

Helpers such as `runner()` and `rake()` which expand to `execute(:rails, "runner", ...)` and
`execute(:rake, ...)` are convenience helpers for Ruby, and Rails based apps.

## Parallel

Notice on the `on()` call the `in: :sequence` option, the following will do
what you might expect:

```ruby
on(in: :parallel) { ... }
on(in: :sequence, wait: 5) { ... }
on(in: :groups, limit: 2, wait: 5) { ... }
```

The default is to run `in: :parallel` which has no limit. If you have 400 servers,
this might be a problem and you might better look at changing that to run in
`groups`, or `sequence`.

Groups were designed in this case to relieve problems (mass Git checkouts)
where you rely on a contested resource that you don't want to DDOS by hitting
it too hard.

Sequential runs were intended to be used for rolling restarts, amongst other
similar use-cases.

## Synchronisation

The `on()` block is the unit of synchronisation, one `on()` block will wait
for all servers to complete before it returns.

For example:

```ruby
all_servers = %w{one.example.com two.example.com three.example.com}
site_dir    = '/opt/sites/example.com'

# Let's simulate a backup task, assuming that some servers take longer
# then others to complete
on all_servers do |host|
  within site_dir do
    execute :tar, '-czf', "backup-#{host.hostname}.tar.gz", 'current'
    # Will run: "/usr/bin/env tar -czf backup-one.example.com.tar.gz current"
  end
end

# Now we can do something with those backups, safe in the knowledge that
# they will all exist (all tar commands exited with a success status, or
# that we will have raised an exception if one of them failed.
on all_servers do |host|
  in site_dir do
    backup_filename = "backup-#{host.hostname}.tar.gz"
    target_filename = "backups/#{Time.now.utc.iso8601}/#{host.hostname}.tar.gz"
    puts capture(:s3cmd, 'put', backup_filename, target_filename)
  end
end
```

## The Command Map

It's often a problem that programmatic SSH sessions don't have the same environment
variables as interactive sessions.

A problem often arises when calling out to executables expected to be on
the `$PATH`.  Under conditions without dotfiles or other environmental
configuration, `$PATH` may not be set as expected, and thus executables are not found where expected.

To try and solve this there is the `with()` helper which takes a hash of variables and makes them
available to the environment.

```ruby
with path: '/usr/local/bin/rbenv/shims:$PATH' do
  execute :ruby, '--version'
end
```

Will execute:

    ( PATH=/usr/local/bin/rbenv/shims:$PATH /usr/bin/env ruby --version )

By contrast, the following won't modify the command at all:


```ruby
with path: '/usr/local/bin/rbenv/shims:$PATH' do
  execute 'ruby --version'
end
```

Will execute, without mapping the environmental variables, or querying the command map:

    ruby --version

(This behaviour is sometimes considered confusing, but it has mostly to do with shell escaping: in the case of whitespace in your command, or newlines, we have no way of reliably composing a correct shell command from the input given.)

**Often more preferable is to use the *command map*.**

The *command map* is used by default when instantiating a *Command* object

The *command map* exists on the configuration object, and in principle is
quite simple, it's a *Hash* structure with a default key factory block
specified, for example:

```ruby
puts SSHKit.config.command_map[:ruby]
# => /usr/bin/env ruby
```

To make clear the environment is being deferred to, the `/usr/bin/env` prefix is applied to all commands.
Although this is what happens anyway when one would simply attempt to execute `ruby`, making it
explicit hopefully leads people to explore the documentation.

One can override the hash map for individual commands:

```ruby
SSHKit.config.command_map[:rake] = "/usr/local/rbenv/shims/rake"
puts SSHKit.config.command_map[:rake]
# => /usr/local/rbenv/shims/rake
```

Another opportunity is to add command prefixes:

```ruby
SSHKit.config.command_map.prefix[:rake].push("bundle exec")
puts SSHKit.config.command_map[:rake]
# => bundle exec rake

SSHKit.config.command_map.prefix[:rake].unshift("/usr/local/rbenv/bin exec")
puts SSHKit.config.command_map[:rake]
# => /usr/local/rbenv/bin exec bundle exec rake
```

One can also override the command map completely, this may not be wise, but it
would be possible, for example:

```ruby
SSHKit.config.command_map = Hash.new do |hash, command|
  hash[command] = "/usr/local/rbenv/shims/#{command}"
end
```

This would effectively make it impossible to call any commands which didn't
provide an executable in that directory, but in some cases that might be
desirable.

*Note:* All keys should be symbolised, as the *Command* object will symbolize it's
first argument before attempting to find it in the *command map*.

## Interactive commands
> (BETA) (Added in version #.##)

By default, commands against remote servers are run in a *non-login, non-interactive* ssh session.
This is by design, to try and isolate the environment and make sure that things work as expected,
regardless of any changes that might happen on the server side. This means that,
although the server may have prompted you, and be waiting for it,
**you cannot send data to the server by typing into your terminal window**.
Wherever possible, you should call commands in a way that doesn't require interaction
(eg by specifying all options as command arguments).

However in some cases, you may want to programmatically drive interaction with a command
and this can be achieved by specifying an `:interaction_handler` option when you `execute`, `capture` or `test` a command.

**It is not necessary, or desirable to enable `Netssh.config.pty` to use the `interaction_handler` option.
Only enable `Netssh.config.pty` if the command you are calling won't work without a pty.**

An `interaction_handler` is an object which responds to `on_stdout(stdout, channel, command)` or `on_stderr(stderr, channel, command)`.
The `interaction_handler`'s methods will be called once per line of `stdout` or `stderr` from the server and
can send data back to the server using the `channel` parameter.
This allows scripting of command interaction by responding to `stdout` or `stderr` lines with any input required.

For example, an interaction handler to change the password of your linux user using the `passwd` command could look like this:

```ruby
class PasswdInteractionHandler
  def on_stderr(channel, stderr, command)
    puts stderr
    case stderr
      when '(current) UNIX password: '
        channel.send_data("old_pw\n")
      when 'Enter new UNIX password: ', 'Retype new UNIX password: '
        channel.send_data("new_pw\n")
      when 'passwd: password updated successfully'
      else
        raise "Unexpected stderr #{stderr}"
    end
  end
end

# ...

execute(:passwd, interaction_handler: PasswdInteractionHandler.new)
```

Often, you want to map directly from an output string returned by the server to the corresponding input string (as in the case above).
For this case you can use a `MappingInteractionHandler`:

```ruby
execute(:passwd, interaction_handler: MappingInteractionHandler.new(
  '(current) UNIX password: ' => 'old_pw',
  'Enter new UNIX password: ' => 'new_pw',
  'Retype new UNIX password: ' => 'new_pw',
  'passwd: password updated successfully' : nil # For stdout/stderr which can be ignored, map a nil input
))
```

`MappingInteractionHandler`s map output from `stdout` or `stderr` using the same single map.
If no mapping is found, a warning will show the string you need to add to your map:

`Unable to find interaction handler mapping for stdout: "Server output\n" so no response was sent`

`MappingInteractionHandler`s are stateless, so you can assign one to a constant and reuse it:

```ruby
ENTER_PASSWORD = MappingInteractionHandler.new('Please Enter Password' : 'some_password')

# ...

execute(:first_command, interaction_handler: ENTER_PASSWORD)
execute(:second_command, interaction_handler: ENTER_PASSWORD)
```

`:interaction_handler`s can also be stateful if you need them to be and you can capture user data with `$stdin`:

```ruby
class PromptUserForPasswordAndCache
  @password_cache = {}

  def on_stderr(channel, stderr, command)
    if data =~ /Sorry.*\stry\sagain/
      @password_cache[command.host] = nil
    end
    if data =~ /password.*:/
      pass = password_cache[command.host]
      unless pass
        pass = $stdin.noecho(&:gets)
        password_cache[command.host] = pass
      end
      ch.send_data(pass)
    end
  end
end

# ...

prompt_or_use_cached = PromptUserForPasswordAndCache.new

execute(:first_command, interaction_handler: prompt_or_use_cached)
execute(:second_command, interaction_handler: prompt_or_use_cached)

```

When using the `Netssh` backend, the `channel` parameter of `on_stdout(channel, stdout, command)` or
`on_stderr(channel, stderr, command)` is a
[Net::SSH Channel](http://net-ssh.github.io/ssh/v2/api/classes/Net/SSH/Connection/Channel.html).
When using the `Local` backend, it is a [ruby IO](http://ruby-doc.org/core/IO.html) object.
If you need to support both sorts of backends with the same interaction handler,
you need to call methods the appropriate API depending on the channel type.
One approach is to detect the presence of the API methods you need -
eg `channel.respond_to?(:send_data) # Net::SSH channel` and `channel.respond_to?(:write) # IO`.
See the `MappingInteractionHandler` for an example of this.

## Output Handling

![Example Output](https://raw.github.com/leehambley/sshkit/master/examples/images/example_output.png)

By default, the output format is set to `:pretty`:

```ruby
SSHKit.config.format = :pretty
```

However, if you prefer minimal output, `:dot` format will simply output red or green dots based on the success or failure of operations.

To output directly to $stdout without any formatting, you can use:

```ruby
SSHKit.config.output = $stdout
```

#### Custom formatters

Want custom output formatting? Here's what you have to do:

1. Write a new formatter class in the `SSHKit::Formatter` namespace. As an example, check out the default [pretty](https://github.com/capistrano/sshkit/blob/master/lib/sshkit/formatters/pretty.rb) formatter.
1. Set the output format as described above. E.g. if your new formatter is called `Foobar`:

        SSHKit.config.format = :foobar


## Output Verbosity

By default calls to `capture()` and `test()` are not logged, they are used
*so* frequently by backend tasks to check environmental settings that it
produces a large amount of noise. They are tagged with a verbosity option on
the `Command` instances of `Logger::DEBUG`. The default configuration for
output verbosity is available to override with `SSHKit.config.output_verbosity=`,
and defaults to `Logger::INFO`.

At present the `Logger::WARN`, `ERROR` and `FATAL` are not used.

## Connection Pooling

SSHKit uses a simple connection pool (enabled by default) to reduce the
cost of negotiating a new SSH connection for every `on()` block. Depending on
usage and network conditions, this can add up to a significant time savings.
In one test, a basic `cap deploy` ran 15-20 seconds faster thanks to the
connection pooling added in recent versions of SSHKit.

To prevent connections from "going stale", an existing pooled connection will
be replaced with a new connection if it hasn't been used for more than 30
seconds. This timeout can be changed as follows:

```ruby
SSHKit::Backend::Netssh.pool.idle_timeout = 60 # seconds
```

If you suspect the connection pooling is causing problems, you can disable the
pooling behaviour entirely by setting the idle_timeout to zero:

```ruby
SSHKit::Backend::Netssh.pool.idle_timeout = 0 # disabled
```

## Tunneling and other related SSH themes

In order to do special gymnasitcs with SSH, tunneling, aliasing, complex options, etc with SSHKit it is possible to use [the underlying Net::SSH API](https://github.com/capistrano/sshkit/blob/master/EXAMPLES.md#setting-global-ssh-options) however in many cases it is preferred to use the system SSH configuration file at [`~/.ssh/config`](http://man.cx/ssh_config). This allows you to have personal configuration tied to your machine that does not have to be committed with the repository. If this is not suitable (everyone on the team needs a proxy command, or some special aliasing) a file in the same format can be placed in the project directory at `~/yourproject/.ssh/config`, this will be merged with the system settings in `~/.ssh/config`, and with any configuration specified in [`SSHKit::Backend::Netssh.ssh_options`](https://github.com/capistrano/sshkit/blob/master/lib/sshkit/backends/netssh.rb#L133).

These system level files are the preferred way of setting up tunneling and proxies because the system implementations of these things are faster and better than the Ruby implementations you would get if you were to configure them through Net::SSH. In cases where it's not possible (Windows?), it should be possible to make use of the Net::SSH APIs to setup tunnels and proxy commands before deferring control to Capistrano/SSHKit..

## SSHKit Related Blog Posts

[SSHKit Gem Basics](http://www.rubyplus.com/articles/591)

[SSHKit Gem Part 2](http://www.rubyplus.com/articles/601)

[Embedded Capistrano with SSHKit](http://ryandoyle.net/posts/embedded-capistrano-with-sshkit/)
