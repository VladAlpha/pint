.. _tutorial:


Tutorial
========
Pint Basic Concepts
---------------------

Pint uses several simple concepts to help you keep track of real values in a clean safe way.  A quick introduction to the terms used in this guide follows here, and hopefully will make the rest of this manual a clear read.

Unit Registry:  this is the master ?object? that defines the units you will be using. It is generated from a text file and can be modified on the fly, which will define the relationships between units and dimensions etc.

Quantity: A quantity in Pint is the name of the type of object which makes everything work in Pint.  It will typically contain:

 * value: how big it is IE 10
 * units: what the size of the value is being measured in IE Inches
 * dimensionality: what kind of thing the quantity defines, IE is it a length or an area, this is derived directly from the units

Dimension: This is the "type" of measure something is, be it time, length, or volume etc.  These are defined in the unit registry inside of square brackets[]. Dimensions can be based on combinations of other dimensions, but no two dimensions are the exact same thing.  Generally, but not always, in this manual dimensions they will be referred to in square brackets IE [time].

Unit: This is a specific way of measuring a dimension.  Examples of units are inch, meter, and second.  Multiple units can be equivalent to each other, for example "sec" is the same as "second".

Base Units: The unit registry being used defines for each dimension a base unit.  This is the first unit used in the definition file.  Looking at defaults_en.txt, which is the default unit registry file, we can see an example line::

	meter = [length] = m = metre

Here we can see several things.  First we are defining a unit called "meter".  It has a dimensionality of [length].  It is also equal to "m" as well as "metre".  Since "meter" is the first unit defined, it is the "base unit" for all Quantities with a dimensionality of [length]. Continuing down to line 140 as of version 0.7, we can see more [length] units being defined, all based on the meter::

	# Length
	angstrom = 1e-10 * meter = Ångström = Å
	inch = 2.54 * centimeter = in = international_inch = inches = international_inches

*NOTE* Base Units currently are mgs, which is close but not quite SI standard units, so be catious that you request the units you want when using pint.  The default behavior currently has no SI prefix on any base unit, so grams is used instead of kilograms.

Base Dimensions: A Base Dimension is one which is a fundamental form of measure, and can not be reduced further. A clearer but less useful definition is a Base Dimension is a dimension that is not a Derived Dimension.

Derived Dimensions: A Derived Dimension is a dimension that is defined as a combination of other dimensions.  For example [area] is a derived by multiplying [length] by [length], and [speed] is defined as [length]/[time]. These Derived Dimensions also have Base Units just like Base Dimensions do.

Dimensionality:  A Quantity object has a property known as "dimensionality" which expresses exactly what kind of measure it is.  This is a key concept in Pint as it enforces only using legal operations between quantities such as subraction. inches and meters have the same dimensionality, where meters and square meters have different dimensionality.  


Converting Quantities
---------------------

Pint has the concept of Unit Registry, an object within which units are defined and handled. You start by creating your registry::

   >>> from pint import UnitRegistry
   >>> ureg = UnitRegistry()

.. testsetup:: *

   from pint import UnitRegistry
   ureg = UnitRegistry()
   Q_ = ureg.Quantity

If no parameter is given to the constructor, the unit registry is populated with the default list of units and prefixes.
You can now simply use the registry in the following way:

.. doctest::

   >>> distance = 24.0 * ureg.meter
   >>> print(distance)
   24.0 meter
   >>> time = 8.0 * ureg.second
   >>> print(time)
   8.0 second
   >>> print(repr(time))
   <Quantity(8.0, 'second')>

In this code `distance` and `time` are physical quantity objects (`Quantity`). Physical quantities can be queried for their magnitude, units, and dimensionality:

.. doctest::

   >>> print(distance.magnitude)
   24.0
   >>> print(distance.units)
   meter
   >>> print(distance.dimensionality)
   [length]

and can handle mathematical operations between:

.. doctest::

   >>> speed = distance / time
   >>> print(speed)
   3.0 meter / second

As unit registry knows about the relationship between different units, you can convert quantities to the unit of choice:

.. doctest::

   >>> speed.to(ureg.inch / ureg.minute )
   <Quantity(7086.614173228345, 'inch / minute')>

This method returns a new object leaving the original intact as can be seen by:

.. doctest::

   >>> print(speed)
   3.0 meter / second

If you want to convert in-place (i.e. without creating another object), you can use the `ito` method:

.. doctest::

   >>> speed.ito(ureg.inch / ureg.minute )
   <Quantity(7086.614173228345, 'inch / minute')>
   >>> print(speed)
   7086.614173228345 inch / minute

If you ask Pint to perform an invalid conversion:

.. doctest::

   >>> speed.to(ureg.joule)
   Traceback (most recent call last):
   ...
   pint.pint.DimensionalityError: Cannot convert from 'inch / minute' (length / time) to 'joule' (length ** 2 * mass / time ** 2)


There are also methods 'to_base_units' and 'ito_base_units' which automatically convert to the reference units with the correct dimensionality:

.. doctest::

   >>> height = 5.0 * ureg.foot + 9.0 * ureg.inch
   >>> print(height)
   5.75 foot
   >>> print(height.to_base_units())
   1.7526 meter
   >>> print(height)
   5.75 foot
   >>> height.ito_base_units()
   >>> print(height)
   1.7526 meter


