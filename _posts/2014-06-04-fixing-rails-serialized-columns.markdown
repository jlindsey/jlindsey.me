---
title: Fixing Rails Serialized Columns
tags:
  - Rails
  - Ruby
---

Working with [serialized attributes][1] in Rails is a great experience. It's one of those areas where Rails 
Just Works... Mostly. The only issue is that, in order to protect against silent data loss, ActiveRecord 
always saves models with serialized attributes even if there were no changes made. It also, of course, updates 
the `updated_at` column. This can really mess things up in certain situations: causing cache thrashing, 
unnecessary database load, and difficulty figuring out if a model has really been changed using the normal 
ActiveRecord interfaces.

This has been [known to the community][2] for a long time, and as discussed in that Issue thread, is intended 
behavior. Most supplied workarounds (such as wrapping all calls to `#save` with `#changed?` checks) simply 
didn't work. This was causing me all kinds of issues in a recent project, and I finally decided to do 
something about it.  Using Rails' great [`ActiveSupport::Concern`][3], I made a simple include module that 
gets around this issue safely by using Ruby's `Object#hash` to "snapshot" the state of a deserialized object 
every time a model is initialialized or fetched, and after it's saved. It overrides some internal AR methods 
to rehash the current state of a deserialized object and check that against the stored hash, and considers the 
attribute changed or not based on that.

{% gist jlindsey/f42a6c091eaae90d7701 %}

There are, of course, some caveats with this. If you are storing very large or deeply-nested objects as 
serialized data (especially if you have a high number of data reads and writes), this may introduce a 
significant slowdown as AR will have to hash all those objects. If you are overriding `Object#hash` for any 
reason, you should tread very carefully lest you get silent data loss.

There is currently an [open pull request][4] that fixes this issue using a very similar method as this (baked 
into AR instead of being in a Concern, of course) but until/unless that gets merged in and released in a 
future version of Rails, devs will still have to make do ourselves with workarounds like this.

[1]: http://apidock.com/rails/ActiveRecord/AttributeMethods/Serialization/ClassMethods/serialize
[2]: https://github.com/rails/rails/issues/8328
[3]: http://api.rubyonrails.org/classes/ActiveSupport/Concern.html
[4]: https://github.com/rails/rails/pull/15458

