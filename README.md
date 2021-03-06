# LazyDoc

NOTE: LazyDoc is currently in alpha and is not quite ready for use.

[![Build Status](https://travis-ci.org/ryanoglesby08/lazy-doc.png)](https://travis-ci.org/ryanoglesby08/lazy-doc)



An implementation of the [Embedded Document](http://martinfowler.com/bliki/EmbeddedDocument.html) pattern for POROs.

LazyDoc provides a declarative DSL for extracting deeply nested values from a JSON document. The embedded JSON is lazily
parsed so that needed attributes from the document are only parsed when accessed. Finally, parsed values are cached
so that subsequent access does not access the JSON again.

*Currently, LazyDoc only supports JSON. XML support will be added later.*

## Installation

Add this line to your application's Gemfile:

    gem 'lazy_doc'

## DSL Options

1. Basic usage. `access :name` will look for a property called 'name' at the top level of the embedded document.
2. Basic usage with multiple attributes. `access :name, :phone, :address` will look for all three properties. *This option does not currently support using any options.*
2. `via`: `access :job, via: [:profile, :occupation]` will look for a property called 'job' by parsing through
'profile' -> 'occupation'.
3. `finally`: `access :initials, finally: lambda { |initials| initials.upcase }` will call the supplied block, passing in
'initials,' and will return the result of that block.
4. `as`: `access :profile, as: Profile` will pass the sub-document found at 'profile' into a new 'Profile' object, and will return
the newly constructed Profile object. This is great for constructing nested LazyDoc relationships.

## Example Usage

```ruby
class User
    include LazyDoc::DSL

    access :name                                        # Access the attribute "name"
    access :address, via: :street_address               # Access the attribute "street_address"
    access :job, via: [:profile, :occupation, :title]   # Access the attribute "title" found at "profile" -> "occupation"

    def initialize(json)
      lazily_embed(json)                                # Initialize the LazyDoc object
    end
end

user = User.new(json)
puts user.name
puts user.address
puts user.job
```

## To Do

1. DONE - Full path parsing more than just top level.  ex: `access :name, by: [:profile, :basic_info, :name]`
2. DONE - Error throwing for incorrectly specified paths
3. Default value if json does not contain specified path or is null or empty. ex: `access :currency, default: 'USD'`
4. DONE - Transforms. ex: `access :name, finally: lambda { |name| name.gsub('-',' ') }`
5. DONE - Objects from sub-trees.  ex: `access :profile, as: Profile` (This would construct a LazyDoc Profile object and pass the json found at "profile" to it)
6. Collections.
    - Map. For example, extract array of customer names from array of customers. ex: `access :customers, extract: :name`
    - Objects from collection. Instead of extracting just the name, extract whole objects like in #5.  ex:  `access :customers, as: Customer`
7. Joins
    - Using previously defined attributes. ex: `join :address, from: [:street, :city, :state:, :zip]`
    - Defining attributes in place.
8. Throw exception immediately when trying to use incompatible options (ex: `'finally' and 'as'`)
9. DONE - Multiple simple paths in one line (ex: `access :name, :street, :city, :state`)
10. XML support

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
