# FastStyle

First off, before you read this, let me plainly state that I am deeply and profoundly stupid and am often predisposed towards overzealousness. It is entirely possible that everything I have written (and, in some cases, Claude has written) in this codebase is fundamentally misguided and, like its creator, very stupid.

With that being said, this system has helped me write good looking web pages in FastHTML. If you don't want to get all of your styling from a plug-and-play opinionated style/component library, or if you want to communicate a bespoke design system in Python (especially with FastHTML), this may actually be quite useful for you. It also doesn't require you to learn a whole lot of new concepts to start using (hopefully). 

FastStyle provides a declarative API for defining styles with automatic CSS generation, style composition, and usage tracking in pure Python.

Well, maybe "pure" is not a useful term here. It actually has JavaScript in it (which perhaps could be said about all things at this point).

Anyways, enjoy!

## Features

- **Type-safe CSS properties** - When you type "." (provided you using modern IDE with autocomplete features), you get the benefit of being hardened CSS shaman without any of the up-front cost (cost come later)
- **Design token integration** - FastHTML recommends MonsterUI, but this is lots of complex systems under the hood you have to know about. And you have to use Tailwind. And if you have any opinions about what it should look like, complexity go brrrr. This easier for custom design systems.
- **Style composition** - You can basically apply your hard opinions about how code should be organized to your CSS. Issoke. Compose, decompose, SoC or LoB, DRY/SOLID, Clean/Dirty, whatever man. Do what feels right. Go nuts. 
- **Nested selectors** - Support for pseudo-classes, media queries, and child selectors - call FastStyle for your "thing" (component, utility class, whatever you want my guy). Pass in ChildStyle calls for variations on/permutations of "thing". Hell, throw ChildStyle calls in for children of "thing", or mobile representation of "thing". Or don't and just do all that explicitly without any higher-order Agarthan function magic. "It's your 'thing': do what you wanna do" - The Isley Brothers
- **Automatic CSS generation** - Styles are collected and output as optimized CSS. Write it to a file and sideload it as a resource (*spits*). Be a based mega-dict chad and just yeet it into a header style tag. You actually don't need to care about which, and your users will *never* care about it either (they just want page make load fast see stuff sooner).
- **Optimized CSS** - You can very easily just not include any CSS thats not used at a per-page level. Again, write it to a file and sideload then cache, or build then cache and load in header style tag. Or don't cache who gibs fuk. Is all still pretty fast at the end of the day. And if you're using this it probably will never matter which it is.
- **Python-native design tokens** - Comes with its own token build system so you don't need Node.js or Style Dictionary (unless you want to use them - we're not judgy). Define your tokens in JSON/YAML, run `faststyle tokens build`, and boom - you got Python enums, CSS variables, whatever you need.
- **Zero runtime overhead** - All CSS is generated at build time. Well, not "all". Maybe not "zero" runtime overhead. TBH Claude may have went overboard here. There might be *some* runtime overhead if you choose to have dynamic style definition based on user interactions. You can do that too. But default case is "all" and maybe "zero" but also probably not "zero" but still pretty quick. Is gud.

## Installation

```bash
pip install faststyle
```

## Quick Start

Here's the simplest example that actually does something useful:

```python
from faststyle import FastStyle, ChildStyle

# Define a button style
muh_button = FastStyle(
    "btn-primary",
    padding="12px 24px",
    background_color="#007bff",
    color="white",
    border_radius="4px",
    cursor="pointer",
    transition="all 0.2s ease",
    
    # Hover state
    ChildStyle("&:hover",
        background_color="#0056b3",
        transform="translateY(-1px)"
    ),
    
    # Responsive design
    ChildStyle("@media (max-width: 768px)",
        padding="8px 16px",
        font_size="14px"
    )
)

# Use in your template - it's just a string when you need it to be
html = f'<button class="{muh_button}">Click meh</button>'

# Use with FastHTML and skip template hell
button = ft.Button('Click meh', cls=muh_button)
```

## Core Concepts

### FastStyle Function

