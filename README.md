# svgobjects: library to build SVG files programmatically.

This document doubles down as documentation and a test for the library you can execute to ensure it works as intended.

To execute it use the command ⌨️ `python -m doctest README.md` (depending on your system, you might need to use the word `python3` instead). If no message appears, it means all tests passed. You can also use the `-v` flag to display additional information for each test.

> [!NOTE]
> The comment `# doctest: +ELLIPSIS` seen in many of the examples just means we replaced some of the output with an ellipsis (`...`) to make it shorter.

I'm [Kennedy Richard S. Guerra](https://kennedyrichard.com) (born in 1990), creator of this library. Please, consider [supporting](https://indiesmiths.com/donate) my development work for this and many other open-source projects. Also check the author's notes at the end of this document for more info on this library.


## Installation/download

This library can be installed via pip command or downloaded to use as a subpackage in your own projects. To install via pip, execute the following command:

```
pip install svgobjs
```

Depending on your system, you might need to replace the word `pip` with `pip3`.


## Usage


### Importing

Start by importing the classes you want to use. The classes represent SVG elements. Here are all available classes:

```python
>>> from svgobjs.elements import (
...   SVG,
...   G,
...   Line,
...   PolyLine,
...   Polygon,
...   Rect,
...   Circle,
...   Ellipse,
...   Path,
... )

```

There's also an utility function that may come in handy, as we'll see in a later section:

```python
>>> from svgobjs.utils import get_svg_formatted_color

```

### Quick demonstration

Before learning in detail how each object works and how they are combined, let's start with a bang 💥!

Perhaps one of the most beautiful and simple graphics one can create with simple SVG shapes is Japan's national flag:

<p align="center">
  <img alt="Japan's national flag" src="https://i.imgur.com/HgLj5Z6.png" />
</p>

