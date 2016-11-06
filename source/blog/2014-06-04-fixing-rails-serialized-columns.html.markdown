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

~~~ ruby
# See: https://github.com/rails/rails/pull/15458

module ProperSerializationSaving
  extend ActiveSupport::Concern

  included do
    # The check prevents the after_initialize hook to run
    # again if the model is dup()'d.
    after_initialize :store_serialized_attribute_hashes!,
                     :unless => proc { |m| m.serialized_attribute_hashes.present? }
    after_save :store_serialized_attribute_hashes!
  
    attr_accessor :serialized_attribute_hashes
  end

  def keys_for_partial_write
    changed
  end

  def should_record_timestamps?
    self.record_timestamps and (!partial_writes? or changed?)
  end

  def changed
    base_changed = Array(super)
    return (base_changed + changed_serialized_attributes).flatten
  end

  def changed?
    !self.changed.empty?
  end

  def any_serialized_attributes_changed?
    self.serialized_attribute_hashes.any? do |atr, hash|
      self.__send__(atr.to_sym).hash != hash
    end
  end

  def changed_serialized_attributes
    csa = self.serialized_attribute_hashes.map do |atr, hash|
      atr if self.__send__(atr.to_sym).hash != hash 
    end

    csa.compact
  end

  protected
    def store_serialized_attribute_hashes!
      self.serialized_attribute_hashes ||= {}
      self.class.serialized_attributes.keys.each do |atr|
        frozen = self.__send__(atr.to_sym).hash.freeze
        self.serialized_attribute_hashes[atr] = frozen
      end
    end
end
~~~

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

