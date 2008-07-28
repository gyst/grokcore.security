
**********
Components
**********

.. Here we documented the component base classes. For the directive possible
    for each component we document only the specific within its context. We then
    refer to the directives documented in the directives.rst file.

The :mod:`grok` module defines a set of base classes for creating new 
components of different types, that provide basic Zope 3 functionality in a
convenient way. Grok applications are built by subclassing these components.

Components in Grok and Zope 3 can be any plain Python object, that you have
declared implements one or more Interface(s). The inclusion of these Grok base
classes in your own Python classes inheritance automatically handles the
component registration with the Zope Component Architecture. This process of
introspecting your Grok code during initialization and wiring together
components based on common conventions that you follow in the structure
of your source code is called "grokking".


Core components
~~~~~~~~~~~~~~~

:class:`grok.Application`
=========================

Base class for applications. Inherits from :class:`grok.Site`.

:class:`grok.Model`
===================

Base class to define an application "content" or model object. Model objects
provide persistence and containment.

:class:`grok.Container`
=======================

Mixin base class to define a container object. Objects in a container are 
manipulated using the same syntax as you would with the standard
Python Dictionary object. The container implements the
zope.app.container.interfaces.IContainer interface using a BTree, providing
reasonable performance for large collections of objects.

**Example 1: Perform Create, Read, Update and Delete (CRUD) on a container**

.. code-block:: python

    # define a container and a model and then create them
    class BoneBag(grok.Container): pass
    class Bone(grok.Model): pass    
    bag = BoneBag()
    skull = Bone()
    
    # ... your classes are then "grokked" by Grok ...
    
    # store an object in a container
    bag['bone1'] = skull
    
    # test for containment
    bag.has_key('bone1')
    
    # retrieve an object from a container
    first_bone = bag['bone1'] 
    
    # iterate through objects in a container with .values()
    # you can also use .keys() and .items()
    for bone in bag.values():
        bone.marks = 'teeth'
    
    # delete objects using the del keyword
    del bag['bone1']

:class:`grok.Indexes`
=====================

Base class for catalog index definitions.

Adapters
~~~~~~~~

:class:`grok.Adapter`
=====================

Implementation, configuration, and registration of Zope 3 Adapters.

Adapters are components that are constructed from other components. They
take an existing interface and extend it to provide a new interface.

.. class:: grok.Adapter

    Base class to define an adapter. Adapters are automatically
    registered when a module is "grokked".

    .. attribute:: grok.Adapter.context

        The adapted object.

    **Directives:**

    :func:`grok.context(context_obj_or_interface)`
        Maybe required. Identifies the type of objects or interface for
        the adaptation.

    .. seealso::

        :func:`grok.context`

    :func:`grok.implements(\*interfaces)`
        Required. Identifies the interface(s) the adapter implements.

    .. seealso::

        :func:`grok.implements`

    :func:`grok.name(name)`
        Optional. Identifies the name used for the adapter
        registration. If ommitted, no name will be used.

        When a name is used for the adapter registration, the adapter
        can only be retrieved by explicitely using its name.

    .. seealso::

        :func:`grok.name`

    :func:`grok.provides(name)`
        Maybe required.

    .. seealso::

        :func:`grok.provides`

**Example 1: Simple adaptation example**

.. code-block:: python

    import grok
    from zope import interface

    class Cave(grok.Model):
        "start with a cave objects (the adaptee)"

    class IHome(interface.Interface):
        "we want to extend caves with the IHome interface"

    class Home(grok.Adapter):
        "the home adapter turns caves into habitable homes!"
        grok.implements(IHome)

    # Adapation (component look-up) is invoked by passing the adaptee
    # to the interface as a constructor and returns the component adapted to   
    home = IHome(cave)


**Example 2: Register and retrieve the adapter under a specific name**

.. code-block:: python

    import grok
    from zope import interface

    class Cave(grok.Model):
        pass
    class IHome(interface.Interface):
        pass

    class Home(grok.Adapter):
        grok.implements(IHome)
        grok.name('home')

    from zope.component import getAdapter
    home = getAdapter(cave, IHome, name='home')


:class:`grok.MultiAdapter`
==========================