The `FastStyle` function is the main thing you'll use. It returns a `ClassDispatch` object, which sounds fancy but really just means "a thing that acts like a string when you need it to".

```python
card = FastStyle(
    "card",
    background="white",
    border_radius="8px",
    padding="16px",
    box_shadow="0 2px 4px rgba(0,0,0,0.1)"
)

# Use as string
print(card)  # Output: "card"
print(f"my-{card}")  # Output: "my-card"
```

### ChildStyle for Nested Selectors

`ChildStyle` is how you handle all the nested stuff - hover states, media queries, child elements, whatever CSS lets you nest. It's pretty straightforward once you see it in action:

```python
nav = FastStyle(
    "nav",
    display="flex",
    gap="16px",
    
    # Pseudo-class
    ChildStyle("&:hover", background="rgba(0,0,0,0.05)"),
    
    # Child element (accessible via nav.link)
    ChildStyle(".link",
        color="blue",
        text_decoration="none"
    ),
    
    # Media query with nested children
    ChildStyle("@media (max-width: 768px)",
        flex_direction="column",
        ChildStyle(".link", padding="8px")
    )
)
```

Also, ChildStyle come along for ClassDispatch ride - you can access class names for nested ChildStyle selectors with dot notation
```python
# Access parent selector with the ClassDispatch object returned by FastStyle
def mek_nav_menu(*links) -> ft.Div:
    return ft.Div(links, cls=nav)

# Access child classes for use with chirrens
nav_menu = mek_nav_menu(ft.A('muhproducts', cls=nav.link), ft.A('muhservices', cls=nav.link))
```

### Style Composition

You can also make styles reference other styles to inherit their properties. This is helpful when you have base styles you want to reuse (which you may not, no h8 - this just like, an option, man):

```python
# Base styles
text_base = FastStyle("text-base", font_size="16px", line_height="1.5")
interactive = FastStyle("interactive", cursor="pointer", user_select="none")

# Composed style
button = FastStyle(
    "button",
    ref=[text_base, interactive],  # Inherits from multiple styles
    padding="8px 16px",
    background="blue",
    color="white"
)
```

### Design Token Integration

If you're using a design token system (tbh this has become more trouble than its worth but I hold out hope that it will pay off one day), FastStyle comes with some conveniences to make these tokens more accessible in Python (press "." and cure all stupidity woes).

