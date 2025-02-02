# PostgreSQL Column Byte Packer

[![Build Status](https://travis-ci.com/braintree/pg_column_byte_packer.svg?branch=master)](https://travis-ci.org/braintree/pg_column_byte_packer/)

tl;dr: Provides facilities for laying out table column order to optimize for disk space usage both in ActiveRecord migrations and `pg_dump` generated SQL schema files. The general idea and relevant PostgreSQL internals are described in [On Rocks and Sand](https://www.2ndquadrant.com/en/blog/on-rocks-and-sand/).

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'pg_column_byte_packer'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install pg_column_byte_packer

## Usage

There are two ways you can use this library to byte-pack your tables' column layout:

### Re-ordering columns using ActiveRecord migrations

Loading the library automatically patches ActiveRecord's table creation tree walker to automatically re-order columns by alignment size. Therefore all `create_table` calls in ActiveRecord migrations executed after loading the gem will be byte-packing optimized.

Note: Because you need the full table definition to re-order columns, the most benefit occurs when the full table is created in one step (rather than added onto with repeated `add_column` migrations).

### Re-ordering SQL structure files (from `pg_dump`)

ActiveRecord defaults to saving your application's database structure to a `schema.rb` file (which is essentially all of the ActiveRecord migrations commands you'd need to generate the current state of the database). However you can also configure it to save a copy of the database structure in SQL format by setting `config.active_record.schema_format = :sql` in your `config/application.rb` file. With this configuration (and running against a PostgreSQL database) ActiveRecord executes the `pg_dump` utility (included in the PostgreSQL client tools) against your database to generate a `structure.sql` file.

If you have an existing `structure.sql` file (or any structure-only file generated by `pg_dump`), you can update that file to have byte-packed `CREATE TABLE` statements with the following:

```ruby
PgColumnBytePacker::PgDump.sort_columns_for_definition_file(
  "<path to structure.sql file>",
  connection: ActiveRecord::Base.connection
)
```

Note: an ActiveRecord connection object is required so that the library can properly determine the data types (and their respective alignment requirements) and column metadata (e.g., `DEFAULT` and `NOT NULL`).

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `bundle exec rspec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment. This project uses Appraisal to test against multiple versions of ActiveRecord; you can run the tests against all supported version with `bundle exec appraisal rspec`.

Running tests will automatically create a test database in the locally running Postgres server. You can find the connection parameters in `spec/spec_helper.rb`, but setting the environment variables `PGHOST`, `PGPORT`, `PGUSER`, and `PGPASSWORD` will override the defaults.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, commit the change, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org). To ignore local changes (say to `.ruby-version`) you can do `rake build release:source_control_push release:rubygem_push`.

Note: if while releasing the gem you get the error ``Your rubygems.org credentials aren't set. Run `gem push` to set them.`` you can more simply run `gem signin`.

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