.. class:: grok.MultiAdapter

    Base class to define a Multi Adapter.
   
    A simple adapter normally adapts only one object, but an adapter may
    adapt more than one object. If an adapter adapts more than one objects,
    it is called multi-adapter.

    **Directives:**

    :func:`grok.adapts(\*objects_or_interfaces)`
        Required. Identifies the combination of types of objects or interfaces
        for the adaptation.

    :func:`grok.implements(\*interfaces)`
        Required. Identifies the interfaces(s) the adapter implements.

    :func:`grok.name(name)`
        Optional. Identifies the name used for the adapter registration. If
        ommitted, no name will be used.

        When a name is used for the adapter registration, the adapter can only be
        retrieved by explicitely using its name.

    :func:`grok.provides(name)`
        Only required if the adapter implements more than one interface.
        :func:`grok.provides` is required to disambiguate for which interface the
        adapter will be registered for.

**Example: A home is made from a cave and a fireplace.**

.. code-block:: python

    import grok
    from zope import interface

    class Fireplace(grok.Model):
       pass

    class Cave(grok.Model):
       pass

    class IHome(interface.Interface):
       pass

    class Home(grok.MultiAdapter):
       grok.adapts(Cave, Fireplace)
       grok.implements(IHome)

       def __init__(self, cave, fireplace):
           self.cave = cave
           self.fireplace = fireplace

    home = IHome(cave, fireplace)

:class:`grok.Annotation`
========================

Annotation components are persistent writeable adapters.

.. class:: grok.Annotation

    Base class to declare an Annotation. Inherits from the
    persistent.Persistent class.

**Example: Storing annotations on model objects**

.. code-block:: python

    import grok
    from zope import interface

    # Create a model and an interface you want to adapt it to
    # and an annotation class to implement the persistent adapter.
    class Mammoth(grok.Model):
        pass

    class ISerialBrand(interface.Interface):
        unique = interface.Attribute("Brands")

    class Branding(grok.Annotation):
        grok.implements(IBranding)
        unqiue = 0
   
    # Grok the above code, then create some mammoths
    manfred = Mammoth()
    mumbles = Mammoth()
   
    # creating Annotations work just like Adapters
    livestock1 = ISerialBrand(manfred)
    livestock2 = ISerialBrand(mumbles)
   
    # except you can store data in them, this data will transparently persist
    # in the database for as long as the object exists
    livestock1.serial = 101
    livestock2.serial = 102

Utilities
~~~~~~~~~

:class:`grok.GlobalUtility`
===========================

.. class:: grok.GlobalUtility

    Base class to define a globally registered utility. Global utilities are
    automatically registered when a module is "grokked".

    **Directives:**

    :func:`grok.implements(\*interfaces)`
        Required. Identifies the interfaces(s) the utility implements.

    :func:`grok.name(name)`
        Optional. Identifies the name used for the adapter registration. If
        ommitted, no name will be used.

        When a name is used for the global utility registration, the global
        utility can only be retrieved by explicitely using its name.

    :func:`grok.provides(name)`
        Maybe required. If the global utility implements more than one interface,
        :func:`grok.provides` is required to disambiguate for what interface the
        global utility will be registered.


:class:`grok.LocalUtility`
==========================

.. class:: grok.LocalUtility

    Base class to define a utility that will be registered local to a
    :class:`grok.Site` or :class:`grok.Application` object by using the
    :func:`grok.local_utility` directive.

    **Directives:**

    :func:`grok.implements(\*interfaces)`
        Optional. Identifies the interfaces(s) the utility implements.

    :func:`grok.name(name)`
        Optional. Identifies the name used for the adapter registration. If
        ommitted, no name will be used.

        When a name is used for the local utility registration, the local utility
        can only be retrieved by explicitely using its name.

    :func:`grok.provides(name)`
        Maybe required. If the local utility implements more than one interface
        or if the implemented interface cannot be determined,
        :func:`grok.provides` is required to disambiguate for what interface the
        local utility will be registered.

  	.. seealso::

	    Local utilities need to be registered in the context of :class:`grok.Site`
	    or :class:`grok.Application` using the :func:`grok.local_utility` directive.

:class:`grok.Site`
==================

Contains a Site Manager. Site Managers act as containers for registerable
components.

If a Site Manager is asked for an adapter or utility, it checks for those
it contains before using a context-based lookup to find another site
manager to delegate to. If no other site manager is found they defer to
the global site manager which contains file based utilities and adapters.

