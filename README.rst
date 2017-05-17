Using Functional Programming for efficient Data Processing and Analysis
=======================================================================

What is functional programming?
-------------------------------

Rectangle (imperative)
~~~~~~~~~

.. code-block:: python

    class Rectangle(object):
        def __init__(self, length, width):
            self.length = length
            self.width = width

        @property
        def area(self):
            return self.length * self.width

        def grow(self, amount):
            self.length *= amount

    >>> r = Rectangle(2, 3)
    >>> r.length
    2
    >>> r.area
    6
    >>> r.grow(2)
    >>> r.length
    4
    >>> r.area
    12

Expensive Rectangle (imperative)
~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    from time import sleep

    class ExpensiveRectangle(Rectangle):
        @property
        def area(self):
            sleep(5)
            return self.length * self.width

    >>> r = ExpensiveRectangle(2, 3)
    >>> r.area
    6
    >>> r.area
    6

Infinite Squares (imperative)
~~~~~~~~~~~~~~~~

.. code-block:: python

    def sum_area(rects):
        area = 0

        for r in rects:
            area += r.area

        return area

    >>> from itertools import count
    >>>
    >>> squares = (
    ...     Rectangle(x, x) for x in count(1))
    >>> squares
    <generator object <genexpr> at 0x11233ca40>
    >>> next(squares)
    <__main__.Rectangle at 0x1123a8400>
    >>> sum_area(squares)
    KeyboardInterrupt                         Traceback (most recent call last)
    <ipython-input-196-6a83df34d1b4> in <module>()
    ----> 1 sum_area(squares)

    <ipython-input-193-3d117e0b93c3> in sum_area(rects)
          3
          4     for r in rects:
    ----> 5         area += r.area

Rectangle (functional)
~~~~~~~~~~~~~~~~

.. code-block:: python

    def make_rect(length, width):
        return (length, width)

    def grow_rect(rect, amount):
        return (rect[0] * amount, rect[1])

    def get_length (rect):
       return rect[0]

    def get_area (rect):
       return rect[0] * rect[1]

    >>> r = make_rect(2, 3)
    >>> get_length(r)
    2
    >>> get_area(r)
    6
    >>> grow_rect(r, 2)
    (4, 3)
    >>> get_length(r)
    2
    >>> get_area(r)
    6

    >>> big_r = grow_rect(r, 2)
    >>> get_length(big_r)
    4
    >>> get_area(big_r)
    12

Expensive Rectangle (functional)
~~~~~~~~~~~~~~~~

.. code-block:: python

    from functools import lru_cache

    @lru_cache()
    def exp_get_area (rect):
        sleep(5)
        return rect[0] * rect[1]

    >>> r = make_rect(2, 3)
    >>> exp_get_area(r)
    6
    >>> exp_get_area(r)
    6

Infinite Squares (functional)
~~~~~~~~~~~~~~~~

.. code-block:: python

    def accumulate_area(rects):
        accum = 0

        for r in rects:
            accum += get_area(r)
            yield accum

    >>> from itertools import islice
    >>>
    >>> squares = (
    ...     make_rect(x, x) for x in count(1))
    >>>
    >>> area = accumulate_area(squares)
    >>> next(islice(area, 6, 7))
    140
    >>> next(area)
    204

    >>> from itertools import accumulate
    >>>
    >>> squares = (
    ...     make_rect(x, x) for x in count(1))
    >>>
    >>> area = accumulate(map(get_area, squares))
    >>> next(islice(area, 6, 7))
    140
    >>> next(area)
    204

Exercise #1
-------------------------------

Problem
~~~~~~~~~

.. code-block:: python

    ratio = function1(x, y, factor)
    hyp = function2(rectangle)
    # z = √(x2 + y2 )
