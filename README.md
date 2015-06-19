# modelle
Data model serialiser and deserialiser, think WTForms but to and from more data sources.

Define a model with attributes that have types, validators and constraints. Take a data source (e.g. HTML Form, JSON, GET parameters, ...) and convert it into this model. Manipulate the model however you'd like then convert it back. The `modelle` library would provide sensible yet extensible ways to describe models, convert data, and validate values.

## Motivation

I have a Flask app that deals with lots of inter-related SQLAlchemy data models. I started work on an admin interface to these models using WTForms to generate and validate forms. Not long after I decided that opening up a REST-like API for these models and letting the admin interface use AJAX to CRUD them would make for a better user experience. My admins wouldn't have to reload pages of forms, one form per object, any more.

Opening up my WTForms models to allow for JSON representation didn't seem that easy. There is a [WTForms-JSON library](http://wtforms-json.readthedocs.org/en/latest/) that probably would do what I wanted but I only discovered it after mulling the idea for `modelle` over in my head for a bit. There are lots of things about WTForms that I love and some that I don't, this issue was the straw that broke the camel's back and made me think of an alternative.

## Draft API

    from modelle import Model, Text, List, Phone, Email
    
    class Person(Model):
        name = Text()
        email = Email()
        phone = List(Phone)
        spouse = Person()  # I know this won't work, this is just a draft
        
    p = Person()
    p.from_json('{"name":"Dave","email":"bad email"}')
    if p.email.errors:
        p.email = 'dave@email.com'
    print p.to_form(method='POST')
    """<form method="POST">
        <div class="field name">
            <label for="name">Name</label>
            <input type="text" value="Dave" />
        </div>
        ...
    </form>"""
    print p.to_json()
    """{"name":"Dave","email":"dave@email.com"}"""

Basically we try to clean up the model definition a bit from WTForms, mostly by describing things by their characteristics, not their HTML representation. Then we provide a set of `from_*` and `to_*` methods that do the serialisation using sensible defaults. Everything is overridable by sub-classing things. Validation is inherent to the class.
    