.. class:: grok.Site

	.. method:: getSiteManager():

		Returns the site manager contained in this object.

		If there isn't a site manager, raise a component lookup.

	.. method:: setSiteManager(sitemanager):
	
		Sets the site manager for this object.

Views
~~~~~

:class:`grok.View`
==================

View components provide context and request attributes. 

The determination of what View gets used for what Model is made by walking the
URL in the HTTP Request object sepearted by the / character. This process is
called Traversal.

.. class:: grok.View

    Base class to define a View.

    .. attribute:: context

        The object that the view is presenting. This is often an instance of
        a grok.Model class, but can also be a grok.Application or grok.Container
        object.

    .. attribute:: request
   
        The HTTP Request object.

    .. attribute:: response

        The HTTP Response object that is associated with the request. This
        is also available as self.request.response, but the response attribute
        is provided as a convenience.

    .. attribute:: static

        Directory resource containing the static files of the view's package.

    .. method:: redirect(url):
   
        Redirect to given URL

    .. method:: url(obj=None, name=None, data=None):
   
        Construct URL.

        If no arguments given, construct URL to view itself.

        If only obj argument is given, construct URL to obj.

        If only name is given as the first argument, construct URL
        to context/name.

        If both object and name arguments are supplied, construct
        URL to obj/name.

        Optionally pass a 'data' keyword argument which gets added to the URL
        as a cgi query string.

    .. method:: default_namespace():

        Returns a dictionary of namespaces that the template
        implementation expects to always be available.

        This method is *not* intended to be overridden by application
        developers.

    .. method:: namespace():
   
        Returns a dictionary that is injected in the template
        namespace in addition to the default namespace.

        This method *is* intended to be overridden by the application
        developer.

    .. method:: update(**kw):
   
        This method is meant to be implemented by grok.View
        subclasses.  It will be called *before* the view's associated
        template is rendered and can be used to pre-compute values
        for the template.

        update() can take arbitrary keyword parameters which will be
        filled in from the request (in that case they *must* be
        present in the request).

    .. method:: render(**kw):
   
        A view can either be rendered by an associated template, or
        it can implement this method to render itself from Python.
        This is useful if the view's output isn't XML/HTML but
        something computed in Python (plain text, PDF, etc.)

        render() can take arbitrary keyword parameters which will be
        filled in from the request (in that case they *must* be
        present in the request).

    .. method:: application_url(name=None):
   
        Return the URL of the closest application object in the
        hierarchy or the URL of a named object (``name`` parameter)
        relative to the closest application object.

    .. method:: flash(message, type='message'):
      
        Send a short message to the user.

:class:`grok.JSON`
==================

Specialized View that returns data in JSON format.

Python data returned is automatically converted into JSON format using
the simplejson library. Every method name in a grok.JSON component is
registered as the name of a JSON View. The exceptions are names that
begin with an _ or special names such as __call__. The grok.require
decorator can be used to protect methods with a permission.

.. class:: grok.JSON

    Base class for JSON methods.

**Example 1: Create a public and a protected JSON view.**

.. code-block:: python

    class MammothJSON(grok.JSON):
        """
        Returns JSON from URLs in the form of:
      
        http://localhost/stomp
        http://localhost/dance
        """

        grok.context(zope.interface.Interface)

        def stomp(self):
            return {'Manfred stomped.': ''}

        @grok.require('zope.ManageContent')
        def dance(self):
            return {'Manfred does not like to dance.': ''}


:class:`grok.XMLRPC`
====================

Specialized View that responds to XML-RPC.

The grok.require decorator can be used to protect methods with a permission.

.. class:: grok.JSON

    Base class for XML-RPC methods.

**Example 1: Create a public and a protected XML-RPC view.**

.. code-block:: python

    from zope import interface
   
    class FooXMLRPC(grok.XMLRPC):
        """
        The methods in this class will be available as XML-RPC methods.
      
        http://localhost/say will return 'Hello world!' encoded in XML-RPC.
        """
        grok.context(interface.Interface)

        def say(self):
            return 'Hello world!'


:class:`grok.Traverser`
=======================

A Traverser is used to map from a URL to an object being published (Model)
and the View used to interact with that object.

