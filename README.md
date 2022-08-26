# rspec_n

rspec_n is a Ruby gem that makes it easy to run a project's RSpec test suite N times. You can customize the command that is used to start RSpec, or let rspec_n guess the best command (based on the files in your project). rspec_n is useful for finding repeatability issues in automated test suites.

![example](https://user-images.githubusercontent.com/2053901/53691471-c6956880-3d4c-11e9-8248-68bbb4c24786.png)

## Versioning Strategy

Releases are versioned using [SemVer 2.0.0](https://semver.org/spec/v2.0.0.html) with the following caveats:

1. Support for a Ruby version, that reaches EOL, is removed in a major or minor release.

## Supported Ruby Versions

Ruby 2.7, 3.0 and 3.1.

## Installation

Install by executing:

    $ gem install rspec_n

The gem will install an executable called `rspec_n`.

Add the following to your project's `.gitignore` to exclude the output generated by rspec_n from your project's repo:

    rspec_n_iteration.*

#### Usage in a Gemfile

If you want to add rspec_n to your Gemfile, use the `require: false` option so rspec_n files aren't loaded into your app. rspec_n doesn't provide any runtime benefit to apps and requiring it will add unnecessary code to your project. Also, rspec_n is designed as a standalone commandline tool and isn't tested for compatibility inside other apps.

```ruby
gem 'rspec_n', require: false
```

## Usage

The simplest way to run rspec_n is to give it a positive integer which tells it how many times to run RSpec:

    $ rspec_n 5

As each iteration completes, output is sent to the screen and dumped to a file. If you need to examine the detailed output of a run, it is available in the output files.

You can also list one or more paths to target specs. You can do anything you would normally do when giving RSpec paths:

    $ rspec_n 5 spec/path/to/something_spec.rb
    $ rspec_n 5 spec/path/to/folder spec/path/to/some/other/file_spec.rb
    $ rspec_n 5 spec/path/to/folder
    $ rspec_n 5 spec/path/to/something_spec.rb:5

By default, `--order rand` is sent to RSpec to force it to run specs in random order. You can use a `defined` order if you don't want randomness:

    $ rspec_n 5 --order defined

Or, let the configuration files in the project determine the order:

    $ rspec_n 5 --order project

#### Config file

You can create `.rspec_n` file in your project's root, and add command line options to it. They'll be used if `rspec_n` is run without options.Options are any arguments that start with `-` or `--`. For example, `rspec_n 10 spec/some_spec.rb` will make rspec_n consider `.rspec_n`, but `rspec_n 10 -c "rm -rf /tmp/* && bundle exec rspec"` won't.

Example file format:

```
--no-file
-s
--order defined
-c "rm -rf tmp && bundle exec rspec"
```

The config file can be multi line or single line.

#### Automatic Command Selection

rspec_n inspects files in your project so it can pick the best way to start RSpec. If it can't make an educated guess, it will use `bundle exec rspec` as the base command and add any extra information you've entered on the command line (like the order or paths). The following is a list of project types that rspec_n can identify and the associated commands it will try to execute:

1. Ruby on Rails Applications: `DISABLE_DATABASE_ENVIRONMENT_CHECK=1 RAILS_ENV=test bundle exec rake db:drop db:create db:migrate && bundle exec rspec`.
2. Everything else: `bundle exec rspec`.

#### Use Custom Command to Start RSpec

Use the `-c` option if you want to specify your own command. The following example deletes the `tmp` folder before starting RSpec:

    $ rspec_n 5 -c 'rm -rf tmp && bundle exec rspec'

There are couple points to consider:

1. Wrap your entire command in a single or double quoted string.
1. Use the `&&` operator to join commands.
1. rspec_n was created to help discover flaky test suites so it adds `--order rand` to your custom command by default. You must use `--order defined` or `--order project` if you want something else.

#### Control File Output

rspec_n writes output for each iteration in a sequence of files `rspec_n_iteration.1`, `rspec_n_iteration.2`, etc... If you want to disable this, add the `--no-file` option to the command.

    $ rspec_n 5 --no-file

**Note:** rspec_n deletes all files matching `rspec_n_iteration.*` when it starts, so be sure to move those files to another location if you want to save them.

#### Stop on First Failure

You can tell rspec_n to abort the first time an iteration fails by using the `-s` flag. All remaining iterations are skipped.

## Understanding the Results

rspec_n uses the `STDOUT`, `STDERR` and `EXIT STATUS` of the `rspec` command to decide if the run passed or failed. The run is considered successful if RSpec returns an `EXIT STATUS` of `0` **regardless of any content in the STDERR stream**. rspec_n considers the run to be a failure if RSpec's `EXIT STATUS` `> 0`.

There are times when `STDERR` has content, even though RSpec returns an `EXIT STATUS` of `0`. This frequently happens with deprecation notices and RSpec itself will pass the test suite in this situation. Also, it's not uncommon for code to write messages to `STDERR` that look like errors, but not actually cause RSpec to fail a spec example. If rspec_n is able to detect this situation, it adds `(Warning)` to the results to indicate there's something extra in the `STDERR` that you might want to investigate.

The results of each run (number of tests, number of failures, etc...) are determined by parsing `STDOUT` and extracting information from the line that says `xyz examples, xyz failures, xyz pending`.

## Development

### Setup

Install dependencies:

    $ bin/setup

That's it!

### Run Automated Tests

Run the automated test suite with:

    $ rake spec

### Start a Console

Choose one of the following to start a conole with the gem loaded:

```bash
$ bin/console  # for Pry
$ rake console # for IRB
```

## Contributing

Contributions are welcome. Please us the following process when submitting work:

1. Create an issue on the [issue page](https://github.com/roberts1000/rspec_n/issues) that targets a single problem/enhancement. All PRs should be tied to an issue.
1. Fork the project.
1. Create a branch. The name of the branch should start with the issue number that the branch will address.
1. Submit the PR. In the PR comment (in the UI), add `Closes #xyz` or `Supports #xyz` where `xyz` is the issue number that the PR addresses. The title of the PR (in the UI) should start with `[PR for #xyz] `.