First, define your tokens in JSON (or YAML if you're that person):

```json
// tokens.json
{
  "base": {
    "color": {
      "blue": { "100": "#e3f2fd", "500": "#2196f3", "900": "#0d47a1" }
    },
    "spacing": {
      "sm": "8px", "md": "16px", "lg": "24px"
    }
  },
  "semantic": {
    "color": {
      "primary": "{base.color.blue.500}",
      "text": { "primary": "#333", "secondary": "#666" }
    }
  }
}
```

Then build your tokens:

```bash
# Generate Python classes and CSS variables
faststyle tokens build

# Or with options
faststyle tokens build --source ./design/tokens.json --output ./src/generated/
```

Now use them in your styles:

```python
# Auto-generated from your tokens
from generated.tokens import s, p  # semantic and primitive tokens

btn_base = FastStyle(
    "btn-base",
    padding=f"{s.spacing.SM} {s.spacing.MD}",  # Semantic tokens
    background=s.color.primary,
    color=p.color.blue._100,  # Primitive tokens (numeric keys prefixed with _)
    font_size=s.typography.body.size
)
```

## Registry System

Styles get automatically registered to a global registry that handles CSS generation. This might sound overcomplicated (and maybe it is), but it means you don't have to manually collect your styles or worry about output order:

```python
from faststyle import StyleRegistry

# Create a custom registry (optional - a default one is provided)
my_registry = StyleRegistry()

# Define styles with custom registry
header = FastStyle(
    "hdr",
    height="60px",
    registry=my_registry
)

# Generate CSS
css = my_registry.generate_css()  # All registered styles
css_optimized = my_registry.generate_css(optimized=True)  # Only used styles
```

## Usage Tracking

FastStyle can track which styles actually get used in your application. This is probably premature optimization, as I find it broadly unlikely you even need the optimization features here, but it would be dumb not to include it since I (see: Claude) did build it already anyways:

```python
# Styles are marked as "used" when converted to string
btn_base = FastStyle("btn-base", background="blue")

# This marks the style as used
muh_button = ft.Button('Click Meh', cls=btn_base)

# Generate optimized CSS (only includes used styles)
css = registry.generate_css(optimized=True)
```

## Advanced Features

### Style Graph Analysis

So, for many unimportant reasons, I ended up using a graph system to track the usage of styles and be able to optimize the CSS load and avoid junking up the DOM with intermediary classes/styles (since I don't sideload CSS files). I'll be honest - this is probably overkill for most use cases, but it exists so I am going to mention it lest ye get confused by it:

```python
from faststyle import get_style_graph

# Analyze style usage
graph = get_style_graph()
graph.analyze_usage()

# Categories:
# - 'direct': Used on elements
# - 'mixin_only': Only referenced by other styles  
# - 'unused': Never used
```

### Multiple Registries

You can create separate registries for different contexts. The main use case I've found for this is keeping email CSS (with all its limitations) separate from your regular web styles:

```python
web_registry = StyleRegistry()
email_registry = StyleRegistry()

# Web styles with advanced CSS
web_button = FastStyle(
    "button",
    background="linear-gradient(45deg, #667eea 0%, #764ba2 100%)",
    backdrop_filter="blur(10px)",
    registry=web_registry
)

# Email-safe styles
email_button = FastStyle(
    "btn-email", 
    background="#667eea",
    border="1px solid #5a67d8",
    registry=email_registry
)
```

### Property Helpers

There are CSS enum classes if you want that extra type safety (or just prefer autocomplete over remembering CSS values):

```python
from faststyle import CSS

card = FastStyle(
    "card",
    display=CSS.display.FLEX,
    flex_direction=CSS.flex_direction.COLUMN,
    align_items=CSS.align_items.CENTER,
    position=CSS.position.RELATIVE,
    cursor=CSS.cursor.POINTER
)
```

I also threw in the EmailCSS convenience class I use to get IDE support for "how mek email CSS that work for most email providers without learning their CSS support" - it only includes commonly supported values:

```python
email_button=FastStyle(
    "btn-email",
    background="#667eea",
    border=f"{EmailCSS.border_width('1px')} {EmailCSS.border_style.SOLID} #5a67d8",
    registry=email_registry
)
```

## API Reference

### FastStyle

```python
def FastStyle(
    class_name: str,
    *children: ChildStyle,
    ref: Optional[ClassDispatch | List[ClassDispatch]] = None,
    registry: Optional[StyleRegistry] = None,
    **css_properties
) -> ClassDispatch
```

**Parameters:**
- `class_name`: The CSS class name (without the dot prefix)
- `*children`: ChildStyle definitions for nested styles
- `ref`: Optional style(s) to inherit properties from
- `registry`: Custom style registry (uses global default if not provided)
- `**css_properties`: Any valid CSS properties (snake_case converts to kebab-case)

**Returns:** 
- `ClassDispatch` object that acts as a string and provides child access

### ChildStyle

```python
def ChildStyle(
    selector: str,
    *nested_children: ChildStyle,
    ref: Optional[ClassDispatch | List[ClassDispatch]] = None,
    **css_properties
) -> StyleDef
```

**Parameters:**
- `selector`: CSS selector pattern that gets processed based on what it contains:
  - `"@media (...)"` - Media query: wraps all nested styles
  - `"&:hover"`, `"&:focus"`, `"&.active"` - The `&` gets replaced with parent selector
  - `".child"`, `"#id"` - Selectors starting with `.` or `#` append as descendant
  - `"header"`, `"footer"` - Plain strings: if it's an HTML element, uses as-is; otherwise adds `.` prefix
  - `"div > span"`, `":not(:last-child)"` - Complex selectors (containing `>`, `+`, `~`, `:`, etc.) used as-is
- `*nested_children`: Further nested ChildStyle definitions
- `ref`: Optional style(s) to inherit properties from
- `**css_properties`: CSS properties for this selector

**Selector Processing Examples:**
```python
card = FastStyle('card',
    # & replacement
    ChildStyle('&:hover', ...),           # → .card:hover
    ChildStyle('& .active', ...),         # → .card .active
    
    # Named children (accessible via card.header, card.body)
    ChildStyle('.header', ...),           # → .card .header
    ChildStyle('header', ...),            # → .card .header (auto-prefixed)
    
    # HTML elements
    ChildStyle('img', ...),               # → .card img
    ChildStyle('div > span', ...),        # → .card div > span
    
    # Complex selectors
    ChildStyle('&:not(:last-child)', ...), # → .card:not(:last-child)
    
    # Reverse selector (parent inside child)
    ChildStyle('.sidebar &', ...),        # → .sidebar .card
)
```

### StyleRegistry

```python
class StyleRegistry:
    def add_style(self, media_query: str, selector: str, properties: Dict[str, str])
    def generate_css(self, optimized: bool = False) -> str
    def clear(self)
```

**Methods:**
- `add_style`: Manually add a style rule
- `generate_css`: Generate CSS output
  - `optimized=False`: Include all registered styles
  - `optimized=True`: Only include used styles
- `clear`: Clear all registered styles

### ClassDispatch

The object returned by `FastStyle`:

```python
class ClassDispatch:
    # String representation
    def __str__(self) -> str  # Returns the class name
    
    # Child access
    def __getattr__(self, name: str) -> str  # Returns child class name
```

## Best Practices

1. **Define styles at module level** - They're meant to be defined once and reused, not recreated on every render
2. **Use design tokens** - It will be easier for you to change shit (and know what shit you can use when writing other shit) if you use the design tokens. But this actually not best practice, this just a suggestion if you using bespoke design system.
3. **Compose, don't duplicate** - Use `ref` to share common properties (but also don't go crazy with the abstraction - here be Dragonball Z weebery)
4. **Organize by component** - Keep your styles near the components that use them so is easier to change shit. Use a common/components.py to organize your global components that you use everywhere. Use page/user flow branch modules to define one-off components that only get used on that page/in that user interaction flow. In any case, write your styles above your components (on god is much easier, even if they long).
5. **Use the CSS enums** - Or don't, I'm not your dad (I'm pretty sure). But the autocomplete mek life real gud.
6. **Don't make new files, use folds** This just a general principle that has helped me when working with FastHTML and this design framework, but ultimately life has been a lot easier for me when I avoid defining new modules. If a file gets too big for you to comprehend, use folding to organize shit. Got 50 styles above 6 different related component factories? Cool. Use custom folding regions to section off the component type, then use other custom folding regions to section off styles vs components:

```python
# ===== FUNCTIONS ===== #

## ===== COMPONENTS ===== ##

### ===== BUTTONS ===== ###

#!> Styles

#!> Base button styles
btn_base = FastStyle(...)

btn_disabled = FastStyle(...)
#!<

#!> Primary button styles
...
#!<

...

#!< end of Styles

#!> Factories
def PrimaryButton(...) -> ft.Button: return ft.Button(..., cls=f'{btn_base} {btn_primary} {btn_lg}')
def GhostButton(...) -> ft.Button: return ft.Button(..., cls=f'{btn_base} {btn_ghost} {btn_sm}')
#!<
```

Now, is this a "best practice" with respect to this specific library? No, this an oddly specific opinion about programming, largely detached from the purpose of this README. But, it has helped me immensely to be fearful of small files and the masculine urge to create lots of them. So, I include it here because I don't have a blog or a YouTube channel (at least, not about programming). I hope it helps you as it has helped me (to mek unnasten code gud).

## Integration Examples

### FastHTML

```python
from fasthtml import Div, Button
from faststyle import FastStyle

# Define styles
button_style = FastStyle(
    "btn",
    padding="8px 16px",
    background="#007bff",
    color="white",
    border_radius="4px"
)

# Use in components
def MyButton(text, **props):
    return Button(text, cls=button_style, **props)
```

### Django

```python
# styles.py
from faststyle import FastStyle, get_default_registry

button = FastStyle("btn", padding="8px 16px", background="#007bff")

def get_styles_css():
    return get_default_registry().generate_css()

# views.py
def my_view(request):
    context = {
        'styles_css': get_styles_css(),
        'button_class': str(button)
    }
    return render(request, 'template.html', context)
```

### Flask

```python
from flask import Flask, render_template_string
from faststyle import FastStyle, get_default_registry

app = Flask(__name__)

# Define styles
card = FastStyle("card", padding="16px", background="white", border_radius="8px")

@app.context_processor
def inject_styles():
    return {
        'styles_css': get_default_registry().generate_css(),
        'card_class': str(card)
    }
```

## Token System

### Building Tokens

FastStyle includes a Python-native token builder (no JavaScript I kno is radical rite?):

```bash
# Basic usage - looks for tokens.json in current dir
faststyle tokens build

# With options
faststyle tokens build \
  --source ./design/tokens.yml \
  --output ./src/generated/ \
  --format css,python
```

Or use the Python API if you no likey CLI:

```python
from faststyle.tokens import TokenBuilder

builder = TokenBuilder('./tokens.json')
builder.add_output('css', './generated/tokens.css')
builder.add_output('python', './generated/tokens.py')
builder.build()
```

### Token File Format

Tokens follow a simple structure. Use `base` for primitives and `semantic` for... semantic tokens:

```json
{
  "base": {
    "color": {
      "gray": {
        "100": { "value": "#f7f7f7" },
        "500": { "value": "#888888" },
        "900": { "value": "#111111" }
      }
    }
  },
  "semantic": {
    "color": {
      "text": {
        "primary": { "value": "{base.color.gray.900}" },
        "secondary": { "value": "{base.color.gray.500}" }
      }
    }
  }
}
```

References (the `{base.color.gray.900}` things) get resolved automatically. It's like variables but with more steps.

You can also add your own third tier if you deranged carpenter (i.e. primitive + alias + mapped, base + semantic + component, etc.).

### What Gets Generated

Running `faststyle tokens build` creates:

1. **tokens.css** - CSS custom properties
   ```css
   :root {
     --color-text-primary: #111111;
     --color-text-secondary: #888888;
   }
   ```

2. **tokens.py** - Python enums and classes
   ```python
   # Semantic tokens (use these mostly)
   class Semantics:
       class color:
           TEXT_PRIMARY = "var(--color-text-primary)"
           TEXT_SECONDARY = "var(--color-text-secondary)"
   
   s = Semantics()

   # Primitive tokens (when you need the raw values)
   class Primitives:
       class color:
           class gray:
               _100 = "#f7f7f7"  # Numeric keys prefixed with _
               _500 = "#888888"
               _900 = "#111111"

    p = Primitives()

    # Add to generated/__init__.py
    from semantics import s
    from primitives import p

    __all__ = ['p', 's']

    # Then you just import:
    from generated import p, s
   ```

### Customizing Token Builds

If you need custom transforms or outputs (likely):

```python
from faststyle.tokens import TokenBuilder, Transform

# Custom transform
def add_pixel_units(token):
    if token.get('type') == 'spacing' and isinstance(token['value'], (int, float)):
        token['value'] = f"{token['value']}px"
    return token

# Custom formatter
def format_scss_map(tokens):
    # Generate SCSS map format
    return "$tokens: (...);"

# Use them
builder = TokenBuilder('./tokens.json')
builder.add_transform(add_pixel_units)
builder.add_output(format_scss_map, './generated/_tokens.scss')
builder.build()
```

## Contributing

If you find this useful and want to contribute, I'm honestly shocked and hey man, thats pretty cool. Welcome aboard. If you find bugs... well, yeah. I figured that would probably happen. Gosh, yknow... jeez, like... man... 

See [Contributing Guide](CONTRIBUTING.md) for details.

## License

MIT License - see [LICENSE](LICENSE) for details. Basically, do whatever you want with it, just don't blame me if it breaks (likely).