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


Solution
~~~~~~~~~

.. code-block:: python

    from math import sqrt, pow

    def get_hyp(rect):
        sum_s = sum(pow(r, 2) for r in rect)
        return sqrt(sum_s)

    def get_ratio(length, width, factor=1):
        rect = make_rect(length, width)
        big_rect = grow_rect(rect, factor)
        return get_hyp(rect) / get_hyp(big_rect)


You might not need pandas
-------------------------

csv data
~~~~~~~~

.. code-block:: python

    >>> from csv import DictReader
    >>> from io import StringIO
    >>>
    >>> csv_str = 'Type,Day\ntutorial,wed\ntalk,fri'
    >>> csv_str += '\nposter,sun'
    >>> f = StringIO(csv_str)
    >>> data = DictReader(f)
    >>> dict(next(data))
    {'Day': 'wed', 'Type': 'tutorial'}

json data
~~~~~~~~

.. code-block:: python

    >>> from urllib.request import urlopen
    >>> from ijson import items
    >>>
    >>> json_url = 'https://api.github.com/users'
    >>> f = urlopen(json_url)
    >>> data = items(f, 'item')
    >>> next(data)
    {'avatar_url': 'https://avatars3.githubuserco…',
     'events_url': 'https://api.github.com/users/…',
     'followers_url': 'https://api.github.com/use…',
     'following_url': 'https://api.github.com/use…',

xls(x) data
~~~~~~~~

.. code-block:: python

    >>> from urllib.request import urlretrieve
    >>> from xlrd import open_workbook
    >>>
    >>> xl_url = 'https://github.com/reubano/meza'
    >>> xl_url += '/blob/master/data/test/test.xlsx'
    >>> xl_url += '?raw=true'
    >>> xl_path = urlretrieve(xl_url)[0]
    >>> book = open_workbook(xl_path)
    >>> sheet = book.sheet_by_index(0)
    >>> header = sheet.row_values(0)
    >>> nrows = range(1, sheet.nrows)
    >>> rows = (sheet.row_values(x) for x in nrows)
    >>> data = (
    ...     dict(zip(header, row)) for row in rows)
    >>>
    >>> next(data)
    {' ': ' ',
     'Some Date': 30075.0,
     'Some Value': 234.0,
     'Sparse Data': 'Iñtërnâtiônàližætiøn',
     'Unicode Test': 'Ādam'}

grouping data
~~~~~~~~

.. code-block:: python

    >>> import itertools as it
    >>> from operator import itemgetter >>>
    >>> records = [
    ...     {'item': 'a', 'amount': 200},
    ...     {'item': 'b', 'amount': 200},
    ...     {'item': 'c', 'amount': 400}]
    >>>
    >>> keyfunc = itemgetter('amount')
    >>> _sorted = sorted(records, key=keyfunc)
    >>> groups = it.groupby(_sorted, keyfunc)
    >>> data = ((key, list(g)) for key, g in groups)
    >>> next(data)
    (200, [{'amount': 200, 'item': 'a'},
           {'amount': 200, 'item': 'b'}])

aggregating data
~~~~~~~~

.. code-block:: python

    >>> key = 'amount'
    >>> value = sum(r.get(key, 0) for r in records)
    >>> {**records[0], key: value}
    {'a': 'item', 'amount': 800}

csv files
~~~~~~~~

.. code-block:: python

    >>> from csv import DictWriter
    >>>
    >>> records = [
    ...     {'item': 'a', 'amount': 200},
    ...     {'item': 'b', 'amount': 400}]
    >>>
    >>> header = list(records[0].keys())
    >>> with open('output.csv', 'w') as f:
    ...     w = DictWriter(f, header)
    ...     w.writeheader()
    ...     w.writerows(records)

Introducing meza
-------------------------------

csv data
~~~~~~~~~

.. code-block:: python

    >>> from meza.io import read
    >>>
    >>> records = read('output.csv')
    >>> next(records)
    {'amount': '200', 'item': 'a'}

JSON data
~~~~~~~~~

.. code-block:: python

    >>> from meza.io import read_json
    >>>
    >>> f = urlopen(json_url)
    >>> records = read_json(f, path='item')
    >>> next(records)
    {'avatar_url': 'https://avatars3.githubuserco…',
     'events_url': 'https://api.github.com/users/…',
     'followers_url': 'https://api.github.com/use…',
     'following_url': 'https://api.github.com/use…',
     …
    }

xlsx data
~~~~~~~~~

.. code-block:: python

    >>> from meza.io import read_xls
    >>>
    >>> records = read_xls(xl_path)
    >>> next(records)
    {'Some Date': '1982-05-04',
     'Some Value': '234.0',
     'Sparse Data': 'Iñtërnâtiônàližætiøn',
     'Unicode Test': 'Ādam'}

aggregation
~~~~~~~~~

.. code-block:: python

    >>> from meza.process import aggregate
    >>>
    >>> records = [
    ...     {'a': 'item', 'amount': 200},
    ...     {'a': 'item', 'amount': 300},
    ...     {'a': 'item', 'amount': 400}]
    ...
    >>> aggregate(records, 'amount', sum)
    {'a': 'item', 'amount': 900}

merging
~~~~~~~~~

.. code-block:: python

    >>> from meza.process import merge
    >>>
    >>> records = [
    ...     {'a': 200}, {'b': 300}, {'c': 400}]
    >>>
    >>> merge(records)
    {'a': 200, 'b': 300, 'c': 400}

grouping
~~~~~~~~~

.. code-block:: python

    >>> from meza.process import group
    >>>
    >>> records = [
    ...     {'item': 'a', 'amount': 200},
    ...     {'item': 'a', 'amount': 200},
    ...     {'item': 'b', 'amount': 400}]
    >>>
    >>> groups = group(records, 'item')
    >>> next(groups)

normalization
~~~~~~~~~

.. code-block:: python

    >>> from meza.process import normalize
    >>>
    >>> records = [
    ...     {
    ...         'color': 'blue', 'setosa': 5,
    ...         'versi': 6
    ...     }, {
    ...         'color': 'red', 'setosa': 3,
    ...         'versi': 5
    ...     }]
    >>> kwargs = {
    ...     'data': 'length', 'column':'species',
    ...     'rows': ['setosa', 'versi']}
    >>>
    >>> data = normalize(records, **kwargs)
    >>> next(data)
    {'color': 'blue', 'length': 5, 'species': 'setosa'}

csv files
~~~~~~~~~

.. code-block:: python

    >>> from meza import convert as cv
    >>> from meza.io import write
    >>>
    >>> records = [
    ...     {'item': 'a', 'amount': 200},
    ...     {'item': 'b', 'amount': 400}]
    >>>
    >>> csv = cv.records2csv(records)
    >>> write('output.csv', csv)

JSON files
~~~~~~~~~

.. code-block:: python

    >>> json = cv.records2json(records)
    >>> write('output.json', json)

Exercise #2
-------------------------------

Problem
~~~~~~~~~

.. code-block:: python

    # create a list of dicts with keys "factor", "length", "width", and "ratio" (for factors 1 - 20)

    records = [
        {
            'factor': 1, 'length': 2, 'width': 2,
            'ratio': 1.0
        }, {
            'factor': 2, 'length': 2, 'width': 2,
            'ratio': 0.6324…
        }, {
            'factor': 3, 'length': 2, 'width': 2,
            'ratio': 0.4472…}
    ]

    # group the records by quartiles of the "ratio" value, and aggregate each group by the median "ratio"

    from statistics import median
    from meza.process import group

    records[0]['ratio'] // .25

    # write the records out to a csv file (1 row per group)

    from meza.convert import records2csv
    from meza.io import write

    # | key | median |
    # | --- | ------ |
    # |   0 | 0.108… |
    # |   1 | 0.343… |
    # |   2 | 0.632… |
    # |   4 | 1.000… |

Solution
~~~~~~~~~

.. code-block:: python

    >>> from statistics import median
    >>> from meza import process as pr
    >>>
    >>> def aggregator(group):
    ...     ratios = (g['ratio'] for g in group)
    ...     return median(ratios)
    >>>
    >>> kwargs = {'aggregator': aggregator}
    >>> gkeyfunc = lambda r: r['ratio'] // .25
    >>> groups = pr.group(
    ...     records, gkeyfunc, **kwargs)
    >>>
    >>> from meza import convert as cv
    >>> from meza.io import write
    >>>
    >>> results = [
    ...     {'key': k, 'median': g}
    ...     for k, g in groups]
    >>>
    >>> csv = cv.records2csv(results)
    >>> write('results.csv', csv)

Introducing riko
-------------------------------

Python Events Calendar
~~~~~~~~~

.. code-block:: python

    # obtaining data
    >>> from riko.collections import SyncPipe
    >>>
    >>> url = 'www.python.org/events/python-events/'
    >>> _xpath = '/html/body/div/div[3]/div/section'
    >>> xpath = '{}/div/div/ul/li'.format(_xpath)
    >>> xconf = {'url': url, 'xpath': xpath}
    >>> kwargs = {'emit': False, 'token_key': None}
    >>> epath = 'h3.a.content'
    >>> lpath = 'p.span.content'
    >>> rrule = [{'field': 'h3'}, {'field': 'p'}]
    >>>
    >>> flow = (
    ...     SyncPipe('xpathfetchpage', conf=xconf)
    ...         .subelement(
    ...             conf={'path': epath},
    ...             assign='event', **kwargs)
    ...         .subelement(
    ...             conf={'path': lpath},
    ...             assign='location', **kwargs)
    ...         .rename(conf={'rule': rrule}))
    >>> stream = flow.output
    >>> next(stream)
    {'event': 'PyDataBCN 2017',
     'location': 'Barcelona, Spain'}
    >>> next(stream)
    {'event': 'PyConWEB 2017',
     'location': 'Munich, Germany'}

    # transforming data
    >>> dpath = 'p.time.datetime'
    >>> frule = {
    ...     'field': 'date', 'op': 'after',
    ...     'value':'2017-06-01'}
    >>>
    >>> flow = (
    ...     SyncPipe('xpathfetchpage', conf=xconf)
    ...         .subelement(
    ...             conf={'path': epath},
    ...             assign='event', **kwargs)
    ...         .subelement(
    ...             conf={'path': lpath},
    ...             assign='location', **kwargs)
    ...         .subelement(
    ...             conf={'path': dpath},
    ...             assign='date', **kwargs)
    ...         .rename(conf={'rule': rrule})
    ...         .filter(conf={'rule': frule}))
    >>> stream = flow.output
    >>> next(stream)
    {'date': '2017-06-06T00:00:00+00:00',
     'event': 'PyCon Taiwan 2017',
     'location': 'Academia Sinica, 128 Academia Road, Section 2, Nankang, Taipei 11529, Taiwan'}

    # Parallel processing
    >>> from meza.process import merge
    >>> from riko.collections import SyncCollection
    >>>
    >>> _type = 'xpathfetchpage'
    >>> source = {'url': url, 'type': _type}
    >>> xpath2 = '{}/div/ul/li'.format(_xpath)
    >>> sources = [
    ...     merge([source, {'xpath': xpath}]),
    ...     merge([source, {'xpath': xpath2}])]

    >>> sc = SyncCollection(sources, parallel=True)
    >>> flow = (sc.pipe()
    ...         .subelement(
    ...             conf={'path': epath},
    ...             assign='event', **kwargs)
    ...         .rename(conf={'rule': rrule}))
    >>>
    >>> stream = flow.list
    >>> stream[0]
    {'event': 'PyDataBCN 2017'}

Exercise #3
-------------------------------

Problem
~~~~~~~~~

.. code-block:: python

    # fetch the Python jobs rss feed
    # tokenize the "summary" field by newlines ("\n")
    # use "subelement" to extract the location (the first "token")
    # filter for jobs located in the U.S.
    # (use the 'fetch', 'tokenizer', 'subelement', and 'filter' pipes)

    from riko.collections import SyncPipe
    url = 'https://www.python.org/jobs/feed/rss'

    # write the 'link', 'location', and 'title' fields of each record to a json
    # file

    from meza.fntools import dfilter
    from meza.convert import records2json
    from meza.io import write

Solution
~~~~~~~~~

.. code-block:: python

    >>> from riko.collections import SyncPipe
    >>>
    >>> url = 'https://www.python.org/jobs/feed/rss'
    >>> fetch_conf = {'url': url}
    >>> tconf = {'delimiter': '\n'}
    >>> rule = {'field': 'location', 'op': 'contains'}
    >>> frule = [
    ...     {'field': 'location', 'op': 'contains', 'value': 'usa'},
    ...     {'field': 'location', 'op': 'contains', 'value': 'united states'}]
    >>>
    >>> fconf = {'rule': frule, 'combine': 'or'}
    >>> kwargs = {'emit': False, 'token_key': None}
    >>> path = 'location.content.0'
    >>> rrule = [
    ...     {'field': 'summary'},
    ...     {'field': 'summary_detail'},
    ...     {'field': 'author'},
    ...     {'field': 'links'}]
    >>>
    >>> flow = (SyncPipe('fetch', conf=fetch_conf)
    ...    .tokenizer(conf=tconf, field='summary', assign='location')
    ...    .subelement(conf={'path': path}, assign='location', **kwargs)
    ...    .filter(conf=fconf)
    ...    .rename(conf={'rule': rrule}))
    >>>
    >>> stream = flow.list
    >>> stream[0]
    {'dc:creator': None,
     'id': 'https://python.org/jobs/2570/',
     'link': 'https://python.org/jobs/2570/',
     'location': 'College Park,MD,USA',
     'title': 'Python Developer - MarketSmart',
     'title_detail': 'Python Developer - MarketSmart',
     'y:published': None,
     'y:title': 'Python Developer - MarketSmart'}
    >>>
    >>> from meza import convert as cv
    >>> from meza.fntools import dfilter
    >>> from meza.io import write
    >>>
    >>> fields = ['link', 'location', 'title']
    >>> records = [
    ...     dfilter(item, blacklist=fields, inverse=True) for item in stream]
    >>>
    >>> json = cv.records2json(records)
    >>> write('pyjobs.json', json)