.. class:: grok.Traverser

    Base class for custom traversers. Override the traverse method to 
    supply the desired custom traversal behaviour.

    .. attribute:: context

        The object that is being traversed.

    .. attribute:: request
   
        The HTTP Request object.

    .. method:: traverse(self, name):
      
        You must provide your own implementation of this method to do what
        you want. If you return None, Grok will use the default traversal
        behaviour.

    .. method:: browserDefault(request):
   
        Returns an object and a sequence of names.
	  
        The publisher calls this method at the end of each traversal path.
        If the sequence of names is not empty, then a traversal step is made
        for each name. After the publisher gets to the end of the sequence,
        it will call browserDefault on the last traversed object.
	  
        The default behaviour in Grok is to return self.context for the object
        and 'index' for the default view name.
	  
        Note that if additional traversal steps are indicated (via a
        nonempty sequence of names), then the publisher will try to adjust
        the base href.

    .. method:: publishTraverse(request, name):

        Lookup a name and return an object with `self.context` as it's parent.
        The method can use the request to determine the correct object.
	  
        The 'request' argument is the publisher request object. The
        'name' argument is the name that is to be looked up. It must
        be an ASCII string or Unicode object.
	  
        If a lookup is not possible, raise a NotFound error.

**Example 1: Traverse into a Herd Model and return a Mammoth Model**

.. code-block:: python

    import grok

    class Herd(grok.Model):

       def __init__(self, name):
           self.name = name

    class HerdTraverser(grok.Traverser):
       grok.context(Herd)

       def traverse(self, name):
           return Mammoth(name)

    class Mammoth(grok.Model):

       def __init__(self, name):
           self.name = name


:class:`grok.PageTemplate`
==========================

:class:`grok.PageTemplateFile`
==============================

Forms
~~~~~

Forms inherit from the `grok.View` class. They are a specialized type of
View that renders an HTML Form.

:class:`grok.Form`
==================

.. class:: grok.Form

    Base class for forms.

    .. attribute:: prefix
    
        Page-element prefix. All named or identified page elements in a
        subpage should have names and identifiers that begin with a subpage
        prefix followed by a dot.

    .. method:: setPrefix(prefix):

        Update the subpage prefix

    .. attribute:: label
    
        A label to display at the top of a form.
        
    .. attribute:: status
    
        An update status message. This is normally generated by success or
        failure handlers.
    
    .. attribute:: errors

        Sequence of errors encountered during validation.

    .. attribute:: form_result
    
        Return from action result method.

    .. attribute:: form_reset
    
        Boolean indicating whether the form needs to be reset.

    .. attribute:: form_fields
    
        The form's form field definitions.

        This attribute is used by many of the default methods.

    .. attribute:: widgets
    
        The form's widgets.

        - set by setUpWidgets

        - used by validate


    .. method:: setUpWidgets(ignore_request=False):
    
        Set up the form's widgets.

        The default implementation uses the form definitions in the
        form_fields attribute and setUpInputWidgets.

        The function should set the widgets attribute.

    .. method:: validate(action, data):
    
        The default form validator

        If an action is submitted and the action doesn't have it's own
        validator then this function will be called.

    .. attribute:: template
    
        Template used to display the form

    .. method:: resetForm():
    
        Reset any cached data because underlying content may have changed.

    .. method:: error_views():
    
        Return views of any errors.

        The errors are returned as an iterable.

    .. method:: applyData(obj, **data):
    
        Save form data to an object.

        This returns a dictionary with interfaces as keys and lists of
        field names as values to indicate which fields in which
        schemas had to be changed in order to save the data.  In case
        the method works in update mode (e.g. on EditForms) and
        doesn't have to update an object, the dictionary is empty.

:class:`grok.AddForm`
=====================

Add forms are used for creating new objects. The widgets for this form
are not bound to any existing content or model object.

.. class:: grok.AddForm

    Base class for add forms.

:class:`grok.EditForm`
======================

Edit forms are used for editing existing objects. The widgets for this form
are bound to the object set in the `context` attribute.

.. class:: grok.EditForm

    Base class for edit forms.

:class:`grok.DisplayForm`
=========================

Display forms are used to display an existing object. The widgets for this
form are bound to the object set in the `context` attribute.

.. class:: grok.DisplayForm

    Base class for display forms.

Security
~~~~~~~~

:class:`Permission`
===================

Permissions are used to protect Views so that they can only be called by
an authenticated principal. If a View in Grok does not have a `grok.require`
directive declaring a permission needed to use the View, then the view will
be public.

