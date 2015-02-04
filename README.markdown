Tod
===

Supplies TimeOfDay class that includes parsing, strftime, comparison, and
arithmetic.

Supplies Shift to represent a period of time, using a beginning and ending TimeOfDay. Allows to calculate its duration and
to determine if a TimeOfDay is included inside the shift. For nightly shifts (when beginning time is greater than ending time),
it supposes the shift ends the following day.

Installation
============

    gem install tod

Examples
========

Loading Tod
-----------

    require 'tod'

Creating from hour, minute, and second
--------------------------------------

    TimeOfDay.new 8                                # => 08:00:00
    TimeOfDay.new 8, 15, 30                        # => 08:15:30

Parsing text
------------

Strings only need to contain an hour. Minutes, seconds, AM or PM, and colons
are all optional.

    TimeOfDay.parse "8"                            # => 08:00:00
    TimeOfDay.parse "8am"                          # => 08:00:00
    TimeOfDay.parse "8pm"                          # => 20:00:00
    TimeOfDay.parse "8p"                           # => 20:00:00
    TimeOfDay.parse "9:30"                         # => 09:30:00
    TimeOfDay.parse "15:30"                        # => 15:30:00
    TimeOfDay.parse "3:30pm"                       # => 15:30:00
    TimeOfDay.parse "1230"                         # => 12:30:00
    TimeOfDay.parse "3:25:58"                      # => 03:25:58
    TimeOfDay.parse "515p"                         # => 17:15:00
    TimeOfDay.parse "151253"                       # => 15:12:53
    TimeOfDay.parse "noon"                         # => 12:00:00
    TimeOfDay.parse "midnight"                     # => 00:00:00

TimeOfDay.parse raises an ArgumentError is the argument to parse is not
parsable. TimeOfDay.try_parse will instead return nil if the argument is not
parsable.

    TimeOfDay.try_parse "3:30pm"                   # => 15:30:00
    TimeOfDay.try_parse "foo"                      # => nil

Values can be tested with TimeOfDay.parsable? to see if they can be parsed.

    TimeOfDay.parsable? "3:30pm"                   # => true
    TimeOfDay.parsable? "foo"                      # => false

Adding or subtracting time
-----------------------------

Seconds can be added to or subtracted TimeOfDay objects. Time correctly wraps
around midnight.

    TimeOfDay.new(8) + 3600                        # => 09:00:00
    TimeOfDay.new(8) - 3600                        # => 07:00:00
    TimeOfDay.new(0) - 30                          # => 23:59:30
    TimeOfDay.new(23,59,45) + 30                   # => 00:00:15

Comparing
--------------------

TimeOfDay includes Comparable.

    TimeOfDay.new(8) < TimeOfDay.new(9)            # => true
    TimeOfDay.new(8) == TimeOfDay.new(9)           # => false
    TimeOfDay.new(9) == TimeOfDay.new(9)           # => true
    TimeOfDay.new(10) > TimeOfDay.new(9)           # => true

Formatting
----------

Format strings are passed to Time#strftime.

    TimeOfDay.new(8,30).strftime("%H:%M")          # => "08:30"
    TimeOfDay.new(17,15).strftime("%I:%M %p")      # => "05:15 PM"
    TimeOfDay.new(22,5,15).strftime("%I:%M:%S %p") # => "10:05:15 PM"

Convenience methods for dates and times
---------------------------------------

Tod adds Date#on and Time#to_time_of_day. If you do not want the core extensions
then require 'tod/time_of_day' instead of 'tod'.

    tod = TimeOfDay.new 8, 30                       # => 08:30:00
    tod.on Date.today                               # => 2010-12-29 08:30:00 -0600
    Date.today.at tod                               # => 2010-12-29 08:30:00 -0600
    Time.now.to_time_of_day                         # => 16:30:43
    DateTime.now.to_time_of_day                     # => 16:30:43

Conversion method
-----------------

Tod provides a conversion method which will handle a variety of input types:

    TimeOfDay(TimeOfDay.new(8, 30))           # => 08:30:00
    TimeOfDay("09:45")                        # => 09:45:00
    TimeOfDay(Time.new(2014, 1, 1, 12, 30))   # => 12:30:00
    TimeOfDay(Date.new(2014, 1, 1))           # => 00:00:00


Shifts
=======================

Represents a period of time, using a beginning and ending TimeOfDay. Allows to calculate its duration and
to determine if a TimeOfDay is included inside the shift. For nightly shifts (when beginning time is greater than ending time),
it supposes the shift ends the following day.

Creating from TimeOfDay
--------------------------------------

    Shift.new(TimeOfDay.new(9), TimeOfDay.new(17))
    Shift.new(TimeOfDay.new(22), TimeOfDay.new(4))

Duration
--------------------

    Shift.new(TimeOfDay.new(9), TimeOfDay.new(17)).duration # => 28800
    Shift.new(TimeOfDay.new(20), TimeOfDay.new(2)).duration # => 21600

Include?
--------------------

    Shift.new(TimeOfDay.new(9), TimeOfDay.new(17)).include?(TimeOfDay.new(12)) # => true
    Shift.new(TimeOfDay.new(9), TimeOfDay.new(17)).include?(TimeOfDay.new(7))  # => false
    Shift.new(TimeOfDay.new(20), TimeOfDay.new(4)).include?(TimeOfDay.new(2))  # => true
    Shift.new(TimeOfDay.new(20), TimeOfDay.new(4)).include?(TimeOfDay.new(18)) # => false


Overlap?
--------------------
    breakfast = Shift.new(TimeOfDay.new(8), TimeOfDay.new(11))
    lunch = Shift.new(TimeOfDay.new(10), TimeOfDay.new(14))
    breakfast.overlaps?(lunch) # => true
    lunch.overlaps?(breakfast) # => true

    dinner = Shift.new(TimeOfDay.new(18), TimeOfDay.new(20))
    dinner.overlaps?(lunch) # => false


Contains?
--------------------
    workday = Shift.new(TimeOfDay.new(9), TimeOfDay.new(17))
    lunch = Shift.new(TimeOfDay.new(10), TimeOfDay.new(14))
    workday.contains?(lunch) # => true
    lunch.contains?(workday) # => false

    dinner = Shift.new(TimeOfDay.new(18), TimeOfDay.new(20))
    dinner.overlaps?(lunch) # => false


Rails Time Zone Support
=======================

If Rails time zone support is loaded, Date#on and TimeOfDay#at will automatically use Time.zone.

Active Record Serializable Attribute Support
=======================
TimeOfDay implements a custom serialization contract for activerecord serialize which allows to store TimeOfDay directly
in a column of the time type.
Example:
```ruby
class Order < ActiveRecord::Base
  serialize :time, Tod::TimeOfDay
end
order = Order.create(time: TimeOfDay.new(9,30))
order.time                                      # => 09:30:00
```

Compatibility
=============

[![Build Status](https://travis-ci.org/jackc/tod.png)](https://travis-ci.org/jackc/tod)

Tod is compatible with Ruby 1.9.3, 2.0.0, 2.1.8, and 2.2.0. It is tested against Rails 3.2, 4.0, 4.1, 4.2.


License
=======

Copyright (c) 2010-2013 Jack Christensen, released under the MIT license