According to [its government](https://www.japan.go.jp/japan/flagandanthem/index.html), its height to width ratio is 2:3 and circle (rising sun) at the center has a diameter of 3/5 of the flag's height (meaning the radius is half of that). The page from the link doesn't specify the exact RGB values of the red in the flag, but we'll use the same red they use on the image shown on the page, which is `#fd2b29`.

The only additional element we'll draw which is not part of the official flag is a black outline, represented by another rect with a black stroke and no fill.

We can create the SVG text defining that flag with this code:

```python
>>> height = 200
>>> width = 300 # height / 2 * 3
>>> radius = 60 # height * (3/5) / 2
>>> center = 150, 100 # half of width and height

>>> flag_svg = SVG(width=width, height=height)

>>> flag_rect = Rect(x=0, y=0, width=width, height=height, fill='white')
>>> flag_circle = Circle(r=radius, cx=center[0], cy=center[1], fill='#fd2b29')
>>> flag_outline_rect = Rect(
... x=0, y=0, width=width, height=height,
... fill='none', stroke='black', stroke_width=1)

>>> flag_svg.append(flag_rect)
>>> flag_svg.append(flag_circle)
>>> flag_svg.append(flag_outline_rect)

>>> svg_text = str(flag_svg)

```

The `svg_text` variable should contain this text:

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="200">
    <rect x="0" y="0" width="300" height="200" fill="white" />
    <circle r="60" cx="150" cy="100" fill="#fd2b29" />
    <rect x="0" y="0" width="300" height="200" fill="none" stroke="black" stroke-width="1" />
</svg>
```

Let's see if it that is true:

```python
>>> svg_text_for_testing = (
... """<svg xmlns="http://www.w3.org/2000/svg" width="300" height="200">
...     <rect x="0" y="0" width="300" height="200" fill="white" />
...     <circle r="60" cx="150" cy="100" fill="#fd2b29" />
...     <rect x="0" y="0" width="300" height="200" fill="none" stroke="black" stroke-width="1" />
... </svg>"""
... )

>>> svg_text == svg_text_for_testing
True

```

Of course, printing will display the same text:

```python
>>> print(flag_svg)
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="200">
    <rect x="0" y="0" width="300" height="200" fill="white" />
    <circle r="60" cx="150" cy="100" fill="#fd2b29" />
    <rect x="0" y="0" width="300" height="200" fill="none" stroke="black" stroke-width="1" />
</svg>

```


### Getting started

The library is designed in a way that allows you to build SVGs in different ways, so you can pick the most suited to your needs.

For instance, you can start by creating an empty SVG element:

```python
>>> mysvg = SVG()

```

However, on top of not having any children, this SVG is also incomplete because it lacks meaningful values for important attributes like `width`, `height` and `viewBox`:

```python
>>> mysvg
SVG(xmlns='http://www.w3.org/2000/svg', width=None, height=None, viewBox=None)

```

At any point in time you can obtain the equivalent SVG text by turning the object into a string:

```python
>>> str(mysvg)
'<svg xmlns="http://www.w3.org/2000/svg">\n</svg>'

>>> print(mysvg)
<svg xmlns="http://www.w3.org/2000/svg">
</svg>

```

Note that values that represent a single number (like the values of the attributes `x` and `y`) are kept as-is, that is, without being converted into strings. That is, the attribute `x` is `0` rather than `"0"`. This is only a convenience, as it makes it easier to perform calculations on those values without having to convert them into numbers then back into strings.

It doesn't cause any harm, since those values are turned into strings automatically when we turn the objects into strings, as shown earlier (when `mysvg` was converted into a string or printed, the results displayed `x="0"` instead of `x=0`).

However, as we said before, `mysvg` lacks some important attributes. Let's provide them:

```python
>>> mysvg.width = 800
>>> mysvg.height = 600
>>> mysvg.viewBox = '0 0 800 600' # not a single number, so we use a string

```

Now it's starting to look better:

```python
>>> mysvg
SVG(xmlns='http://www.w3.org/2000/svg', width=800, height=600, viewBox='0 0 800 600')

>>> print(mysvg)
<svg xmlns="http://www.w3.org/2000/svg" width="800" height="600" viewBox="0 0 800 600">
</svg>

```

Let's add a `<line>` to it:

```python
>>> line = Line()

>>> line # doctest: +ELLIPSIS
Line(x1=0, y1=0, x2=0, y2=0, stroke=None, stroke_width=None, ..., id=None)

>>> str(line)
'<line x1="0" y1="0" x2="0" y2="0" />'

>>> mysvg.append(line) # doctest: +ELLIPSIS

```

Now `mysvg`'s representation in the interpreter indicates it has children, although the children themselves are not shown (only the word `children` is displayed at the end):

```python
>>> mysvg
SVG(xmlns='http://www.w3.org/2000/svg', width=800, height=600, viewBox='0 0 800 600', children)

```

However, turning `mysvg` into a string or printing it displays the full SVG document, including the children. Here's what the full SVG text looks like:

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="800" height="600" viewBox="0 0 800 600">
    <line x1="0" y1="0" x2="0" y2="0" />
</svg>
```

Let's see if that is true:

```python
>>> mysvg_string = str(mysvg)

>>> text_to_compare = (
... """<svg xmlns="http://www.w3.org/2000/svg" width="800" height="600" viewBox="0 0 800 600">
...     <line x1="0" y1="0" x2="0" y2="0" />
... </svg>""")

>>> mysvg_string == text_to_compare
True

```

You can actually access the children list directly:

```python
>>> isinstance(mysvg._children, list)
True

>>> line in mysvg._children
True

>>> mysvg._children # doctest: +ELLIPSIS
[Line(x1=0, y1=0, x2=0, y2=0, stroke=None, stroke_width=None, ...)]

```

You can also populate an SVG instance when creating it:

```python
>>> anothersvg = SVG(
... width=640, height=360, viewBox='0 0 640 360',
... children=[Line(x1=0, y1=0, x2=640, y2=360, stroke='black', stroke_width=1)]
... )

```

As usual, turning it into a string reveals the full SVG document:

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="640" height="360" viewBox="0 0 640 360">
    <line x1="0" y1="0" x2="640" y2="360" stroke="black" stroke-width="1" />
</svg>
```

Again, let's test it:

```python
>>> anothersvg_string = str(anothersvg)
>>> another_text_to_compare = (
... """<svg xmlns="http://www.w3.org/2000/svg" width="640" height="360" viewBox="0 0 640 360">
...     <line x1="0" y1="0" x2="640" y2="360" stroke="black" stroke-width="1" />
... </svg>"""
... )
>>> anothersvg_string == another_text_to_compare
True

```

### Groups/grouping

For grouping and composing more complex shapes, you can use the `G` class, which represents the `<g>` element.

Just like SVG, G can have children. The advantage of G, however, is that it can be nested in the SVG hierarchy and have transforms applied to it, whereas an SVG hierarchy can only have a single SVG element, which sits at the top of the hierarchy.

```python
>>> group = G()
>>> group
G(fill=None, stroke=None, stroke_width=None, transform=None, id=None)

>>> group.append_g(id="inner_group")
>>> group._children[0]
G(fill=None, stroke=None, stroke_width=None, transform=None, id='inner_group')

```

Like any element, we can obtain the equivalent SVG text by turning it into a string:

```svg
<g>
    <g id="inner_group">
    </g>
</g>
```

Again, let's test it:

```python
>>> group_svg_text = str(group)
>>> group_text_to_compare = (
... """<g>
...     <g id="inner_group">
...     </g>
... </g>"""
... )
>>> group_svg_text == group_text_to_compare
True

```


### Other shapes

Naturally, all other shapes work as expected.

```python
>>> polyline = PolyLine(points='0,0 10,10 0,20')
>>> polyline # doctest: +ELLIPSIS
PolyLine(points='0,0 10,10 0,20', stroke=None, stroke_width=None, ..., transform=None, id=None)
>>> str(polyline)
'<polyline points="0,0 10,10 0,20" />'

>>> ellipse = Ellipse(rx=20, ry=10, cx=20, cy=10)
>>> ellipse # doctest: +ELLIPSIS
Ellipse(rx=20, ry=10, cx=20, cy=10, fill=None, stroke=None, ..., transform=None, id=None)
>>> str(ellipse)
'<ellipse rx="20" ry="10" cx="20" cy="10" />'

>>> polygon = Polygon(points='0,50 30,0 60,50')
>>> polygon # doctest: +ELLIPSIS
Polygon(points='0,50 30,0 60,50', fill=None, stroke=None, ..., transform=None, id=None)
>>> str(polygon)
'<polygon points="0,50 30,0 60,50" />'

>>> path = Path(d='M0,0 H50 L25,50')
>>> path # doctest: +ELLIPSIS
Path(d='M0,0 H50 L25,50', fill=None, stroke=None, ..., transform=None, id=None)
>>> str(path)
'<path d="M0,0 H50 L25,50" />'

```

### `get_svg_formatted_color` function

Providing colors to attributes using strings like `'white'` or `'#ffffff'` is fine, since it is expected. However, values like `"(255, 255, 255)"` are not recognized as values for attributes representing colors. That's what the utility function `get_svg_formatted_color` is for: it turns an iterable of numbers into a string recognized as a valid color for an SVG attribute.

```python
>>> get_svg_formatted_color((255, 255, 255))
'rgb(255, 255, 255)'

>>> get_svg_formatted_color([255, 255, 255])
'rgb(255, 255, 255)'

>>> str(Circle(r=20, cx=20, cy=20, fill=get_svg_formatted_color((255, 0, 0))))
'<circle r="20" cx="20" cy="20" fill="rgb(255, 0, 0)" />'

```

Note that strings passed to the function are returned as-is:

```python
>>> get_svg_formatted_color('white')
'white'

>>> get_svg_formatted_color('(255, 255, 255)')
'(255, 255, 255)'

```

Actually, any iterable is accepted:

```python
>>> get_svg_formatted_color(range(100, 161, 30))
'rgb(100, 130, 160)'

```

`None` is also accepted. In this case, the string `'none'` is returned, which is a possible value for an attribute representing a color.

```python
>>> get_svg_formatted_color(None)
'none'

```


## Author's notes

This library was created for usage as a subpackage within the [Nodezator](https://github.com/IndieSmiths/nodezator) app, but also published here as an standalone library for the sake of sharing our improvements in a more effective/efficient way (check the "knowledge management" subsection further ahead for more info on this).


### License

This code is open-source and dedicated to the public domain with [The Unlicense](https://unlicense.org/) with the goal of helping many people.


### Patreon and donations

Please, support this and other useful code/apps/games of the Indie Smiths project (the collection of my open-source projects) by becoming our patron on [patreon][]. You can also make recurrent donations using [GitHub sponsors][], [liberapay][] or [Ko-fi][].

Both [GitHub sponsors][] and [Ko-fi][] also accept one-time donations.

Any amount is welcome and helps. Check the project's [donation page][] for all donation methods available.


### Limitations

At least for now, this library is meant to be used for creating SVGs to be parsed by apps/libraries using NanoSVG, which means only a limited set of SVG features are included, perhaps not even all of the available ones for NanoSVG.


### Why another Python SVG library

Despite the existence of excellent libraries already available for free and with good licensing (like [svg.py](https://github.com/orsinium-labs/svg.py/)), I wanted to create one that people could simply copy as a local subpackage in their projects, without the need for any additional measures like including copies of the original license, etc.

I also want to include specialized (optional) features that may not suit existing libraries, so I also had to create and use my own (this) library. Although I'm not sure when I'll be able to do implement such specialized features.

For now, I'll focus on very basic features, as I build it incrementally.


### Knowledge management

This library is also our first effort towards improving the knowledge management of the Indie Smiths project (again, just a fancy name for the collection of my open-source projects).

Allow me to explain. Most of my Python libraries are created and kept as local subpackages of larger projects, never published as standalone libraries like this one, despite meeting all the requirements. As such, they are much harder to discover by people that could be interested in or in need of them. For instance, since this library would originally only live within Nodezator's code, only people curious enough to inspect Nodezator's source would know about its existence.

Instead, as an open-source maintainer, I'm trying to conceive of better ways to document and make available such code and the related knowledge to people who might need it. Publishing such individual subpackages as standalone libraries is one step in that direction.

You may ask: you'll have different copies of such libraries (at least one as a subpackage and the one published as a standalone library), wouldn't that be confusing or hard to maintain?

Regarding the "confusing" part, my answer is that it won't be confusing because they should be regarded as different forks of the same main idea/library, with no compulsory relationship between their contents. The original copy, the one that was conceived/implemented as a subpackage of a larger app, was created as a local subpackage precisely because it has no ties to projects outside the main package. It exists solely to serve the needs of the main package.

Despite that, because it may be useful outside such context, a no-strings-attached copy is published as a standlone library, like this one. Moreover, nothing prevents us from sharing improvements between those copies if time allows, despite the absence of such obligation/ties.

Regarding the "hard to maintain" part, my answer is that it would indeed be hard to maintain both, but only **if** that was our goal, which **it is not**. Again, our obligation is to the original copy that exists as a subpackage of a main, larger project.

It is also not uncommon for two or more large main projects to have their own dedicated local copies as subpackages as well, each with their own exclusive changes made with the purpose of better serving their respective main project. In other words, although it is something we'd like to do, we are under no obligation to implement changes made in the original copies in the standalone version published as a dedicated git repository.

It might even be impossible to do in some cases, because as we just stated, sometimes there are multiple implementations of a library as subpackages of different main projects and such subpackages may evolve in different ways that don't allow them to be merged together. Again, this isn't even our goal, as each subpackage exists solely to serve the needs of the main package.

The existence of the published standalone version is enough to help us with our goal of sharing code and knowledge that might be useful to others. Perhaps, if we get enough funding and are able to have more people working on the project, we could have people implement changes from existing copies into the one that exists as a standalone copy, that is, changes that are not conflicting, as we discussed in the previous paragraph. The important thing is for code and knowledge to be more easily accessible so people can do whatever they want with.

After all, as stated in the previous subsection, this library was created so anyone can copy it as a local subpackage without any additional obligations. Other repositories like this one that will follow this model, that is, standalone copies of existing subpackages from other main large projects, will be just like that as well (in practice, the whole code of the main projects can be used like that, since it all shares the same public domain open-source license (The Unlicense).


[patreon]: https://patreon.com/KennedyRichard
[GitHub sponsors]: https://github.com/sponsors/KennedyRichard
[liberapay]: https://liberapay.com/KennedyRichard
[Ko-fi]: https://ko-fi.com/kennedyrichard
[donation page]: https://indiesmiths.com/donate