.. class:: grok.Permission

    Base class for permissions. You must specify a unique name for every
    permission using the `grok.name` directive. The convention for ensuring
    uniqueness is to prefix your permission name with the name of your
    Grok package followed by a dot, e.g. 'mypackage.MyPermissionName'.

    .. attribute:: id
    
        Id as which this permission will be known and used. This is set
        to the value specified in the `grok.name` directive.

    .. attribute:: title

        Human readable identifier for this permission.

    .. attribute:: description

        Description of the permission.

    **Directives:**

    :func:`grok.name(name)`
    
        Required. Identifies the unique name (also used as the id) of the
        permission.

    :func:`grok.title(title)`

        Optional. Stored as the title attribute for this permission.
    
    :func:`grok.description(description)`

        Optional. Stored as the description attribute for this permission.

**Example 1: Define a new Permission and use it to protect a View**

.. code-block:: python

    import grok
    import zope.interface
    
    class Read(grok.Permission):
        grok.name('mypackage.Read')

    class Index(grok.View):
        grok.context(zope.interface.Interface)
        grok.require('mypackage.Read')

:func:`grok.define_permission` -- define a permission
=====================================================

.. function:: grok.define_permission(name)

    A module-level directive to define a permission with name
    `name`. Usually permission names are prefixed by a component- or
    application name and a dot to keep them unique.

    Because in Grok by default everything is accessible by everybody,
    it is important to define permissions, which restrict access to
    certain principals or roles.

    **Example:**

    .. code-block:: python

        import grok
        grok.define_permission('cave.enter')


    .. seealso::

        :func:`grok.require`, :class:`grok.Permission`, :class:`grok.Role`

    .. versionchanged:: 0.11

        replaced by :class:`grok.Permission`.

:class:`Role`
=============

Roles provide a way to group together a collection of permissions. Principals
(aka Users) can be granted a Role which will allow them to access all Views
protected by the Permissions that the Role contains.

.. class:: grok.Role

    Base class for roles.

    .. attribute:: id
    
        Id as which this role will be known and used. This is set
        to the value specified in the `grok.name` directive.

    .. attribute:: title

        Human readable identifier for this permission.

    .. attribute:: description

        Description of the permission.

    **Directives:**

    :func:`grok.name(name)`

        Required. Identifies the unique name (also used as the id) of the
        role.

    :func:`grok.permissions(permissions)`

        Required. Declare the permissions granted to this role.

    :func:`grok.title(title)`

        Optional. Stored as the title attribute for this role.

    :func:`grok.description(description)`

        Optional. Stored as the description attribute for this role.

**Example 1: Define a new 'paint.Artist' Role and assign it to the 'paint.grok' principal**

.. code-block:: python

    import grok
    import zope.interface

    class ViewPermission(grok.Permission):
        grok.name('paint.ViewPainting')

    class EditPermission(grok.Permission):
        grok.name('paint.EditPainting')

    class ErasePermission(grok.Permission):
        grok.name('paint.ErasePainting')

    class ApprovePermission(grok.Permission):
        grok.name('paint.ApprovePainting')

    class Artist(grok.Role):
        """
        An Artist can view, create and edit paintings. However, they can
        not approve their painting for display in the Art Gallery Cave.
        """
        grok.name('paint.Artist')
        grok.title('Artist')
        grok.description('An artist owns the paintings that they create.')
        grok.permissions(
            'paint.ViewPainting', 'paint.EditPainting', 'paint.ErasePainting')

    class CavePainting(grok.View):
        grok.context(zope.interface.Interface)
        grok.require(ViewPermission)

        def render(self):
            return 'What a beautiful painting.'

    class EditCavePainting(grok.View):
        grok.context(zope.interface.Interface)
        grok.require(EditPermission)

        def render(self):
            return 'Let\'s make it even prettier.'

    class EraseCavePainting(grok.View):
        grok.context(zope.interface.Interface)
        grok.require(ErasePermission)

        def render(self):
            return 'Oops, mistake, let\'s erase it.'

    class ApproveCavePainting(grok.View):
        grok.context(zope.interface.Interface)
        grok.require(ApprovePermission)

        def render(self):
            return 'Painting owners cannot approve their paintings.'

    # The app variable will typically be your Application instance,
    # but could also be a container within your application.
    from zope.securitypolicy.interfaces import IPrincipalRoleManager
    IPrincipalRoleManager(app).assignRoleToPrincipal(
       'paint.Artixt', 'paint.grok')