In some cases it is useful to define physical quantities objects using the class constructor:

.. doctest::

   >>> Q_ = ureg.Quantity
   >>> Q_(1.78, ureg.meter) == 1.78 * ureg.meter
   True

(I tend to abbreviate Quantity as `Q_`) The built-in parser recognizes prefixed and pluralized units even though they are not in the definition list:

.. doctest::

   >>> distance = 42 * ureg.kilometers
   >>> print(distance)
   42 kilometer
   >>> print(distance.to(ureg.meter))
   42000.0 meter

If you try to use a unit which is not in the registry:

.. doctest::

   >>> speed = 23 * ureg.snail_speed
   Traceback (most recent call last):
   ...
   pint.pint.UndefinedUnitError: 'snail_speed' is not defined in the unit registry

You can add your own units to the registry or build your own list. More info on that :ref:`defining`


String parsing
--------------

Pint can also handle units provided as strings:

.. doctest::

   >>> 2.54 * ureg.parse_expression('centimeter')
   <Quantity(2.54, 'centimeter')>

or using the registry as a callable for a short form:

.. doctest::

   >>> 2.54 * ureg('centimeter')
   <Quantity(2.54, 'centimeter')>

or using the `Quantity` constructor:

.. doctest::

   >>> Q_(2.54, 'centimeter')
   <Quantity(2.54, 'centimeter')>

Numbers are also parsed, so you can use an expression:

.. doctest::

   >>> ureg('2.54 * centimeter')
   <Quantity(2.54, 'centimeter')>

or:

.. doctest::

   >>> Q_('2.54 * centimeter')
   <Quantity(2.54, 'centimeter')>

This enables you to build a simple unit converter in 3 lines:

.. doctest::

   >>> user_input = '2.54 * centimeter to inch'
   >>> src, dst = user_input.split(' to ')
   >>> Q_(src).to(dst)
   <Quantity(1.0, 'inch')>


.. warning:: Pint currently uses eval_ under the hood.
   Do not use this approach from untrusted sources as it is dangerous_.


String formatting
-----------------

Pint's physical quantities can be easily printed:

.. doctest::

   >>> accel = 1.3 * ureg['meter/second**2']
   >>> # The standard string formatting code
   >>> print('The str is {!s}'.format(accel))
   The str is 1.3 meter / second ** 2
   >>> # The standard representation formatting code
   >>> print('The repr is {!r}'.format(accel))
   The repr is <Quantity(1.3, 'meter / second ** 2')>
   >>> # Accessing useful attributes
   >>> print('The magnitude is {0.magnitude} with units {0.units}'.format(accel))
   The magnitude is 1.3 with units meter / second ** 2


.. note::
   In Python 2.6, unnumbered placeholders are invalid. Therefore you need to write `{0}` instead
   of `{}`, `{0!s}` instead of `{!s}` in string formatting operations.


But Pint also extends the standard formatting capabilities for unicode and latex representations:

.. doctest::

   >>> accel = 1.3 * ureg['meter/second**2']
   >>> # Pretty print
   >>> print('The pretty representation is {:P}'.format(accel))
   'The pretty representation is 1.3 meter/second²'
   >>> # Latex print
   >>> 'The latex representation is {:L}'.format(accel)
   'The latex representation is 1.3 \\frac{meter}{second^{2}}'
   >>> # HTML print
   >>> 'The HTML representation is {:H}'.format(accel)
   'The HTML representation is 1.3 meter/second<sup>2</sup>'

.. note:: 
   In Python 2, run ``from __future__ import unicode_literals``
   or prefix pretty  formatted strings with `u` to prevent ``UnicodeEncodeError``.

If you want to use abbreviated unit names, prefix the specification with `~`:

.. doctest::

   >>> 'The str is {:~}'.format(accel)
   'The str is 1.3 m / s ** 2'
   >>> print('The pretty representation is {:~P}'.format(accel))
   The pretty representation is 1.3 m²/s


The same is true for latex (`L`) and HTML (`H`) specs.

Finally, you can specify a default format specification:

   >>> 'The acceleration is {}'.format(accel)
   'The acceleration is 1.3 meter / second ** 2'
   >>> ureg.default_format = 'P'
   >>> 'The acceleration is {}'.format(accel)
   'The acceleration is 1.3 meter/second²'


Using Pint in your projects
---------------------------

If you use Pint in multiple modules within your Python package, you normally want to avoid creating multiple instances of the unit registry.
The best way to do this is by instantiating the registry in a single place. For example, you can add the following code to your package `__init__.py`::

   from pint import UnitRegistry
   ureg = UnitRegistry()
   Q_ = ureg.Quantity


Then in `yourmodule.py` the code would be::

   from . import ureg, Q_

   length = 10 * ureg.meter
   my_speed = Quantity(20, 'm/s')


.. warning:: There are no global units in Pint. All units belong to a registry and you can have multiple registries instantiated at the same time. However, you are not supposed to operate between quantities that belong to different registries. Never do things like this::

    >>> q1 = UnitRegistry().meter
    >>> q2 = UnitRegistry().meter
    >>> # q1 and q2 belong to different registries!
    >>> id(q1._REGISTRY) is id(q2._REGISTRY) # False

.. _eval: http://docs.python.org/3/library/functions.html#eval
.. _dangerous: http://nedbatchelder.com/blog/201206/eval_really_is_dangerous.html
